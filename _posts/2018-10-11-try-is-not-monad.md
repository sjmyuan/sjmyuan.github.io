---
title: Try Is Not A Monad
tags:
  - Scala
---

In this blog we will talk about `scala.util.Try` and try to figure out if it is a Monad.

# What is Try?

`Try` is a data type which can represent`try/catch` in Scala. As long as you want to catch the exception, you can use it. for example

```scala
val a:Try[Int] = Try(1/0)
```

Let's see its rough defination

```scala
sealed trait Try[+A]
case class Success[+A](value:A) extends Try[A]
case class Failure[+A](exception:Throwable) extends Try[A]

object Try{
    def apply[A](r: =>A):Try[A] = 
      try Sucess(r)
      catch {
        case NonFatal(e) => Failure(e)  
      }
}
```

We can see the `apply` of `Try` is just a syntax sugar of `try/catch`.

`Try` also support `map` and `flatMap`

```sh
$ for {
  	a <- Try(1)
  	b <- Try(2)
  } yield a+b
res1: Try[Int] = Success(3)
```

```sh
$ for {
  	a <- Try(1/0)
  	b <- Try(2)
  } yield a+b
res2: Try[Int] = Failure(java.lang.ArithmeticException: / by zero)
```

# Why Try is not a Monad?

Just like `Future`,  `Try`is not `referential transparency`. Let's see the following example

```sh
$ for {
    _ <- Try(println("hello"))
    _ <- Try(println("hello"))
  } yield ()
hello
hello
res3: Try[Unit] = Success(())
```

```sh
$ val a:Try[Unit] = Try(println("hello"))
hello
a: Try[Unit] = Success(())
$ for {
    _ <- a
    _ <- a
  } yield ()
res5: Try[Unit] = Success(())
```

These two implementations have different side-effects, one print two `hello`, the other one just print one `hello`





