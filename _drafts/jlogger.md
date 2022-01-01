---
title: jlogger - print log in JSON format
tags:
  - Scala
categories:
  - Project
---

# TLDR

I developed a scala log library [jlogger](https://github.com/sjmyuan/jlogger), you can add the following line to your `build.sbt` to install it

```scala
  libraryDependencies += "io.github.sjmyuan" %% "jlogger" % "0.0.2",
```

## Features
* Type safe and based on [cats](http://typelevel.org/cats/)
* Support log level
* Support maximum 5 arbitrary data attributes
* Support JSON and String format
* Support logback and self4j

# Why

Our team use [Splunk](https://www.splunk.com/) to collect and analyze log, it support JSON very well, so we'd like to print our log in JSON format.

# How

A picture is worth a thousand words

![](https://images.shangjiaming.com/jlogger-classes-0.0.2.png)

## Overloading functions

Apart from description, log level and timestamp, we usually want to include some key data to help analysis.
We can concat them to a single string, then print them. This is the most common way we can see now.

```scala
val name = "Tom"
val age = 10

logger.info(s"Got request from ${name} who is ${age} years old.") // Got request from Tom who is 10 years old.
```

But I'm too lazy to do this by myself, I want the library to help me, then I can pass the key data


```scala
val name = "Tom"
val age = 10

logger.info("Got request", "name" -> name, "age" -> age) // Got request: name=Tome, age=10.(for example)
```

The easiest solution is to pass a map or list

```scala
logger.info("Got request", Map("name" -> name, "age" -> age))
```

There are two problems about this

* not idle, always need to type Map constructor
* can't support data with different types

For type problem, we will discuss in next section. 

For Map constructor, considering we usually won't pass too many key data(5 at most in our team), I use the overloading function to solve this

```scala
  final def info[A](description: String, data: (String, A)): M[Unit]
  final def info[A1, A2](description: String, data1: (String, A1), data2: (String, A2)): M[Unit]
  final def info[A1, A2, A3](description: String, data1: (String, A1), data2: (String, A2), data3: (String, A3)): M[Unit]
  final def info[A1, A2, A3, A4](description: String, data1: (String, A1), data2: (String, A2), data3: (String, A3), data4: (String, A4)): M[Unit]
  final def info[A1, A2, A3, A4, A5](description: String, data1: (String, A1), data2: (String, A2), data3: (String, A3), data4: (String, A4), data5: (String, A5)): M[Unit]
```

## Formatter

Now let's fix the type problem.

Let's review what's the step to print a log:
1. Give a description to explain what's going on
2. Convert the key data to some data type which can be converted to string, usually we will use string directly.
3. Concat them together and pass the string to logger

We can do something in step 2 to tell the logger how to convert the data to the given type. This is the purpose of `Formatter`

```scala
trait Formatter[A, B] {
  def format(key: String, value: A): B
}
```

Because we want to print log in JSON format, we can define a formatter based on circe like this:

```scala
  implicit def generateJsonFormatter[A: Encoder]: Formatter[A, Json] =
    new Formatter[A, Json] {
      def format(key: String, value: A): Json = Json.obj(key -> value.asJson)
    }
```

Then we can pass the formatter to the logger to tell them how to convert the data to the target type(B in this example)

```scala
  final def info[A: Formatter[*, B]](description: String, data: (String, A)): M[Unit]
  final def info[A1: Formatter[*, B], A2: Formatter[*, B]](description: String, data1: (String, A1), data2: (String, A2)): M[Unit]
  final def info[A1: Formatter[*, B], A2: Formatter[*, B], A3: Formatter[*, B]](description: String, data1: (String, A1), data2: (String, A2), data3: (String, A3)): M[Unit]
  final def info[A1: Formatter[*, B], A2: Formatter[*, B], A3: Formatter[*, B], A4: Formatter[*, B]](description: String, data1: (String, A1), data2: (String, A2), data3: (String, A3), data4: (String, A4)): M[Unit]
  final def info[A1: Formatter[*, B], A2: Formatter[*, B], A3: Formatter[*, B], A4: Formatter[*, B], A5: Formatter[*, B]](description: String, data1: (String, A1), data2: (String, A2), data3: (String, A3), data4: (String, A4), data5: (String, A5)): M[Unit]
```

## JLogger

Now we can get a list of target type(B in the last section), which is converted from key data, how do we convert them to a single log message?

In `JLogger`, I require the target type to implement the `Monoid` type class, then we can combine them to a single target type.

```scala
abstract class JLogger[M[_]: Monad, B: Monoid](implicit
    clock: Clock[M],
    stringFormatter: Formatter[String, B],
    instantFormatter: Formatter[Instant, B]
) { ... }
```

Then we define a abstract function to allow child classes to convert the target type to a single log message and print it.

```scala
  def log(logLevel: LogLevel, attrs: B): M[Unit]
```


# Usage

## Print log with self4j

```scala
import io.github.sjmyuan.jlogger.SimpleJsonLogger
import cats.effect.IO
import cats.effect.IOApp
import org.slf4j.LoggerFactory

object App extends IOApp {
    val logger = new Self4jJsonLogger[IO](LoggerFactory.getLogger(getClass()))

    val program = for {
      _ <-logger.warn("This is a json logger")
      _ <-logger.error("This is a json logger")
      _ <-logger.info("This is a json logger")
    } yield()

    program.unsafeRunSync()
}
```

## Print log with println

```scala
import io.github.sjmyuan.jlogger.SimpleJsonLogger
import cats.effect.IO
import cats.effect.IOApp

object App extends IOApp {
    val logger = new SimpleJsonLogger[IO]()

    val program = for {
      _ <-logger.warn("This is a json logger")
      _ <-logger.error("This is a json logger")
      _ <-logger.info("This is a json logger")
    } yield()

    program.unsafeRunSync()
}
```