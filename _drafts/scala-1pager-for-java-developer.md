---
title: Scala 1Pager for Java Developer
tags:
  - Scala
---

The purpose of this blog is to help Java Developer contribute qualified Scala code in 2 weeks.

I won't dig into the details of Scala/FP, instead, I will try to list all the scenarios we may encounter when implementing features and give the copy-paste solution.

# Variable

## How to create a variable independently?

```scala
val intVar: Int = 1

val doubleVar: Double = 1.0

val stringVar: String = "hello world"

val listVar: List[String] = List("hello", "world")

val mapVar: Map[String, Int] = Map("hello" -> 1, "world" -> 2)

val tupleVar: (String, Int) = ("hello", 1)

// trait DemoTrait {....}
val traitVar: DemoTrait = new DemoTrait{}

// class DemoClass(name:String) {....}
val classVar: DemoClass = new DemoClass(name = "Tom")

// case class DemoCaseClass(name:String, age: Int)
val caseClassVar: DemoCaseClass = DemoCaseClass(name="Tome", age=1)

val functionVar: Int => Int = (x:Int) => x +1
```

## How to create a variable in for-expression?

```scala
for {
 ...
 classVar = new DemoClass(name = "Tom")
 ...
} yield ...
```

# Method and function

## How to create a method?

```scala
class Calculator {
  def add(x: Int, y:Int): Int = x + y
}
```

## How to create a method with default parameter?

```scala
class Calculator {
  def add(x: Int, y:Int = 1): Int = x + y
}
```

## How to create a method with function parameter?

```scala
class UserRepository {
def getUsers(userIds: List[String], fetch: (id:String) => User): List[User] = userIds.map(id => fetch(id))
}
```

## How to create a lambda?

```scala
val intList: List[Int] = List(1, 2, 3, 4)

intList.map(x => x+1) // List(2, 3, 4, 5), x => x+1 is a lambda
```

# Class

## How to write a main function?

```scala

object Main extends IOApp {
 override def run() = {
   ....
 }
}
```

## How to create a service/repository?

```scala
trait UserRepository[M[_]] {
 def getUser(id: String): M[User]
}

class HttpUserRepository[M[_]: Sync] {
  def getUser(id: String): M[User] = ???
}
```

## How to inject dependency?

```scala
trait UserRepository[M[_]] {
 def getUser(id: String): M[User]
}

class HttpUserRepository[M[_]: Sync](httClient: Client[M]) { // httpClient is dependency
  def getUser(id: String): M[User] = httClient.get[User](s"https://example.com/${id}")
}
```

## How to raise an error?

```scala
trait UserRepository[M[_]] {
 def getUser(id: String): M[User]
}

class HttpUserRepository[M[_]: Sync](httClient: Client[M]) {
  def getUser(id: String): M[User] = if(id.empty) {
      Sync[M].raiseError("The id should not be empty")
    }else {
      httClient.get[User](s"https://example.com/${id}")
    }
}
```

## How to return pure value?

```scala
trait UserRepository[M[_]] {
 def getUser(id: String): M[User]
}

class HttpUserRepository[M[_]: Sync](httClient: Client[M]) {
  def getUser(id: String): M[User] = if(id.empty) {
      Sync[M].pure(User(name="default"))
    }else {
      httClient.get[User](s"https://example.com/${id}")
    }
}
```

## How to invoke unsafe function?

```scala
trait UserRepository[M[_]] {
 def getUser(id: String): M[User]
}

class HttpUserRepository[M[_]: Sync](httClient: Client[M]) {
  def getUser(id: String): M[User] = if(id.empty) {
      for {
        _ <- Sync[M].delay(println("return default user"))
      } yield User(name="default")
    }else {
      httClient.get[User](s"https://example.com/${id}")
    }
}
```

## How to instance dependency?

```scala
object Main extends IOApp {
 def run():ExitCode = {
   ...

   val httpClient = ???
   val userRepository = new HttpUserRepository(httpClient)

   ...
 }
}
```

## How to adapt error type?

```scala
trait UserRepository[M[_]] {
 def getUser(id: String): M[User]
}

class HttpUserRepository[M[_]: Sync](httClient: Client[M]) {
  def getUser(id: String): M[User] = if(id.empty) {
      for {
        _ <- Sync[M].delay(println("return default user"))
      } yield User(name="default")
    }else {
      httClient.get[User](s"https://example.com/${id}").adapt( ??? ) // TODO
    }
}
```

## How to return Option?

```scala
import cats.implicits._
import cats.effect.Sync
import cats.effect.IO

def getFirstElement[M[_]: Sync](list: List[String]): M[String] = {
  Sync[M].fromOption(list.headOption, new Error("The list is empty"))
}

getFirstElement[IO](List()).attempt.unsafeRunSync() // :Either[Throwable, String] = Left(value = java.lang.Error: The list is empty)
getFirstElement[IO](List("1", "2")).unsafeRunSync() // :String = "1"
```

## How to return Either?

```scala
import cats.implicits._
import cats.effect.Sync
import cats.effect.IO

def add[M[_]: Sync](x: String, y: String): M[Int] = for {
  xInt <- Sync[M].fromEither(Either.catchNonFatal(x.toInt))
  yInt <- Sync[M].fromEither(Either.catchNonFatal(y.toInt))
} yield xInt + yInt

add[IO]("1", "2").unsafeRunSync() // :Int = 3
add[IO]("a", "2").attempt.unsafeRunSync() // :Either[Throwable, Int] = Left(value = java.lang.NumberFormatException: For input string: "a")
```

# Flow Control

## How to check Option value?

```scala
def checkOption(x: Option[String]): String = {
  x match {
    case None => "empty value"
    case Some(v) => v
  }
}

checkOption(Some("hello world")) // :String = "hello world"
checkOption(None) // :String = "empty value"
```

## How to check Either value?

```scala
def checkEither(x: Either[String, Int]): String = {
  x match {
    case Left(e)  => s"left is ${e}"
    case Right(v) => s"right is ${v}"
  }
}

checkEither(Left("Something wrong")) // :String = "left is Something wrong"
checkEither(Right(1)) // :String = "right is 1"
```

## How to check List value?

```scala
def checkList(list: List[String]): String = list match {
  case head :: Nil => s"There is only one element ${head}"
  case first :: second :: tail =>
    s"There are more than 2 elements and the first one is ${first} and the second one is ${second}"
  case List(x, y, z) =>
    s"There are exact 3 element, first is ${x}, second is ${y}, third is ${z}"
  case _ => "No match"
}

checkList(List("hello"))
// res0: String = "There is only one element hello"
checkList(List("hello", "world"))
// res1: String = "There are more than 2 elements and the first one is hello and the second one is world"
checkList(List("hello", "world", "Ha ha"))
// res2: String = "There are more than 2 elements and the first one is hello and the second one is world"
checkList(List("hello", "world", "Ha ha", "no match"))
// res3: String = "There are more than 2 elements and the first one is hello and the second one is world"
```

## How to compose multiple expression?

```scala
def composeExpression(
    f1: () => Either[String, Int],
    f2: () => Either[String, Int]
): Either[String, Int] = for {
  x1 <- f1()
  x2 <- f2()
} yield x1 + x2

composeExpression(() => Left("Something wrong"), () => Right(1))
// res6: Either[String, Int] = Left(value = "Something wrong")
composeExpression(() => Right(1), () => Right(1))
// res7: Either[String, Int] = Right(value = 2)
```

# Library

## JSON

### How to encode class to JSON?

### How to decode JSON to class?

## HTTP

### How to write a route?

### How to send a request?

### How to return a response?

## Database

### How to execute SQL?

# Test

## How to create a test?

## How to test expected value?

## How to mock service/repository?
