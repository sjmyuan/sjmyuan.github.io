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
  "org.tpolecat" %% "doobie-hikari"    % "0.8.8",          // HikariCP transactor.
  "org.tpolecat" %% "doobie-postgres"  % "0.8.8",          // Postgres driver 42.2.9 + type mappings.
  "org.tpolecat" %% "doobie-quill"     % "0.8.8",          // Support for Quill 3.4.10
  "org.tpolecat" %% "doobie-specs2"    % "0.8.8" % "test", // Specs2 support for typechecking statements.
  "org.tpolecat" %% "doobie-scalatest" % "0.8.8" % "test"  // ScalaTest support for typechecking statements.
)
```

`doobie-core` is the essential dependency.

`doobie-h2` and `doobie-postgres` are the JDBC driver, please choose them as needed

`doobie-quill` can help you generate sql from your model which is more safe and easy to maintain, but you can still compose sql by raw string.

`doobie-specs2` and `doobie-scalatest` are for test, choose them according to your test framework.

# Core Concept

## ConnectionIO

A free monad to represent all the operations of JDBC, usually we won't be aware of this model except some return type, doobie will help us compose program using it.

```scala
type ConnectionIO[A] = Free[ConnectionOp, A]

trait ConnectionOp[A]

final case object Commit extends ConnectionOp[Unit]
final case object CreateStatement extends ConnectionOp[Statement]
final case class  PrepareStatement(a: String) extends ConnectionOp[PreparedStatement]
.....
```

## Fragment

A fragment of sql statement, this is the most common type we will use.

But we usually won't construct the type directly, it supply some StringContext to help us do it, such as `sql`, `fr`, `fr0`.

```scala
val statment = sql"select * from example_table"
```

## Query0 and Update0

Used to generate the `ConnectionIO`

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

We can use `sql` to construct a statement, and generate `Query0` or `Update0`, then pass it to `Transactor`, finally run the IO to get result.

```scala
val program: ConnectionIO[Int] = sql"select 42".query[Int].unique
val io:IO[Int] = program2.transact(xa)
io.unsafeRunSync
```

## How to insert a record?

## How to update a record?

## How to delete a record?

## How to run a query?

## How to map column to model?

## How to map model to column?

## How to map record to model?

## How to map model to record?

## How to manage the connections?

## How to do test?

# Summary
