---
title: Practice in Real Project
tags:
  - Scala
categories:
  - Scala Tutorial
---

In this blog, I will share some practice in our project, hope this can help you.

# Project Management

## Specify the sbt version

We need the sbt version of given project to be always same on different device,
then we can ensure the compiled jar of application are same between local development, release package and production.

Like `.ruby-version`, `.python-version`, sbt can also specify the sbt version in `project/build.properties`

```scala
sbt.version=1.4.1
```

It doesn't matter what's the sbt version in your local environment,
sbt will always download the specified version for the given project.

## Speed up dependency download

sbt will download the dependencies in sequence which is very slow, it may take half hour.

To speed up this process, we can use the plugin `sbt-coursier`.

To also speed up the plugin download, add the following line in `project/project/plugins.sbt`

```scala
addSbtPlugin("io.get-coursier" % "sbt-coursier" % "2.0.0-RC6-8")
```

Add the following line in `project/plugins.sbt`

```scala
addSbtCoursier
```

## Apply code formatter 

It's necessary to use unified code style across all team members, all IDE and all device.

Add plugin [scalafmt](https://github.com/scalameta/scalafmt)

```scala
addSbtPlugin("org.scalameta" % "sbt-scalafmt" % "2.4.0"
```

Add configuration `.scalafmt.conf`

```scala
version = 2.7.5
```

Check the code style before test

```scala
addCommandAlias("styleCheckAndTest", ";clean;scalafmtCheck;test")
```

Format code

```sh
sbt scalafmtAll
```

Use scalafmt in IntellJ IDEA

![](https://images.shangjiaming.com/3529c8fa-5197-4a56-92a4-491197285da1.jpeg)

## Optimise compiler option

A good set of compiler options can let us find lots of potential issue in compile process,
you can find the best practice in [scalac-flags](https://tpolecat.github.io/2017/04/25/scalac-flags.html)

## Other useful plugins

### FP

In FP code, sometimes we want to do some type projection to get partial applied type.

For example, We want all functions to return error or normal value, then we can define an unified return type.

```scala
type AppErrorOr[A] = Either[Throwable, A]
```

But what if we have a function like this

```scala
def identity[A, F[_]](value: F[A]): F[A] = value 
```

And we want to apply it to `Either`

```scala
val either:Either[String, Int] = Right(1)

identity(either) // can't be compiled
```

Compiler tell us we need to give the type parameter explicitly.
And we found it requires kind F[_], but Either is F[_, _].
It's not possible to define a partial applied type explicitly every time.
Here we need the type to be defined anonymously, lucklily Scala support it

```scala
identity[Int, ({type L[A] = Either[String, A]})#L](either) // success
```

It's pretty hard and ugly to use this syntax, plugin [kind-projector](https://github.com/typelevel/kind-projector) give a better implmentation.

```scala
identity[Int, Either[String, *]](either)
```

> Note: kind-projector involve * in this [PR](https://github.com/typelevel/kind-projector/pull/91) to support Scala 3.0, you can still use ?.

To use this plugin, add the following line into `build.sbt`

```scala
addCompilerPlugin("org.typelevel" %% "kind-projector" % "0.11.0" cross CrossVersion.full)
```

### Package

[sbt-native-packager](https://github.com/sbt/sbt-native-packager) can help us package our app, then build an docker image.

Add the following line in `project/plugins.sbt`

```scala
addSbtPlugin("com.typesafe.sbt" % "sbt-native-packager" % "1.7.6")
```

Add the following line in `build.sbt` to enable `JavaAppPackaging` and `DockerPlugin`

```scala
enablePlugins(JavaAppPackaging)
```

> Note: JavaAppPackaging will enable DockerPlugin automatically

Add `docker.sbt` which is just like a Dockerfile written in Scala.

```scala
import com.typesafe.sbt.packager.docker.Cmd

defaultLinuxInstallLocation in Docker := s"/opt/rea/apps/${name.value}"

version in Docker := scala.util.Properties.envOrElse("VERSION", "v0." ++ scala.util.Properties.envOrElse("BUILDKITE_BUILD_NUMBER", "DEV"))

dockerBaseImage := ???
dockerRepository := ???
packageName in Docker := ???

dockerCommands ++= Seq(
  Cmd("ENV", s"""JAVA_OPTS="-javaagent:${(defaultLinuxInstallLocation in Docker).value}/newrelic/newrelic.jar -Xms1024m -Xmx1024m""""),
  Cmd("RUN", s""" \\
                | echo "RUN Command" \\
  """.stripMargin)
)

dockerEntrypoint := ???
```

> Note: we can also put the content of docker.sbt in build.sbt, they will be merged when we run sbt command.

# Coding

## Pattern

We tried [Cake Pattern](https://blog.shangjiaming.com/scala%20tutorial/cake-pattern/),
[Eff](https://github.com/atnos-org/eff) and [Tagless Final](https://degoes.net/articles/zio-environment) in our projects, the winner is Tagless Final.
But it's still hard to manage the dependency injection, we are trying to use Tagless Final and [ReaderT pattern](https://www.fpcomplete.com/blog/2017/06/readert-design-pattern/) together.

In the future, we may try [ZIO](https://github.com/zio/zio) which is a combination of Tagless Final and ReaderT pattern.

## Framework

### FP

Definitely [cats](https://github.com/typelevel/cats)

Add the following line in `build.sbt` 

```scala
libraryDependencies += "org.typelevel" %% "cats-core" % "2.1.1"
```

### Http

[Http4s](https://github.com/http4s/http4s) supply both http server and client based on cats

```scala
libraryDependencies ++= Seq(
  "org.http4s" %% "http4s-blaze-server" % "0.21.8",
  "org.http4s" %% "http4s-blaze-client" % "0.21.8",
  "org.http4s" %% "http4s-circe" % "0.21.8",
  "org.http4s" %% "http4s-dsl" % "0.21.8"
)
```

### JSON

[Circe](https://github.com/circe/circe)

```scala
libraryDependencies ++= Seq(
  "io.circe" %% "circe-core" % "0.12.3",
  "io.circe" %% "circe-generic" % "0.12.3",
  "io.circe" %% "circe-parser" % "0.12.3"
)
```

### Database Connection

[Doobie](https://github.com/tpolecat/doobie)

```scala
libraryDependencies ++= Seq(
  "org.tpolecat" %% "doobie-core"      % "0.9.0",
  "org.tpolecat" %% "doobie-hikari"    % "0.9.0",          // HikariCP transactor.
  "org.tpolecat" %% "doobie-postgres"  % "0.9.0",          // Postgres driver 42.2.12 + type mappings.
  "org.tpolecat" %% "doobie-quill"     % "0.9.0",          // Support for Quill 3.5.1
  "org.tpolecat" %% "doobie-specs2"    % "0.9.0" % "test", // Specs2 support for typechecking statements.
)
```

### Database Migration

[Flyway](https://github.com/flyway/flyway)

```scala
libraryDependencies += "org.flywaydb" % "flyway-core" % "7.1.1"
```

## Code

### No var

We can modify a variable with var in any time without compiler warning,
definitely should not be used.

Create a new instance with modified value instead.

### No null

`null` can be assigned to any `AnyRef` variable.
For a variable with given type,
we don't know if it really store the value of given type or just `null`,
which will confuse the meaning of type.

Use Option if your variable need to store `null`

### No Any

Any type can be assigned to `Any`.
For a variable with `Any` type, we really don't know what value it store.
The code will be hard to read and maintain.

Use concrete type as much as possible, you don't need `Any`, trust me.

### No return

In function, Scala will treat the value of last expression as return value,
we don't need to return explicitly.

Can write less code, why not?

### Use for instead of nested map/flatMap

Bad

```scala
f1()
  .flatMap(x1 => 
    f2(x1).flatMap(x2 => 
      f3(x2).flatMap(x3 => 
        f4(x3).map(x4 => x4))))
```

Good

```scala
for {
  x1 <- f1()
  x2 <- f2(x1)
  x3 <- f3(x2)
  x4 <- f4(x3)
} yield x4
```

for expression is easier to read and maintain.

### Use implicts cautiously

`implicts` is a powerful tool, it is very easy to be abused.
Most of time, it is `implicts` which make the code hard to read and maintain.
It's also the biggest blocker for newbies to learn scala.

Don't use it if possible except you can prove you have to do that.

### Use pattern-matching instead of fold

Bad

```scala
val a:Option[Int] = ???
a.fold(
  for {
   x1 <- f1()
   x2 <- f2(x1)
  } yield x2
)(x => 
  for {
    x3 <- f3(x)
    x4 <- f4(x3)
  } yield x4
)
```

Good

```scala
val a:Option[Int] = ???
a match {
case None => 
  for {
   x1 <- f1()
   x2 <- f2(x1)
  } yield x2
case Some(x) => 
  for {
    x3 <- f3(x)
    x4 <- f4(x3)
  } yield x4
}
```

Most of time, pattern-matching will be easier to read.

### Use sealed if possible

For data type with sealed, compiler can help us to check if we cover all the branch

### Use trait group implict instances and inject them into companion object

For a data type `T`, we may have lots of implict instances, usually they can be grouped like this 

* Instances

  Define some implict instances used by other components. 

  ```scala
  implict decoder: Decoder[T] = ???
  implict ordering: Ordering[T] = ???
  ```

* Syntax

  Add more methods to the given type.

  ```scala
  implict class TAddOps(v: T) {
    def add(other: T): T = ???
  }

  implict class THttpOps(v: T) {
    def send: HttpResponse = ???
  }
  ```

We don't want to import a package every time to use the implict instances.
And according to the rule of implict, companion object is the fall back scope to find the implict instances of `T`.
So it make sense to put all of them into companion object.

We still want to group these instances better, so the pattern may look like this

```scala
trait TInstances {
  implict decoder: Decoder[T] = ???
  implict ordering: Ordering[T] = ???
}

trait TSyntax {
  implict class TAddOps(v: T) {
    def add(other: T): T = ???
  }

  implict class THttpOps(v: T) {
    def send: HttpResponse = ???
  }
}

object T extends TInstances with TSyntax
```

### Use case class instead of class if possible

`case class` is easier to be copied and can be used in pattern-matching.
Most of time, algebraic data type are composed by `trait` and `case class`.

### Don't use Option.get, List.head, Either.get and Try.get

These functions may throw exception and easy to be ignored,
There are safer function like `Option.getOrElse`, `List.headOption`, `Either.getOrElse` and `Try.getOrElse`.

### Use F[_], G[_], A, B, C as type parameter

Using a set of unified symbol as type parameter can make the code easier to read.

### Use @tail-recursive annotation

If we are writing a recursive function,
add `@tail-recursive` annotation,
then compiler can help us to check if it is really a tail recursive function.

# Test

## Framework

[specs2](https://github.com/etorreborre/specs2)

```scala
libraryDependencies ++= Seq("org.specs2" %% "specs2-core" % "4.10.0" % "test")
```

spec2 have two type of test style,
mutable style is similar to other language and also our preffered style.

```scala
import org.specs2.mutable.Specification
import org.specs2.specification.Scope
class HelloSpec extends Specification {
  "Hello" should {
    "should print hello" in new Scope {
      1.toString should_==("1")
    }
  }
}
```

## Mock

[mockito-scala](https://github.com/mockito/mockito-scala)

```scala
libraryDependencies ++= Seq(
  "org.hamcrest" % "hamcrest" % "2.2" % Test,
  "org.mockito" %% "mockito-scala-specs2" % "1.15.0" % Test,
)
```

specs2 also support mockito,
but it doesn't support function with default parameter and not good at fp.
[They also suggest us to use mockito-scala](https://github.com/etorreborre/specs2/issues/854#issuecomment-674999804).

## Test Coverage

[sbt-scoverage](https://github.com/scoverage/sbt-scoverage)

Add the following line in `project/plugins.sbt`

```scala
addSbtPlugin("org.scoverage" % "sbt-scoverage" % "1.6.1")
```

Generate coverage report when running test in `build.sbt`

```scala
addCommandAlias("TestWithCoverage", ";clean;coverage;test;coverageReport;coverageOff")
```

# Tips

* Stay in the sbt console to run compile/test repeatedly, which is faster.
* Use sbt instead of IntellJ IDEA to compile project, which will give more information.
* Declare type explicitly if you can not understand the error message
* Use [ammonite-repl](https://ammonite.io/) instead of scalac to run experiment code.
* Readable is more important than fantastic syntax.
