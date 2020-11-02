---
title: Practice in Real Project
tags:
  - Scala
categories:
  - Scala Tutorial
---

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

## Enforce style check

## Optimise compiler option

A good set of compiler options can let us find lots of potential issue in compile process,
you can find the best practice in [scalac-flags](https://tpolecat.github.io/2017/04/25/scalac-flags.html)

## Other useful plugins

### FP

In FP code, sometimes we want to do some type projection to get partial applied type.
for example, our functions just return error or normal value, then we can define a return type for all function.

```scala
type AppErrorOr[A] = Either[Throwable, A]
```

But what if we have a function want to return the left of Either?

```scala
def getLeft[E](either: ???, default: E): E
```

Both the type of Left and Right are not fixed, it's not possible to define a partial applied type for `getLeft`. 

Here we need the type to be defined anonymously, lucklily Scala support it

```scala
def getLeft[E](either: ({type L[A] = Either[E, A]})#L, default: E): E
```

It's pretty hard and ugly to use this syntax, plugin [kind-projector](https://github.com/typelevel/kind-projector) give a better implmentation.

```scala
def getLeft[E](either: Either[E, *], default: E): E
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

Add `docker.sbt` which is used to build docker image for our app which is just like a Dockerfile written by Scala.

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

We tried Cake Pattern, Eff and Tagless Final in our projects, the winner is Tagless Final.
But it's still hard to manage the dependency injection, we are trying to use Tagless Final and ReaderT pattern together.

In the future, we may try ZIO which is a combination of Tagless Final and ReaderT pattern.

## Framework

### FP

Definitly [cats](https://github.com/typelevel/cats) 

### Http

### JSON

### Database Connection

### Database Migration

## Code

### No var

### No null

### No Any

### No return

### Less if-else

### Use for instead of nested map/flatMap

### Declare type explicitly if possible

### Use implicts cautiously

### Use pattern-matching instead of fold

### Use sealed if possible

### Use trait group implict instances and inject then into companion object

### Use case class instead of class if possible

### Use by-name assginment for case class

### Don't use Option.get, List.head, Either.get, Try.get

### Use F[_], G[_], A, B, C as type parameter

### Use tail-recursive annotation

# Test

## Framework

## Mock

## Test Coverage

# Tips
