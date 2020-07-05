---
title: Reader Pattern
tags:
- Scala
categories:
- Scala Tutorial
date: 2020-07-05 22:35 +0800
---
We already introduced [Reader Monad](https://blog.shangjiaming.com/scala%20tutorial/reader-monad/),
It can inject dependency to function by returning a Reader effect.

In this blog, let's call the first type parameter of Reader Monad as context. we will put dependencies into the context and
use Reader Monad to implement the requirement in [Cake Pattern](https://blog.shangjiaming.com/scala%20tutorial/cake-pattern/)

# Context with minimum dependencies

Not like transaction in [Reader Monad](https://blog.shangjiaming.com/scala%20tutorial/reader-monad/),
we need to pass more than one dependency here, which can't be done directly by Reader Monad.
So we need to create a new type as context to wrap all the dependencies.

```scala
case class Env(http: HttpRequest, database: Database)
```

To use Reader Monad, we also need to modify the function return type of DataSource and DataStore.

```scala
trait DataSource[A] {
  def getData: Reader[A, List[Int]]
}

trait DataStore[A] {
  def save(data: List[Int]): Reader[A, Unit]
}
```

Then we can specify A as Env in the implementation

```scala
class HttpDataSource extends DataSource[Env] {
  override def getData: Reader[Env, List[Int]] =
    for {
      env <- Reader.ask[Env]
    } yield env.http.get("http://example.com/data").split(",").map(_.toInt).toList
}

class DatabaseStore extends DataStore[Env] {
  override def save(data: List[Int]): Reader[Env, Unit] =
    for {
      env <- Reader.ask[Env]
    } yield env.database.runSql(
      s"insert into data_table values(${data.mkString(",")})"
    )
}
```

Because DataJob don't need to access the dependencies in Env, we can leave context as A

```scala
class DataJob[A](
    source: DataSource[A],
    store: DataStore[A],
    encoder: DataEncoder
) {
  def run: Reader[A, Unit] =
    for {
      data <- source.getData
      val encodedData = encoder.encode(data)
      _ <- store.save(encodedData)
    } yield ()
}
```

Then the dependencies can be wired in main like this

```scala
object Main {
  def main() {

    val http = new LogHttpRequest()
    val database = new LogDatabase()
    val env = Env(http, database)

    val source = new HttpDataSource

    val store = new DatabaseStore

    val encoder = new PlusOneEncoder()

    val program = new DataJob[Env](source, store, encoder)

    program.run.run(env)
  }
}
```

You can find the full code in [reader-pattern-basic-in-context.scala](https://gist.github.com/sjmyuan/ed7d3439ddda35e840f5b703df6725e7)

Not like parameter pattern and cake pattern

* HttpDataSource doesn't depend on LogHttpRequest directly
* DatabaseStore doesn't depend on LogDatabase directly
* All the dependencies(env) is injected at the last step

Except DataJob, we can instantiated other components separately.

# Context with maximum dependencies

Maybe you will ask, could we also put source, store and encoder into context?
Then DataJob can also be instantiated separately.

Let's try

```scala
case class Env[A](http: HttpRequest, database: Database, source: DataStore[A], store: DataStore[A], encoder: DataEncoder)
```

DataStore and DataSource need a type parameter here, so we add a type parameter A to Env.
Then our implementation could be

```scala
class HttpDataSource extends DataSource[Env[???]] {
  override def getData: Reader[Env[???], List[Int]] =
    for {
      env <- Reader.ask[Env[???]]
    } yield env.http.get("http://example.com/data").split(",").map(_.toInt).toList
}

class DatabaseStore extends DataStore[Env[???]] {
  override def save(data: List[Int]): Reader[Env[???], Unit] =
    for {
      env <- Reader.ask[Env[???]]
    } yield env.database.runSql(
      s"insert into data_table values(${data.mkString(",")})"
    )
}
```

We use `Env[???]` here, because we don't know its final type.
It may looks like `Env[Env[Env[Env[....]]]]`, which is a dead loop.

To compile the code, maybe we can hard code the type parameter in Env

```scala
case class Env(http: HttpRequest, database: Database, source: DataStore[Env], store: DataStore[Env], encoder: DataEncoder)
```

Then the implementation could be

```scala
class HttpDataSource extends DataSource[Env] {
  override def getData: Reader[Env, List[Int]] =
    for {
      env <- Reader.ask[Env]
    } yield env.http.get("http://example.com/data").split(",").map(_.toInt).toList
}

class DatabaseStore extends DataStore[Env] {
  override def save(data: List[Int]): Reader[Env, Unit] =
    for {
      env <- Reader.ask[Env]
    } yield env.database.runSql(
      s"insert into data_table values(${data.mkString(",")})"
    )
}
```

The DataJob become

```scala
class DataJob {
  def run: Reader[Env, Unit] =
    for {
      env <- Reader.ask[Env]
      data <- env.source.getData
      val encodedData = env.encoder.encode(data)
      _ <- env.store.save(encodedData)
    } yield ()
}
```

Now we can instantiated all the components separately

```scala
object Main {
  def main() {

    val http = new LogHttpRequest()
    val database = new LogDatabase()
    val source = new HttpDataSource
    val store = new DatabaseStore
    val encoder = new PlusOneEncoder()
    val program = new DataJob

    val env = Env(http, database, source, store, encoder)
    program.run.run(env)
  }
}
```

You can find the full code in [reader-pattern-all-in-context.scala](https://gist.github.com/sjmyuan/2e16726a34b2b6b329954f03284a7afc)

Most of the code looks better now

* Don't need to modify HttpDataSource and DatabaseStore.
* Add one line `env <- Reader.ask[Env]` in DataJob, but we don't need to care about class parameter anymore.
* Components don't need to depend on each other explicitly
* The main is simpler, just instantiated all the components and put them into Env.

But we need to pay the price

* There are circular type dependency in Env, which is harder to understand
* No restriction to use the memebers of Env, which is easy to make mistake. 

  For example, we can even invoke `store.save` in the implementation

  ```scala
  class DatabaseStore extends DataStore[Env] {
    override def save(data: List[Int]): Reader[Env, Unit] =
      for {
        env <- Reader.ask[Env]
        _ <- env.store.save(data) // recursive call
      } yield env.database.runSql(
        s"insert into data_table values(${data.mkString(",")})"
      )
  }
  ```
* The Env will become bigger and bigger, which is hard to maintain
* The dependency graph is not clear, we need to dig into the code to find out it.

Seems the price is too high,
we think context with minimum dependencies is better,
we'd better only put the components without any dependency into the context.

# Refinement

Based on context with minimum dependencies, we still have two pain points

* Even we only put components without dependency into context, we still inject too many things.
  If someone call other dependencies, we can't know easily except digging into the code.
  Could we just inject what we need?

* When we do unit test for components, such as HttpDataSource and DatabaseStore, we have to prepare all the data of context, but we just want to use part of them.

Let's recall the [Implicit](https://blog.shangjiaming.com/scala%20tutorial/implicit/) usage scenarios, it can apply restriction to the type parameter.
Since we don't want to inject the whole context, then let's use `type parameter + implicit` to filter the dependencies we need.

Imagine there is a type `A`, we expect it has a variable `http: HttpRequest`, how do we apply the restriction?
Right, we can require a function `A => HttpRequest` together with `A`

```scala
def getData[A](a: A)(getHttpRequest: A => HttpRequest) = 
  getHttpRequest(a).get("http://example.com/data").split(",").map(_.toInt).toList
```

But the signature of this function is too common, it can not highlight our purpose, let's use [Type Class](https://blog.shangjiaming.com/scala%20tutorial/type-classes/) to make it more targeted.

```scala
trait HasHttpRequest[A]{
  def getHttpRequest(a: A): HttpRequest
}

def getData[A: HasHttpRequest](a: A) = 
 implicitly[HasHttpRequest[A]].getHttpRequest(a).get("http://example.com/data").split(",").map(_.toInt).toList
```

Then for Env we can implement the type class to get http request

```scala
object Env {
  implicit object EnvHasHttpRequest extends HasHttpRequest[Env] {
    override def getHttpRequest(v: Env): HttpRequest = v.http
  }
}
```

Compared to `def getData(a: Env)`, `def getData[A: HasHttpRequest](a: A)` don't care about the real type of `A`,
It just require `A` implement the `HasHttpRequest` type class.

Use this way, the components will only know the dependencies which are declared to access and we just need to mock the data we need in test

```scala
case class MockEnv(http: HttpRequest)
object MockEnv {
  implicit object MockEnvHasHttpRequest extends HasHttpRequest[MockEnv] {
    override def getHttpRequest(v: MockEnv): HttpRequest = v.http
  }
}
```

You can find the full code in [reader-pattern.scala](https://gist.github.com/sjmyuan/d01e0cd978822858b5ec8b087b6b0bdc)

# Summary

Reader Pattern can do dependency injection easily and it can restrict the dependency scope very well.
But we can't put all the dependencies into context, it will be harder and harder to maintain with code growing. 
It's fine to just put the dependencies without dependency into context, which minimise the probability of mistake.
