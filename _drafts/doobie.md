---
title: Doobie Introduction
tags:
- Scala
categories:
- Scala Tutorial
---

Database is always a thing for developer to consider, there are lots of tools in other languages. 
In this blog, Let's introduce a tool for Scala developer: [doobie](https://github.com/tpolecat/doobie)

# What is doobie?

> doobie is a pure-functional JDBC layer for Scala.

> doobie provides low-level access to everything in java.sql (as of Java 8), allowing you to write any JDBC program in a pure functional style.

We can say doobie is just a FP wrapper of JDBC, it just help you translate the FP style code to normal JDBC code.

# How to install?

Add the following configuration into your `build.sbt`

```scala
scalacOptions += "-Ypartial-unification" // 2.11.9+

libraryDependencies ++= Seq(

  // Start with this one
  "org.tpolecat" %% "doobie-core"      % "0.8.8",

  // And add any of these as needed
  "org.tpolecat" %% "doobie-h2"        % "0.8.8",          // H2 driver 1.4.200 + type mappings.
  "org.tpolecat" %% "doobie-postgres"  % "0.8.8",          // Postgres driver 42.2.9 + type mappings.
  "org.tpolecat" %% "doobie-hikari"    % "0.8.8",          // HikariCP transactor.
  "org.tpolecat" %% "doobie-quill"     % "0.8.8",          // Support for Quill 3.4.10
  "org.tpolecat" %% "doobie-specs2"    % "0.8.8" % "test", // Specs2 support for typechecking statements.
  "org.tpolecat" %% "doobie-scalatest" % "0.8.8" % "test"  // ScalaTest support for typechecking statements.
)
```

`doobie-core` is the essential dependency.

`doobie-h2` and `doobie-postgres` are the JDBC driver, please choose them as needed

`doobie-hikari` supply another implementation of `Transactor` which support to manage the connection pool.

From version 0.7, `doobie-quill` can help you generate sql from your model which is more safe and easy to maintain, but you can still compose sql by raw string.

`doobie-specs2` and `doobie-scalatest` are for test, choose them according to your test framework.

# Core Concept

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1gdtpn4uxfkj30un0lmgpa.jpg)

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1gdtpn7h95vj30qm0lcjuz.jpg)

In simple terms doobie translate all the model under `java.sql` to a corresponding `Free Monad`, and use these `Free Monad` to compose program, then interpret the program to real `java.sql` as needed. 

## ConnectionIO

The `Free Monad` of `java.sql.Connection`, all the program of doobie will become `ConnectionIO` finally.

All the `ConnectionIO` chained by `map` or `flatMap` will be run in the same transaction.

```scala
type ConnectionIO[A] = Free[ConnectionOp, A]

trait ConnectionOp[A]

final case object Commit extends ConnectionOp[Unit]
final case object CreateStatement extends ConnectionOp[Statement]
final case class  PrepareStatement(a: String) extends ConnectionOp[PreparedStatement]
.....
```

## Fragment

In Java, we use the lieral string to define sql statement and use `?` as the placeholder of parameter which can be supplied laterly. for example

```java
PreparedStatement updateAge = null;

String updateString = "update table person set age = ? where name = ?";

updateAge = con.prepareStatement(updateString);

updateAge.setInt(1, 18);
updateAge.setString(2, "Tom");

updateAge.executeUpdate();
```

To get a completed `PreparedStatement`, we need to prepare 3 things

1. The sql template
2. The parameter posiontion 
3. The parameter type

We don't know if the sql is correct before runing it in Java.

To make it easier, doobie defined `Fragment` which can apply type level checking when we prepare the sql and we don't need to remember the parameter position and type anymore. for example

```scala
val age: Int =18
val name: String = "Tom"
val sql = sql"update table person set age = $age where name = $name"
sql.update.run.unsafeRunSync
```

Here the `sql` interpolator will help us construct `Fragment` which will store the parameter information.

## Query and Update

In Java, we run query or update by invoking different method of `PreparedStatement`, for example

```java

PreparedStatement selectPerson = connection.prepareStatement("select name from person");

ResultSet rs = selectPerson.executeQuery()

while (rs.next()) {
    String name = rs.getString("name");
    System.out.println("name: " + name);
}

PreparedStatement updateAge = connection.prepareStatement("update table person set age = 18");
selectPerson.executeUpdate()
```

We can see the process of query and update are different, so doobie supply two differnt models for them: `Query` and `Update`

### Query

In `Query` we can control the expected type of query result which can even let us apply more flexible checking on the number of result.

* unique
* option
* to

### Update

## Transactor

A Transactor is a data type that knows how to connect to a database, hand out connections, and clean them up; and with this knowledge it can transform ConnectionIO ~> I

# Usage

## How to connect to a database?

To connect to a database, we need to add the corresponding database dependency in our configuration.
And pass the JDBC driver name to `Transactor`.

```scala
implicit val cs = IO.contextShift(ExecutionContexts.synchronous)
val xa = Transactor.fromDriverManager[IO](
  "org.postgresql.Driver",     // driver classname
  "jdbc:postgresql:world",     // connect URL (driver-specific)
  "postgres",                  // user
  ""                          // password
)
```

## How to run sql?

1. Use `sql` to construct a statement 
2. Generate a `Query` or `Update`
3. Generate `ConnectionIO`
3. Pass `ConnectionIO` to `Transactor` to geneate `IO`
4. Run the `IO`

```scala
val program: ConnectionIO[Int] = sql"select 42".query[Int].unique
val io:IO[Int] = program2.transact(xa)
io.unsafeRunSync
```
## How to run a query?

* Single Column

  ```scala
  sql"select name from country"
  .query[String]    // Query0[String]
  .stream           // Stream[ConnectionIO, String]
  .take(5)          // Stream[ConnectionIO, String]
  .compile.toList   // ConnectionIO[List[String]]
  .transact(xa)     // IO[List[String]]
  .unsafeRunSync    // List[String]
  ```

* Multiple Column

  ```scala
  sql"select code, name, population, gnp from country"
    .query[(String, String, Int, Option[Double])]
    .stream
    .take(5)
    .quick
    .unsafeRunSync
  ```

* Custom Model

  ```scala
  case class Country(code: String, name: String, pop: Int, gnp: Option[Double])
  sql"select code, name, population, gnp from country"
    .query[Country]
    .stream
    .take(5)
    .quick
    .unsafeRunSync
  ```

## How to insert a record?

## How to update a record?

## How to delete a record?


## How to map column to model?

## How to map model to column?

## How to map record to model?

## How to map model to record?

## How to manage the connections?

## How to do test?

# Summary
