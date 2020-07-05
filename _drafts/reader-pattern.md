---
title: Reader Pattern
tags:
- Scala
categories:
- Scala Tutorial
---

We already introduced [Reader Monad](https://blog.shangjiaming.com/scala%20tutorial/reader-monad/),
which can carry some information everywhere and supply a solution of global variable/function in FP. 

What if we put dependencies into Reader Monad as something like global context?
In this blog, let's use Reader Monad to implement the requirement in [Cake Pattern](https://blog.shangjiaming.com/scala%20tutorial/cake-pattern/)

# Global Context

Not like transaction in [Reader Monad](https://blog.shangjiaming.com/scala%20tutorial/reader-monad/),
we need to pass more than one dependency here, which can't be done in Reader Monad.
So we need to create a new type to contain all the dependencies.

```scala
case class Env(http: HttpRequest, database: Database)
```

To use Reader Monad, we also need to modify the method of DataSource and DataStore, let them return Reader Monad.

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

And the dependencies can be wired in main like this

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

At least, except DataJob we can instantiated other module separately.

Maybe you will ask, could we also put source, store and encoder into env? then DataJob can be free(Doesn't have direct dependency)

Let's try

```scala
case class Env[A](http: HttpRequest, database: Database, source: DataStore[A], store: DataStore[A], encoder: DataEncoder)
```

DataStore and DataSource need a type parameter here, so we add a type parameter to Env. Then our implementation could be

```scala
class HttpDataSource extends DataSource[Env[???]] {
  override def getData: Reader[Env, List[Int]] =
    for {
      env <- Reader.ask[Env]
    } yield env.http.get("http://example.com/data").split(",").map(_.toInt).toList
}

class DatabaseStore extends DataStore[Env[???]] {
  override def save(data: List[Int]): Reader[Env, Unit] =
    for {
      env <- Reader.ask[Env]
    } yield env.database.runSql(
      s"insert into data_table values(${data.mkString(",")})"
    )
}

class DataJob {
  def run: Reader[Env[???], Unit] =
    for {
      env <- Reader.ask[Env[???]]
      data <- env.source.getData
      val encodedData = env.encoder.encode(data)
      _ <- env.store.save(encodedData)
    } yield ()
}
```

We use `Env[???]` for every type parameter, because we don't know the final type of `Env[_]`.
It may looks like `Env[Env[Env[Env[....]]]]`, which is a dead loop.

Maybe we can hard code the type parameter in Env

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

And the DataJob is free now

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
* Only add one line `env <- Reader.ask[Env]` in DataJob, but we don't need to care class parameter anymore.
* Modules don't need to know each other until the last two line

But we need to pay the price

* There are circular type dependency in Env, which is harder to understand
* No restriction to use the memeber of Env, which may involve big issue in the future. 

  For example, we can even invoke `store.save` in the implementation of `store.save`

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

Seems the price is too high, we think the first solution is better, we'd better only put the module without any dependency to the Env.

# Refinement

Based on the first solution in the last section, we still have two pain points

* Even we only put module without dependency to Env, we still inject too much things, if someone call other dependencies, we can't know easily except digging into the code.
  Could we just inject what we need?
* When we do test for some module, such as HttpDataSource and DatabaseStore, we have to prepare all the data of Env, but we just want to use part of them.

Let's recall the [Implicit](https://blog.shangjiaming.com/scala%20tutorial/implicit/) usage, it can restrict the type parameter.
Since we don't want to inject the whole Env, then let's use type parameter + implicit to apply some restriction.

Imagine we have a type `A`, we expect it have a variable `http: HttpRequest`, how do we apply the restriction?
Right, we can require a function `A => HttpRequest` together with `A`

```scala
def getData[A](a: A)(getHttpRequest: A => HttpRequest) = 
  getHttpRequest(a).get("http://example.com/data").split(",").map(_.toInt).toList
```

The usage of function is too common, it can not highlight our purpose, let's use type class to make it more targeted.

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

Use this way, the module will only know the dependency it declare to access and we just need to mock the data we need in test

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
But we can't put all the dependencies in the context, it will be harder and harder to maintain with code growing. 
It's fine to just put the dependency without other dependency in context, which minimise the probability of mistake.
