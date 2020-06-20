---
title: Cake Pattern
tags:
- Scala
categories:
- Scala Tutorial
date: 2020-06-20 21:37 +0800
---
Cake Pattern is the inborn Dependency Injection in Scala.

After lots of practice, more and more people moved to other patterns,
but there are still some excellent frameworks adopting this pattern, such as [ZIO](https://zio.dev/).

It's still worth to know it.

# A Requirement

One day, Product tell us they want to store some data to somewhere

>Q: What type of data?  
>A: An array of integer
>
>Q: Where does the data come from?  
>A: For now, a RESTful API can supply the data, may be changed in the future.
>
>Q: Where do you want to store the data?  
>A: Haven't decided, maybe a database, maybe s3 bucket, we can store it to database temporarily.
>
>Q: Is there any security requirement?  
>A: Yes, we can only store the encrypted data

There are more details we need to know, but these are enough for us to build a simplified program,
let's try to do it now.

# Implementation

Obviously we at least need three components: DataSource, DataStore, DataEncoder

```scala
trait DataSource {
  def getData: List[Int]
}

trait DataStore {
  def save(data: List[Int]): Unit
}

trait DataEncoder {
  def encode(data: List[Int]): List[Int]
}
```

Product mentioned we can get data from RESTful API and store data in database temporarily, so we need two more components

```scala
trait HttpRequest {
  def get(url: String): String
}

trait DataStore {
  def save(data: List[Int]): Unit
}
```

To simplify the program, we can give default implementation to these two components which will just print log.

```scala
trait LogHttpRequest extends HttpRequest {
  override def get(url: String): String = {
    println(s"send request to ${url}")
    List(1, 2, 3, 4, 5, 6).mkString(",")
  }
}

trait LogDatabase extends Database {
  override def runSql(sql: String): Unit = println(s"run sql ${sql}")
}
```

For DataEncoder, we can also give it a default implementation 

```scala
trait PlusOneEncoder extends DataEncoder {
  def encode(data: List[Int]): List[Int] = {
    println(s"encoding ${data}")
    data.map(_ + 1)
  }
}
```

Now we expect the main component can access DataSource, DataStore and DataEncoder, then implement the main logic

```scala
  def run: Unit = {
    val data = source.getData
    val encodedData = encoder.encode(data)
    store.save(encodedData)
  }
```

The left things are

* How to let DataSource know HttpRequest to get data from the RESTful API?
* How to let DataStore know Database to store the data to database?
* How to let the main component know DataSource, DataStore and DataEncoder to implement the main logic?

Actually these are saying same thing: How let one component know its dependency? or How to do Dependency Injection?

## Constructor Pattern(Classic Solution)

The intuitive solution is just pass the dependency to the component's constructor

```scala
class HttpDataSource(http: HttpRequest) extends DataSource {
  override def getData: List[Int] =
    http.get("http://example.com/data").split(",").map(_.toInt).toList
}

class DatabaseStore(database: Database) extends DataStore {
  override def save(data: List[Int]): Unit =
    database.runSql(s"insert into data_table values(${data.mkString(",")})")
}

class DataJob(source: DataSource, store: DataStore, encoder: DataEncoder) {
  def run: Unit = {
    val data = source.getData
    val encodedData = encoder.encode(data)
    store.save(encodedData)
  }
}
```

Then we need to wire components explicitly in main function

```scala
object Main {
  def main() {

    val http = new LogHttpRequest {}
    val source = new HttpDataSource(http)

    val database = new LogDatabase {}
    val store = new DatabaseStore(database)

    val encoder = new PlusOneEncoder {}

    val program = new DataJob(source, store, encoder)

    program.run
    //send request to http://example.com/data
    //encoding List(1, 2, 3, 4, 5, 6)
    //run sql insert into data_table values(2,3,4,5,6,7)
  }
}
```

## Cake Pattern

We already introduced [Self-types](https://blog.shangjiaming.com/scala%20tutorial/self-type/) in other blog.
It can be used to do Dependency Injection here.

```scala
trait HttpDataSource extends DataSource {
  self: HttpRequest =>

  override def getData: List[Int] =
    get("http://example.com/data").split(",").map(_.toInt).toList
}

trait DatabaseStore extends DataStore {
  self: Database =>

  override def save(data: List[Int]): Unit =
    runSql(s"insert into data_table values(${data.mkString(",")})")

}

trait DataJob {
  self: DataSource with DataStore with DataEncoder =>

  def run: Unit = {
    val data = getData
    val encodedData = encode(data)
    save(encodedData)
  }
}
```

Then we need to mix in all Traits when instantiated DataJob

```scala
object Main {
  def main() {
    val program = new DataJob
        with LogHttpRequest
        with DatabaseStore
        with HttpDataSource
        with PlusOneEncoder
        with LogDatabase {}

    program.run
    //send request to http://example.com/data
    //encoding List(1, 2, 3, 4, 5, 6)
    //run sql insert into data_table values(2,3,4,5,6,7)
  }
}
```

## Cake Pattern V2

Based on the Cake Pattern, what if we want to rename Database.runSql to Database.run and pass a notes to DataJob.run?

```scala
trait DatabaseStore extends DataStore {
  self: Database =>

  override def save(data: List[Int]): Unit =
    run(s"insert into data_table values(${data.mkString(",")})")

}

trait DataJob {
  self: DataSource with DataStore with DataEncoder =>

  def run(notes: String): Unit = {
    println(notes)
    val data = getData
    val encodedData = encode(data)
    save(encodedData)
  }
}

object Main {
  def main() {
    val program = new DataJob
        with LogHttpRequest
        with DatabaseStore
        with HttpDataSource
        with PlusOneEncoder
        with LogDatabase {}

    program.run("This is Cake Pattern solution")
    //cake-pattern.scala:67: overriding method run in class DataJob of type (notes: String)Unit;
    // method run in trait LogDatabase of type (sql: String)Unit cannot override a concrete member without a third member that's overridden by both (this rule is designed to prevent ``accidental overrides'')
    //    val program = new DataJob with HttpDataSource with PlusOneEncoder with DatabaseStore with LogHttpRequest with LogDatabase
    //                      ^
    //Compilation Failed
  }
}
```

Oops, We got a compile error. 

Obviously the program is just an instance mixing in a set of Traits. But this set of Traits may have same variable/function which will override each other and make the code hard to understand, then we have to be careful to develop the components to avoid same variable/function.

The root cause here is the content of Trait is mixed in instance flatly, we can't identify the source of variable/function in the instance. 

To solve this problem, we can put the content of Trait to a variable, and mix in this variable to instance, then we just need to ensure each Trait(Component) have different variable name, which will be eaisier than above.

For HttpRequest, we can mix in it to DataSource like this

```scala
trait DataSourceComponent {
  val source: DataSource
  trait DataSource {
    def getData: List[Int]
  }
}

trait HttpRequestComponent {
  val http: HttpRequest
  trait HttpRequest {
    def get(url: String): String
  }
}

trait LogHttpRequestComponent extends HttpRequestComponent {

  val http: HttpRequest = new LogHttpRequest {}

  trait LogHttpRequest extends HttpRequest {
    override def get(url: String): String = {
      println(s"send request to ${url}")
      List(1, 2, 3, 4, 5, 6).mkString(",")
    }
  }
}

trait HttpDataSourceComponent extends DataSourceComponent {
  self: HttpRequestComponent =>

  val source: DataSource = new HttpDataSource {}

  trait HttpDataSource extends DataSource {
    override def getData: List[Int] =
      http.get("http://example.com/data").split(",").map(_.toInt).toList
  }
}
```

We wrap HttpRequest with HttpRequestComponent and define a variable http in the component.

When we mix in HttpRequestComponent, the HttpDataSourceComponent can only get a HttpRequest variable named http, which will be eaiser for us to invoke correct function.

```scala
http.get("http://example.com/data").split(",").map(_.toInt).toList // code in cake pattern v2
get("http://example.com/data").split(",").map(_.toInt).toList // code in cake pattern
```

You can get all the code in [cake-pattern-v2.scala](https://gist.github.com/sjmyuan/f219bcdd2b123d7d0c16c3aa27e8c30e)

# Summary

## Highlight
The main difference between Cake Pattern and Constructor Pattern are

* Component Implementation

  * Constructor Pattern doesn't have any restriction, can be Class or Trait, just follow the OO design.
  * Cake Pattern have to follow a template
    ```scala
    trait Component {
      val component: ComponentInterface
      trait ComponentInterface {
        .....
      }
    }

    trait ComponentImpl extends Component {
      val component: ComponentInterface = new ComponentInterfaceImpl {}
      trait ComponentInterfaceImpl extends ComponentInterface {
       ....
      }
    }
    ```
* Dependency Injection

  * Constructor Pattern inject dependency by parameters

    ```scala
    class DataJob(source: DataSource, store: DataStore, encoder: DataEncoder) {
     ....
    }
    ```

  * Cake Pattern inject dependency by Self-types
    ```scala
    class DataJob {
      self: DataSource with DataStore with DataEncoder =>
      ....
    }
    ```

* Components Wiring

  * Constructor Pattern wire components by normal Class instantiation

    ```scala
    val http = new LogHttpRequest {}
    val source = new HttpDataSource(http)

    val database = new LogDatabase {}
    val store = new DatabaseStore(database)

    val encoder = new PlusOneEncoder {}

    val program = new DataJob(source, store, encoder)
    ```
  * Cake Pattern wire components by Trait mixin

    ```scala
    val program = new DataJob
        with LogHttpRequest
        with DatabaseStore
        with HttpDataSource
        with PlusOneEncoder
        with LogDatabase {}
    ```

## What's the problem of Cake Pattern?

[Cake antipattern](https://kubuszok.com/2018/cake-antipattern/) give a very detailed discussion about the problem of Cake Pattern, I just give a summary here.

* Component Implementation

  We may have more than 20 components in our application, there will be lots of boilerplate which is noisy.

* Dependency Injection

  When there are lots of components in different files, We may involve cyclic dependency, but the compiler won't warn us until we run the program.

  ```scala
  trait AComonent {
    def runA: Unit
  }

  trait AComonentImpl extends AComonent {
    self: BComponent =>
      def runA:Unit = {
        println("running A")
        runB
      }
  }

  trait BComponent {
     def runB: Unit
  }

  trait BComponentImpl extends BComponent {
    self: AComonent => 
      def runB:Unit = {
        println("running B")
        runA
      }
  }

  object Main {
    def main():Unit = {
      val program = new AComonentImpl with BComponentImpl {}
      program.runA
    }
  }
  ```

  We even can't find the issue by test, our mocked component may not have cyclic dependency.

  ```scala
  trait BComponentMock extends BComponent {
    def runB:Unit = {
      println("mocked running B")
    }
  }

  val testA = new AComonentImpl with BComponentMock {}
  ```

* Component Wiring

  We can't get a clear dependency graph from the Trait mixin, could you imagine there are more than 20 Traits mixing in?

  ```scala
    val program = new ComponentImpl1
        with ComponentImpl2
        with ComponentImpl3
        with ComponentImpl4
        with ComponentImpl5
        ...
        with ComponentImplN {}
  ```

  According to this code, we don't know how components depend on each other, and there is no easy way to split the component wiring to small pieces.

  What if we miss one component in the above code? we will get the error

  ```scala
  self-type X does not conform to Y
  ```

  If there is only 3 or 4 components in the application, we can figure out it eaisily. How about 50? We have to review the code line by line to see what we missed.


## Suggestion

For small application, Cake Pattern works well. 
But for a large application which may have 10 or even more components, we'd better choose other pattern to make it easy to understand and maintain.
