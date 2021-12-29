---
title: jlogger
tags:
  - Scala
categories:
  - Project
---

# TLDR

I developed a scala log library [jlogger](https://github.com/sjmyuan/jlogger), you can add the following line to your `build.sbt` to install it

```scala
  libraryDependencies += "io.github.sjmyuan" %% "jlogger" % "0.0.1",
```

## Features
* Type safe and based on [cats](http://typelevel.org/cats/)
* Support log level
* Support maximum 5 arbitrary data attributes
* Support JSON and String format
* Support logback and self4j

# Why

Our team use [Splunk](https://www.splunk.com/) to collect and analyze log, it support JSON very well, so we'd like to print our log in json format.

# How

A picture is worth a thousand words

![](https://images.shangjiaming.com/jlogger-classes.png)

## Overloading functions

When we print log, we usually want to add some useful information to help analysis, such as description, time, key data etc.
Usually we will convert the data to string and concat them, then print it in a single line, so the easiest logger can just accept the log level and string.

But I'm too lazy to concat the useful information by myself, I want the logger to do it, then I just need to pass all the information to it. That's why I use lots of overloading functions for `info`, `warn` and `error`.


## Formatter

Now I can pass arbitrary number of data to logger, but these data may have different type, how does logger know the formatting logic of these data?

The answer is I will tell it. I use `Formatter` to define the formatting logic of all possible type and pass the `Formatter` of each data type to the overloading functions.

```scala
  final def info[A: Formatter[*, B]](
      description: String,
      data: (String, A)
  )(implicit stringFormatter: Formatter[String, B]): M[Unit]

  final def info[A1: Formatter[*, B], A2: Formatter[*, B]](
      description: String,
      data1: (String, A1),
      data2: (String, A2)
  )(implicit stringFormatter: Formatter[String, B]): M[Unit]
```

## Logger

After we format all the data to given type, we need to convert the type to string and print it, that's what logger does. 

First it will combine all the formatted data to one data

```scala
  def generateContent(logLevel: LogLevel, now: Instant, attrs: Seq[B]): B
```

Then print the log 

```scala
  final def log(logLevel: LogLevel, attrs: B*): M[Unit]
```




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
trait Printer[M[_], A] {
  def print(logLevel: LogLevel, content: A): M[Unit]
}
```

Support different log format
```scala
trait Formatter[A, B] {
  def format(key: String, value: A): B
}
```