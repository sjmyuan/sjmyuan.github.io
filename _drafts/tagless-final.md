---
title: Tagless Final
tags:
  - Scala
categories:
  - Scala Tutorial
---

Tagless Final is a coding pattern in Scala. 

I know you want to ask what is Tagless Final, It's a long story and won't block us to use it,
So let's talk about it in the future(or you can read [Introduction to Tagless Final](https://serokell.io/blog/tagless-final) first).

In this blog, we will focus on how to use it.

# Requirement

Let's start from a simple requirement. 

Say we have an api which can get a user by id, and to make it easy to track the operation, we want to log the id every time.

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

interface UserApi {
  public User getUser(id: String);
}

interface Logger {
  public void info(message: String);
}

class ConsoleLogger extends Logger {
  public void info(message: String) {
    System.out.println(message);
  }
}

class InMemoryUserApi implement UserApi {
  Logger logger;
  Map<String, User> cache;

  InMemoryUserApi(logger: Logger, cache: Map<String, User>) {
   this.logger = logger;
   this.cache = cache
  }

  public User getUser(id: String) {
    logger.info("Getting user by ${id}");
    return cache.get(id);
  }
}
```

# Scala implementation

It can be translated to Scala directly

```scala
case class User(name: String, age: Int, id: String)

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

# Pure implementation

You may noticed the function `getUser` and `info` is not pure. 
To make them pure, we need to involve some higher-kind type to express the side effect, such as `Option`, `Either` or `IO` etc.

We can use `F[_]` to stand for them and call it as effect in the following part. then the code become

```scala
case class User(name: String, age: Int, id: String)

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

    And it's hard to maintain the code if the effects are different between functions in one class.

2. How to re-implement the logic by `F[_]`

    We need to make all the function pure.
    In FP, to make a log operation pure, we can use type class `Sync` of `F[_]`

    ```scala
    class ConsoleLogger[F[_]:Sync] extends Logger[F] {
      def info(message: String): F[Unit] = {
        Sync[F].delay(System.out.println(message));
      }
    }
    ```

    To chain the pure expression, we can use type class `Monad` of `F[_]`. 
    Because `Sync` is a `Monad` and we also need to use `Sync` in other place,
    We can just involve `Sync`.

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
because `IO` already implemented the type class `Sync`, which is the minimum requirement of `F[_]` to implement the logic.

Why not use `IO` directly? 

Why should we only involve the minimum requirement of `F[_]`?

Could we use `Sync` everywhere?

There may be lots of questions in our mind.

If we use `IO` directly, what if we want to use other effect in the future? for example `Task` or `ZIO`.
what if some team member just wrap all the code in one `IO`? it is valid for compiler but we know it's a very bad code.

So there are two benefits by choosing the effect in main 

1. Change the effect easily in the future
2. Only inject required dependencies(type class) to class

Even it's unlikely to change the effect in the future, it's still worth to use it to manage dependencies.

# Summary

Ok, let's give a simple definition of Tagless Final to help us to understand it

> Tagless Final is just OO + Effect

We can use normal OO technique to compose our code,
but remember to add an effect to every Interface/Class 
and utilize the existing type class of effect to make the function pure.
