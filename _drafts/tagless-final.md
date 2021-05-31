---
title: Tagless Final
tags:
  - Scala
categories:
  - Scala Tutorial
---

Tagless Final is a coding pattern in Scala. 
I know you want to ask what is `Tagless Final`, It's a long story and won't block us to use it,
So let's talk about it in the future.
In this blog, we will focus on how to use it.

Let's start from a simple requirement. 

Say we have an api which can get a user by id, and to make it easy to track the operation, we want to log the id every time.

If we are Java developer, we may implement it like this

```java
interface UserApi {
  public User getUser(id: String): User
}

interface Logger {
  public info(message: String): void
}

class ConsoleLogger extends Logger {
  public info(message: String): void {
    System.out.println(message);
  }
}

class InMemoryUserApi(logger: Logger, cache: Map<String, User>) extends UserApi {
  publich User getUser(id: String): User {
    logger.info("Getting user by ${id}");
    cache.get(id)
  }
}
```

It can be translated to Scala directly

```scala
trait UserApi {
  def getUser(id: String): User
}

trait Logger {
  def info(message: String): Unit
}

class ConsoleLogger extends Logger {
  def info(message: String): Unit = {
    System.out.println(message);
  }
}

class InMemoryUserApi(logger: Logger, cache: Map[String, User]) extends UserApi {
  def getUser(id: String): User = {
    logger.info("Getting user by ${id}");
    cache.get(id)
  }
}
```

You may noticed the function `getUser` and `info` is not pure. 
To make them pure, we need to involve some higher-kind type to express the side effect, such as `Option`, `Either`, `IO` etc.

We can use `F[_]` to stand for them, then the code become

```scala
trait UserApi[F[_]] {
  def getUser(id: String): F[User]
}

trait Logger[F[_]] {
  def info(message: String): F[Unit]
}

class ConsoleLogger[F[_]] extends Logger[F] {
  def info(message: String): F[Unit] = {
    // System.out.println(message);
  }
}

class InMemoryUserApi[F[_]](logger: Logger[F], cache: Map[String, User]) extends UserApi[F] {
  def getUser(id: String): F[User] = {
    // logger.info("Getting user by ${id}");
    // cache.get(id)
  }
}
```

There are two problem here

1. Why put `F[_]` on class level not function level?

   If we put `F[_]` on function level, the functions between parent and child class can not know the `F[_]` of each other

2. How to re-implement the logic by `F[_]`

If we ignore the type parameter `F[_]`, this is just a normal Java code, no magic.

* Use Interface(trait) to decouple
* Easy to mock dependencies
* Easy to understand

The type parameter `F[_]` allow us 
* Decide the effect when we compose and run the application.
* Localize the restriction of `F[_]` in implementation
