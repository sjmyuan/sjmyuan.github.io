---
title: Reader Monad
tags:
- Scala
categories:
- Scala Tutorial
---

Reader Monad is very popular in FP, you can find lots of high qulity blogs by Google.

[...And Monads for (Almost) All: The Reader Monad](https://dev.to/riccardo_cardin/and-monads-for-almost-all-the-reader-monad-1ife)
use a very good example to explain why we need Reader Monad.

[A Simple Reader Monad Example](https://blog.ssanj.net/posts/2014-09-23-A-Simple-Reader-Monad-Example.html)
give a simple definition of Reader Monad which filter out the noise of ReaderT.

In this blog, I will derive the Reader Monad from the real problem I got in project.

# The Problem

Let's continue to use the simplified example in [Cake Pattern](https://blog.shangjiaming.com/scala%20tutorial/cake-pattern/)

```scala
import scala.concurrent.Future
import scala.concurrent.ExecutionContext.Implicits._
object Main {
  def getData: List[Int] = {
    println(s"fetching data")
    List(1, 2, 3)
  }
  def encode(data: List[Int]): List[Int] = {
    println(s"data before encode ${data}")
    val result =  data.map(_ + 1)
    println(s"data after encode ${result}")
    result
  }

  def save(data: List[Int]): Unit = {
    println(s"saving data ${data}")
  }
  def job():Unit = {
    val data = getData
    val encodedData = encode(data)
    save(encodedData)
  }
  def main() = {
    Future(job())
    Future(job())

    //fetching data
    //fetching data
    //data before encode List(1, 2, 3)
    //data before encode List(1, 2, 3)
    //data after encode List(2, 3, 4)
    //data after encode List(2, 3, 4)
    //saving data List(2, 3, 4)
    //saving data List(2, 3, 4)
  }
}
```

Imagine we get a new requirement, all the logs in one thread need to contain the same transaction id,
then we can identify the whole workflow even in parallel program. The output should look like this

```scala
trans1 - fetching data
trans2 - fetching data
trans2 - data before encode List(1, 2, 3)
trans1 - data before encode List(1, 2, 3)
trans1 - data after encode List(2, 3, 4)
trans2 - data after encode List(2, 3, 4)
trans1 - saving data List(2, 3, 4)
trans2 - saving data List(2, 3, 4)
```

# Solution

## Pass transaction id by parameter

The intuitive solution is adding one more parameter to every function to pass the transaction id

```scala
import scala.concurrent.Future
import scala.concurrent.ExecutionContext.Implicits._
object Main {
  def getData(transaction: String): List[Int] = {
    println(s"$transaction - fetching data")
    List(1, 2, 3)
  }
  def encode(data: List[Int], transaction: String): List[Int] = {
    println(s"$transaction - data before encode ${data}")
    val result =  data.map(_ + 1)
    println(s"$transaction - data after encode ${result}")
    result
  }

  def save(data: List[Int], transaction: String): Unit = {
    println(s"$transaction - saving data ${data}")
  }

  def job(transaction: String):Unit = {
    val data = getData(transaction)
    val encodedData = encode(data, transaction)
    save(encodedData, transaction)
  }

  def main() = {
    Future(job("trans1"))
    Future(job("trans2"))

    //trans1 - fetching data
    //trans2 - fetching data
    //trans1 - data before encode List(1, 2, 3)
    //trans2 - data before encode List(1, 2, 3)
    //trans2 - data after encode List(2, 3, 4)
    //trans1 - data after encode List(2, 3, 4)
    //trans2 - saving data List(2, 3, 4)
    //trans1 - saving data List(2, 3, 4)
  }
}
```

This can work, but what if we have dozens of functions?
When it happened in my project, I always think why I have to pass one parameter everywhere but I don't need to care about it most of the time. 
It become more nosiy with more functions.

Maybe we can use a Class to wrap the functions, then all the functions can share one transaction variable in Class

```scala
import scala.concurrent.Future
import scala.concurrent.ExecutionContext.Implicits._

class Job(transaction: String) {
  def getData: List[Int] = {
    println(s"$transaction - fetching data")
    List(1, 2, 3)
  }
  def encode(data: List[Int]): List[Int] = {
    println(s"$transaction - data before encode ${data}")
    val result =  data.map(_ + 1)
    println(s"$transaction - data after encode ${result}")
    result
  }

  def save(data: List[Int]): Unit = {
    println(s"$transaction - saving data ${data}")
  }

  def run:Unit = {
    val data = getData
    val encodedData = encode(data)
    save(encodedData)
  }
}

object Main {

  def main() = {
    Future(new Job("trans1").run)
    Future(new Job("trans2").run)
  }
}
```

This looks cleaner, but we can't wrap all function in one Class, they will exist in different Class which follow the SOLID principle.
If we want to use this solution, we have to pass transaction to every Class constructor, which will have same problem with function parameter and just increase the threshold we can bear.

## Return function

Let's just think about why we feel noisy about the transaction parameter.

* The parameter has nothing to do with business logic, what we do is just get it from somewhere and pass it to somewhere else.
* Only few code use it, but we have to carry the context everywhere.
* The value is same most of the time, can't we put it to some global variable and get it if need?.

I think we agree the function need to know the transaction id, there are 3 ways to achieve this

* Function parameter
* External variable(Global/Class)
* Return a function in which the parameter is a placeholder of transaction id.

We already talked about the problem of Function/Class parameter, how about External global variable?

```scala
import scala.concurrent.Future
import scala.concurrent.ExecutionContext.Implicits._
object Main {
  var transaction = ""
  def getData: List[Int] = {
    println(s"$transaction - fetching data")
    List(1, 2, 3)
  }
  def encode(data: List[Int]): List[Int] = {
    println(s"$transaction - data before encode ${data}")
    val result =  data.map(_ + 1)
    println(s"$transaction - data after encode ${result}")
    result
  }

  def save(data: List[Int]): Unit = {
    println(s"$transaction - saving data ${data}")
  }

  def job():Unit = {
    val data = getData
    val encodedData = encode(data)
    save(encodedData)
  }

  def main() = {
    Future({
      transaction = "trans1"
      job()
    })
    Future({
      transaction = "trans2"
      job()
    })

    //trans1 - fetching data
    //trans2 - fetching data
    //trans2 - data before encode List(1, 2, 3)
    //trans2 - data before encode List(1, 2, 3)
    //trans2 - data after encode List(2, 3, 4)
    //trans2 - data after encode List(2, 3, 4)
    //trans2 - saving data List(2, 3, 4)
    //trans2 - saving data List(2, 3, 4)
  }
}
```

We can see it doesn't work, one global variable can't separate multiple threads.

Let's try the last option, return a function.

```scala
import scala.concurrent.Future
import scala.concurrent.ExecutionContext.Implicits._
object Main {
  def getData: String => List[Int] = transaction => {
    println(s"$transaction - fetching data")
    List(1, 2, 3)
  }
  def encode(data: List[Int]): String => List[Int] = transaction => {
    println(s"$transaction - data before encode ${data}")
    val result =  data.map(_ + 1)
    println(s"$transaction - data after encode ${result}")
    result
  }

  def save(data: List[Int]): String => Unit = transaction => {
    println(s"$transaction - saving data ${data}")
  }

  def job(transaction: String):Unit = {
    val getDataFunc = getData
    val encodedDataFunc: String => List[Int] = trans => encode(getDataFunc(trans))(trans)
    val saveDataFunc: String => Unit = trans => save(encodedDataFunc(trans))(trans)
    saveDataFunc(transaction)
  }

  def main() = {
    Future(job("trans1"))
    Future(job("trans2"))

    //trans1 - fetching data
    //trans2 - fetching data
    //trans2 - data before encode List(1, 2, 3)
    //trans1 - data before encode List(1, 2, 3)
    //trans1 - data after encode List(2, 3, 4)
    //trans2 - data after encode List(2, 3, 4)
    //trans1 - saving data List(2, 3, 4)
    //trans2 - saving data List(2, 3, 4)
  }
}
```

It works, compared to the original code, the difference are

* Return type changed from `A` to `String => A`, which is hard to understand. 
* The job function become a little bit complicated
* We need to follow some rule to compose functions to ensure they get the same transaction id.

If we can make these parts easy to maintain, it looks like a better solution.

## Return effect

Let's recall what's effect in [What is Funcional Programming?](https://blog.shangjiaming.com/scala%20tutorial/what-is-fp/)

> An effect is just a message from the inner of function to the outside of function

When we return a function in last section, what message we want to send to the outside? 
Maybe something like this

> Hey there, I can't do the job now, there are some required information missed,
> but I already delegated the job to an agency, 
> you just need to supply the required information, then it can do the job for you.

Hmm, what if we get two agencies which need to work together?

> Hey there, I already told all my agencies if any of them get the required information,
> they need to share it with each other.
> So don't worry about this, just let them do the job, they will pass the information if need.

Ok, from the last section, the agency is actually a function. 
According to [Algebraic Data Type](https://blog.shangjiaming.com/scala%20tutorial/algebraic-data-type/), we can define a simple effect for it.

```scala
case class Reader[A, B](run: A => B)
```

Because all the agencies need to read the required information first then do the job, we call the effect Reader here.

According to [Monad](https://blog.shangjiaming.com/scala%20tutorial/scala-monad/), we need to compose Reader to make them work together, let's define `map` and `flatMap` for it. 

```scala
case class Reader[A, B](run: A => B) {
  def map[C](f: B => C): Reader[A, C] = {
    val runN: A => C = (x: A) => f(run(x))
    Reader(runN)
  }

  def flatMap[C](f: B => Reader[A, C]): Reader[A, C] = {
    val runN: A => C = (x: A) => f(run(x)).run(x)
    Reader(runN)
  }
}
```

`map` is straightforward. 

`flatMap` apply some rule when composing 

```scala
f(run(x)).run(x)
```

It not only pass the required information(x) to current Reader(`run(x)`), but also next Reader(`f.run(x)`).
By this way, we pass `x` everywhere implicitly.

Then our code can be refined by Reader like this

```scala
import scala.concurrent.Future
import scala.concurrent.ExecutionContext.Implicits._

case class Reader[A, B](run: A => B) {
  def map[C](f: B => C): Reader[A, C] = {
    val runN: A => C = (x: A) => f(run(x))
    Reader(runN)
  }

  def flatMap[C](f: B => Reader[A, C]): Reader[A, C] = {
    val runN: A => C = (x: A) => f(run(x)).run(x)
    Reader(runN)
  }
}

object Reader {
  def ask[A]: Reader[A, A] = Reader[A, A](identity)
}

object Main {
  def getData: Reader[String, List[Int]] = {
    for {
      trans <- Reader.ask[String]
    } yield {
      println(s"$trans - fetching data")
      List(1, 2, 3)
    }
  }
  def encode(data: List[Int]): Reader[String, List[Int]] = {
    for {
      trans <- Reader.ask[String]
    } yield {
      println(s"$trans - data before encode ${data}")
      val result =  data.map(_ + 1)
      println(s"$trans - data after encode ${result}")
      result
    }
  }

  def save(data: List[Int]): Reader[String, Unit] = {
    for {
      trans <- Reader.ask[String]
    } yield {
      println(s"$trans - saving data ${data}")
    }
  }

  def job(transaction: String): Unit = {
    val program: Reader[String, Unit] = for {
    data <- getData
    encodedData <- encode(data)
     _ <- save(encodedData)
    } yield ()

    program.run(transaction)
  }

  def main() = {
    Future(job("trans1"))
    Future(job("trans2"))

    //trans2 - fetching data
    //trans1 - fetching data
    //trans1 - data before encode List(1, 2, 3)
    //trans2 - data before encode List(1, 2, 3)
    //trans1 - data after encode List(2, 3, 4)
    //trans2 - data after encode List(2, 3, 4)
    //trans2 - saving data List(2, 3, 4)
    //trans1 - saving data List(2, 3, 4)
  }
}
```

We add an util function `ask` here to make it easy to read the required information.

Now we have a Reader monad to help us solve the 3 problems raised in the last section

* The returned type is Reader, which is easier to understand than `String => A`.
* Use for-expression in job function, which has similiar structure with original code.
* Use Reader monad to compose function, which pass transaction everywhere implicitly.

The only price is we need to involve a Monad and understand its behavior. 
but it is worth to pay it.

# Summary

First, We can give a description of noisy information in this blog 

> Only few nested functions use it, but all the parent functions need to pass through it.

The signature of method in Java is just include method name and method parameters.
When we code, we also pay more attention to the parameters than the return value of function. 
If our function have to access some noisy information, putting them in return value is a better option.

Reader monad is a good choice to do this work.
