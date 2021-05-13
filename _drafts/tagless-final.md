---
title: Tagless Final
tags:
  - Scala
categories:
  - Scala Tutorial
---

Tagless Final is a coding pattern in Scala, which is similar with the normal Java Classs inheriting Interface if we ignore the type parameter. 

Let's look at a simple example

```scala

trait UserApi[F[_]] {
  def getUser(id: String): F[User]
}

class HttpUserApi[F[_]: MonadError](url:String) extends UserApi[F] {
  def getUser(id: String): F[User] = ???
}

trait UserService[F[_]] {
  def getUsers(ids: List[String]): F[List[User]]
}

class CustomerService[F[_]: MonadError](userApi: UserApi[F]) extends UserService[F[_]] {
  def getUsers(ids: List[String]): F[List[User]] = ids.traverse(x => userApi.getUser(x))
}

object Main extends App {
  val userApi = new HttpUserApi[IO](???)
  val userService = new CustomerService[IO](userApi)

  val users = userService.getUsers(List('1','2','3'))

  println(users.unsafeRunSync)
}

```

If we ignore the type parameter `F[_]`, this is just a normal Java code, no magic.

* Use Interface(trait) to decouple
* Easy to mock dependencies
* Easy to understand

The type parameter `F[_]` allow us 
* Decide the effect when we compose and run the application.
* Localize the restriction of `F[_]` in implementation
