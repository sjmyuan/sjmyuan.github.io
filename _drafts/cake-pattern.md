---
title: Cake Pattern
tags:
  - Scala
categories:
  - Scala Tutorial
---
There were some day to day pains for us in using the Cake Pattern. 
The obvious first one was the need for all the boilerplate `*Component` and `*ComponentImpl` code for every service and repository, 
which is tedious to write and read.

The second and bigger pain was that when wiring (cooking?) up all the layers of the cake 
in the controllers (or in controller or service unit tests), 
it was necessary to explicitly include not just the controller’s direct dependencies (typically just a single service), 
but rather all of the controller’s transitive dependencies.

I have a large codebase that I made into a cake years ago.
I'm gradually moving away from it but it's not so easy.
Here are some issues with it:1.
It's viral: the more things need to be in the cake, the more other things need to be as well.    
2.  Since your dependencies are not a list of imports but a composite self-type,
when you refactor and make something less coupled it's very hard to tell which dependencies you still need. 
As a result you often depend on more things than you need.
With parameters you can see what classes are imported and when you refactor the IDE can remove unused imports or the compiler can tell you about them.

The cake pattern is in theory pretty neat, being able to do DI without any libraries, however:*   It can make compile times a lot longer    
*   It can be hard to refactor as it is not immediately clear what layers of the cake need to be added or deleted when changing your dependency setup.    
*   It is hard to escape the cake. You can only solve dependencies by mixing them in, which cannot be done partially, as that would result in compile-errors. 

So in the end you need to build an enormous cake in your main to get it to work.

# A Requirement

One day, Product tell us they want to save some data to somewhere

Q: What type of data?
A: An integer array

Q: Where does the data come from?
A: For now, one RESTful API will supply the data, may be changed in the future.

Q: Where do you want to store the data?
A: Haven't decided, maybe a database, maybe s3 bucket, we can store it to a database temporarily.

Q: Is there any security requirements?
A: Yes, we can only store the encrypted data

There are more details we need to know, but the above information is enough for us to build a simplified program,
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

So the left things are

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

Then we need to wire the component explicitly in main function

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
We can use it to do Dependency Injection here.

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

Then we need to inject all dependency when instantiated DataJob

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

## Module Pattern

Based on the Cake Pattern solution, what if we want to rename Database.runSql to Database.run and pass a notes to DataJob.run?

```scala
trait DatabaseStore extends DataStore {
  self: Database =>

  override def save(data: List[Int]): Unit =
    run(s"insert into data_table values(${data.mkString(",")})")

}

trait DataJob {
  self: DataSource with DataStore with DataEncoder =>

  def run(notes): Unit = {
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

We can see this code can not compile

# New Requirement

# Summary
