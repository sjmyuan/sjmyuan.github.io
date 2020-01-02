---
title: Underscore
tags:
  - Scala
categories:
  - Scala Tutorial
---

In this small blog, we will try to summarize the usage of underscore in Scala.

# Function

## Convert method to function

Sometimes we want to assign a method of class/object to a function variable, then we can use underscore to do it.

```scala
object Sum {
  def sum(x: Int, y: Int) = x + y
}
```

```scala
val f:(Int, Int) => Int = Sum.sum // error
val f:(Int, Int) => Int = (Sum.sum _) // (Int, Int) => Int
```
## Inline function(Lambda)

Lambda is an important feature in Scala, it can make our code short and clean, let's see how we can use it.

```scala
(Some(1), Some(2)).mapN(_+_) //_ + _ equals to  (a,b) => a + b
List(1, 2, 3).map(_+1) // _ + 1 equals to (a) => a + 1
```

```scala
def PlusOne(x:Int) = x + 1
List(1, 2, 3).map(PlusOne(_)) // PlusOne(_) equals to (a) => PlusOne(a)
List(1, 2, 3).map(PlusOne) // For the function with same signature, you can just type the name
```

```scala
def Add(x:Int, y:Int) = x + y
List(1,2,3).map(Add(_, 1)) // Add(_, 1) equals to (a) => Add(a, 1)
List(x:Int => x+1, x:Int => x+2).map(_(1)) // _(1) equals to (f) => f(1)
```

Ok, it's just a simple replacement game.
The number of underscore means the number of function parameter, then compiler just use the parameter to replace the underscore with same index.

![](https://tva1.sinaimg.cn/large/006tNbRwly1gaifd3fzbbj304r033dfp.jpg)

## Ignore input

Sometimes we don't use some input parameter, but the compiler will give a warning which say some variable has not been used.
To eliminate the warning and reduce the likelihood of error, we can use underscore to ignore the input parameter.

```scala
List(1,2,3).map(_ => 0)
```

```scala
for {
  _ <- List(1, 2, 3)
} yield 0
```

## Ignore output

This is a very edge case. When we don't care about the result of last expression, we can use underscore to ignore it.

```scala
def dummyAdd(x: Int, y: Int): Unit = {
  val _ = x + y
}
```

# Pattern matching

Sometimes we don't care about the value of matching variable and also don't want compiler to raise warning, then we can use underscore to match any value.

```scala
case class User(name:String)
val user1 = User("Bob")

user1 match {
  case User(_) => println("I don't care about what's the name")
  case _ => println("But it must be a user")
}
```

# Type parameter

We know higher kind type(such as Option, Either, List...) can't be instanced directly.
It looks like a function of type which accept another type as parameter and then return a new type which can be instanced.

Sometimes we need to use the higher kind type as type parameter,
to seperate them from normal type, we need to tell compiler how many types they need to create a new type which can be instanced.
Then we can use underscore to do it, the number of underscore means the number of type they need.

```scala
def process[M[_], A](v:M[A]):Unit = 
  println(
    """
     I only accept a value whose type is composed by M and A, 
     and M must have a hole which mean it can accept another type parameter"
    """
  )
```
