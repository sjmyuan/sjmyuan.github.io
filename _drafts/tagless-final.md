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

  If we put `F[_]` on function level, 
  we can't involve any type classes of `F[_]` in the child class 
  which mean we can't utilize Monad, Functor or Sync  to implement the logic

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

  We need to make all the expression pure.
  In FP, to make a log operation pure, we can use `Sync` of `F[_]`

  ```scala
  class ConsoleLogger[F[_]:Sync] extends Logger[F] {
    def info(message: String): F[Unit] = {
      Sync[F].delay(System.out.println(message));
    }
  }
  ```

  To chain the pure expression, we can use `Monad` of `F[_]`. 
  Because `Sync` is also a `Monad` and we need to use `Sync` in some expression,
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
def main()={
  val logger = new ConsoleLogger[IO]
  val userApi = new InMemoryUserApi[IO](logger, Map("1" -> "Jame", "2" -> "Tom"))

  userApi.get("1").unsafeRunSync // "Jame"
  userApi.get("3").unsafeRunSync // Error
}
```

You can see we determine the `F[_]` to be `IO` in main, 
our implementation just need to care about the minimum requirement of `F[_]` to implement the logic.

Why don't we use `IO` directly? Why should we involve the minimum requirement of `F[_]`?

If we use `IO` directly, what if we want to use other effect in the future? for example `Task` or `ZIO`.
what if some team member just wrap all the code in one `IO`? it is valid for compiler but we know it's a very bad code.


If we ignore the type parameter `F[_]`, this is just a normal Java code, no magic.

* Use Interface(trait) to decouple
* Easy to mock dependencies
* Easy to understand

The type parameter `F[_]` allow us 
* Decide the effect when we compose and run the application.
* Localize the restriction of `F[_]` in implementation
