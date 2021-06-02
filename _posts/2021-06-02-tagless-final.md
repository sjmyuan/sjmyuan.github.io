---
title: Tagless Final
tags:
- Scala
categories:
- Scala Tutorial
date: 2021-06-02 13:20 +0800
---
Tagless Final is a coding pattern in Scala. 

You may ask why it was called Tagless Final, it's a long story but won't block us to use it,
so let's answer the question in the future(or you can read [Introduction to Tagless Final](https://serokell.io/blog/tagless-final) first).

In this blog, we will focus on how to use it.

# Requirement

Let's start from a simple requirement. 

Say we have an api which can get a user by id, and to make it easy to debug, we want to log the id every time.

# Java implementation

If we are Java developer, we may implement it like this

```java
class User {
  String name;
  Int age;
  String id;

  User(String name, Int age, String id) {
    this.name = name;
    this.age = age;
    this.id = id;
  }
}

interface Logger {
  public void info(message: String);
}

class ConsoleLogger extends Logger {
  public void info(message: String) {
    System.out.println(message);
  }
}

interface UserApi {
  public User getUser(id: String);
}

class InMemoryUserApi implement UserApi {
  Logger logger;
  Map<String, User> cache;

  InMemoryUserApi(logger: Logger, cache: Map<String, User>) {
    this.logger = logger;
    this.cache = cache
  }

  public User getUser(id: String) {
    logger.info("Getting user by " + id);
    return cache.get(id);
  }
}
```

# Scala implementation

It can be translated to Scala directly

```scala
case class User(name: String, age: Int, id: String)

trait Logger {
  def info(message: String): Unit
}

class ConsoleLogger extends Logger {
  def info(message: String): Unit = {
    println(message)
  }
}

trait UserApi {
  def getUser(id: String): User
}

class InMemoryUserApi(logger: Logger, cache: Map[String, User]) extends UserApi {
  def getUser(id: String): User = {
    logger.info(s"Getting user by ${id}")
    cache.get(id)
  }
}
```

# Pure implementation

You may notice the function `getUser` and `info` have side effect(console output and exception). 
To make them pure, we need to involve some higher-kind type to express the side effect, such as `Option`, `Either` or `IO` etc.

We use `F[_]` to stand for them here and call it as effect in the following part. 

Then the code become

```scala
case class User(name: String, age: Int, id: String)


trait Logger[F[_]] {
  def info(message: String): F[Unit]
}

class ConsoleLogger[F[_]] extends Logger[F] {
  def info(message: String): F[Unit] = {
    // println(message)
  }
}

trait UserApi[F[_]] {
  def getUser(id: String): F[User]
}

class InMemoryUserApi[F[_]](logger: Logger[F], cache: Map[String, User]) extends UserApi[F] {
  def getUser(id: String): F[User] = {
    // logger.info("Getting user by ${id}")
    // cache.get(id)
  }
}
```

There are two questions here

1. Why not put `F[_]` on function?

    If we put `F[_]` on function, 
    we can't involve any type class of `F[_]` in the child class 
    which mean we can't utilize Monad, Functor or Sync to implement the logic

    ```scala
    def info[F[_]](message: String): F[Unit]
    ```
    is different from

    ```scala
    def info[F[_]: Monad](message: String): F[Unit]

    // ===

    def info[F[_]](message: String)(implicit M: Monand[F]): F[Unit]
    ```

    And it's hard to maintain the code if the effects are different in one class.

2. How to re-implement the logic by `F[_]`

    We need to make all the function pure, 
    so we need the type class of `F[_]` which can be injected by context bound.

    We can use  `Sync` of `F[_]` to print log without side effect.

    ```scala
    class ConsoleLogger[F[_]:Sync] extends Logger[F] {
      def info(message: String): F[Unit] = {
        Sync[F].delay(println(message));
      }
    }
    ```

    We can use  `Monad` of `F[_]` to chain expression. 
    `Sync` is also a `Monad`, we can still use `Sync` here.

    ```scala
    class InMemoryUserApi[F[_]:Sync](logger: Logger[F], cache: Map[String, User]) extends UserApi[F] {
      def getUser(id: String): F[User] = for {
        _ <- Sync[F].delay(logger.info("Getting user by ${id}"));
        user <- Sync[F].delay(cache.get(id))
      } yield user
    }
    ```

Now we have an implementation using Tagless Final pattern, but the `F[_]` is still undetermined, how should we run it in main?

```scala
def main()= {
  val logger = new ConsoleLogger[IO]
  val userApi = new InMemoryUserApi[IO](logger, Map("1" -> User("James", 21, "1"), "2" -> User("Tom", "38", "2")))

  userApi.get("1").unsafeRunSync // "Jame"
  userApi.get("3").unsafeRunSync // Error
}
```

We choose `IO` to be `F[_]` in main,
because `IO` already implemented the type class `Sync`, which is the minimum requirement of implementation.

There may be more questions here

1. Why not use `IO` directly in implementation? 

    If we use `IO` directly, we can not use other effect in the future, for example `Task` or `ZIO`.

    But to be honest, it's unlikely to happen in real system.

2. Why should we only involve the minimum requirement of `F[_]`?

    If we use `Sync` everywhere for example, 
    we can not prevent some team member from wrapping all the code in one `Sync.delay`, which is valid for compiler but a very bad code.

    We'd better just inject the dependencies required by implementation.

If your project complexity is low and team member don't have enough experience, 
I think it's ok to use an effect directly in implementation which is happening in my project.

# Summary

Ok, let's give a simple definition of Tagless Final to help us understand it(not accurate but easier to understand)

> Tagless Final is just OO + Effect

We can use normal OO technique to compose our code,
but remember to add effect to Interface/Class,
and inject the required type class by context bound,
then utilize them to make the function pure.
