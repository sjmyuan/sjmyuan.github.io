---
title: jlogger
tags:
  - Scala
categories:
  - Project
---

# TLDR

I developed a scala log library [jlogger](https://github.com/sjmyuan/jlogger), you can add the following line to your `build.sbt` to intall it

```scala
  libraryDependencies += "io.github.sjmyuan" %% "jlogger" % "0.0.1",
```

## Features
* Type safe and based on [cats](http://typelevel.org/cats/)
* Support log level
* Support maximum 5 arbitrary data attributes
* Support JSON and String format
* Support logback

# What

Print log without any library

```scala
println("Something went wrong, the error message is ...")
```

Make it pure and testable
```scala
IO(println("Something went wrong, the error message is ..."))
Sync.delay(println("Something went wrong, the error message is ..."))

trait Logger[M[_]: Sync] {
  def log(message: String): M[Unit] =
       Sync.delay(println("Something went wrong, the error message is ..."))
}
```


Support log level and timestamp

```scala
sealed trait LogLevel

object LogLevel {
  case object INFO extends LogLevel
  case object WARNING extends LogLevel
  case object ERROR extends LogLevel
}

trait Logger[M[_]: Sync] {
  def log(logLevel: LogLevel, message: String): M[Unit] =
       Sync.delay(println(s"${logLevel}: ${message}"))
}
```

Print log in JSON format

```scala
import io.circe.Json
trait Logger[M[_]: Sync] {
  def log(logLevel: LogLevel, message: String): M[Unit] ={
        val body = Json.obj("level" -> logLevel.toString.asJson, "message" -> message.asJson)
		Sync.delay(println(body.noSpaces))
  }
}
```

Support any data attribute type
```scala
import io.circe.Json
import io.circe.Encoder
trait Logger[M[_]: Sync] {
  def log[A1: Encoder, A2: Encoder](logLevel: LogLevel, attr1:String -> A1, attr2: String -> A2): M[Unit] ={
        val body = Json.obj(attr1._1 -> attr1._2.asJson, attr2._1 -> attr2._2.asJson,)
		Sync.delay(println(body.noSpaces))
  }
}
```

Support different ways to print log
```scala
// TODO
```

Support different log format
```scala
//TODO
```



# Why
# How

How to install the library?

```scala
//TODO
```