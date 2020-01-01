---
title: Algebraic Data Type
tags:
  - Scala
categories:
  - Scala Tutorial
---

In this blog, we will talk about `Algebraic Data Type` in Scala and try to find out why we need it.

# Question

Say we have a function like this

![](https://tva1.sinaimg.cn/large/006tNbRwly1gag8ykendij30h90aq74m.jpg)

We know it's not a pure function, because it throw exception when y equals 0.

To use Functional Programming in our program, we need to make it to be pure.

How should we do?

# Solution

According to the definition of pure function, to make a function to be pure, we need to eliminate the `Side-Effect`.

But we don't want to lose the effect given by the exception, So we need to make the effect of exception returned by the `return-expression`.

Then the function should look like this

```scala
def div(x: Double, y: Double): Double = {
  if(y == 0) 
    new Exception("/ by zero")
  else 
    x/y
}
```

Unfortunately, this code can't be compiled, because the type of `return-expression` is `Any` which can't be assigned to the expected type `Double`.

We definitely can change the return type to be `Any` 

```scala
def div(x: Double, y: Double): Any = {
  if(y == 0) 
    new Exception("/ by zero")
  else 
    x/y
}
```

it can be compiled, but the type checking is too weak, compiler can't help us if we make mistake like this

```scala
def div(x: Double, y: Double): Any = {
  if(y == 0) 
    new Exception("/ by zero")
  else 
    (x/y).toString
}
```

All the existing consumers of `div` use its result as `Double`, but we return `String` here and it can be compiled, then we can only find the bug in running time.

So we have two requirements here
* The function should be able to return two different effects which have different types.
* Keep the type checking strong.

To achieve them, we need to find out the nearest parent type of these effects and set it to be the return type of function.

Both `Exception` and `Double` are primitive types, their common parent type is `Any`, which make the type checking weak then can't be used.

What should we do now?

Now that we can't use the primitive type, how about use another custom type to wrap them? then we can give the custom types an common parent type. Let's try.

Firstly, wrap all the effects we want to return with custom type 

```scala
case class DivResult(v: Double)
case class ExceptionResult(e: Exception)
```

And give them a parent type

```scala
trait Result
case class DivResult(v: Double) extends Result
case class ExceptionResult(e: Exception) extends Result
```

Then the `div` function can be refactored like this

```scala
def div(x: Double, y: Double): Result = {
  if(y == 0) 
    ExceptionResult(new Exception("/ by zero"))
  else 
    DivResult(x/y)
}
```

It works! we can return `Exception` and `Double` together and use the custom type to make the type checking still strong.

And if we want to return more information for one effect, we can just modify the corresponding custom type.

For example, if we want to return the input parameter together with `Exception`, we can refine the custom types like this

```scala
trait Result
case class DivResult(v: Double) extends Result
case class ExceptionResult(e: Exception, parameter: (Double, Double)) extends Result
```

```scala
def div(x: Double, y: Double): Result = {
  if(y == 0) 
    ExceptionResult(new Exception("/ by zero"), (x, y))
  else 
    DivResult(x/y)
}
```

# Refactor

We know we can return more information for given effect by refining the custom type, but how do you know what's the future requirements?

Is there what we can do to make the custom type more generic?

Now that we don't know what information should be returned for given effect and we definitely know which effect should be returned for given scenario,
how about we just leave the information of effect to be generic?

Let's see if we can make it

```scala
trait Result[E, A]
case class DivResult[E, A](v: A) extends Result[E, A]
case class ExceptionResult[E, A](e: E) extends Result[E, A]
```

Then we can refactor the `div` function more flexiable

```scala
def div(x: Double, y: Double): Result[Exception, Double] = {
  if(y == 0) 
    ExceptionResult[Exception, Double](new Exception("/ by zero"))
  else 
    DivResult[Exception, Double](x/y)
}
```

```scala
def div(x: Double, y: Double): Result[(Exception, (Double, Double)), Double] = {
  if(y == 0) 
    ExceptionResult[(Exception, (Double, Double)), Double]((new Exception("/ by zero"), (x, y)))
  else 
    DivResult[(Exception, (Double, Double)), Double](x/y)
}
```

We know `sqrt` have the same type of effects with `div`, then we can also refine it

```scala
def sqrt(x: Double): Result[Exception, Double] = {
  if(x < 0) 
    ExceptionResult[Exception, Double](new Exception("sqrt by negative"))
  else 
    DivResult[Exception, Double](Math.sqrt(x))
}
```

Hmm, it can be compiled, but our function is `sqrt`, the return type is `DivResult`, it doesn't make sense. 

We need to make the name of custom type more generic. Now that we only return one effect at the same time and each effect only use one type parameter, let's refine the custom type like this

```scala
trait Either[E, A]
case class Right[E, A](v: A) extends Either[E, A]
case class Left[E, A](e: E) extends Either[E, A]
```

Then we can refine `div` and `sqrt` like this

```scala
def div(x: Double, y: Double): Either[Exception, Double] = {
  if(y == 0) 
    Left[Exception, Double](new Exception("/ by zero"))
  else 
    Right[Exception, Double](x/y)
}
```

```scala
def sqrt(x: Double): Either[Exception, Double] = {
  if(x < 0) 
    Left[Exception, Double](new Exception("sqrt by negative"))
  else 
    Right[Exception, Double](Math.sqrt(x))
}
```

Bingo! This is `Either` monad, we will talk about it more in the future.

# Summary

* To make all the effects of function retuned by `return-expression` and make the type checking strong, we create custom type to wrap primitive types 

  ```scala
  trait Result
  case class DivResult(v: Double) extends Result
  case class ExceptionResult(e: Exception) extends Result
  ```

* To make the custom type more generic, we use type parameter to refine it

  ```scala
  trait Result[E, A]
  case class DivResult[E, A](v: A) extends Result[E, A]
  case class ExceptionResult[E, A](e: E) extends Result[E, A]
  ```

* To let the name of custom type make more sense, we refine its name

  ```scala
  trait Either[E, A]
  case class Right[E, A](v: A) extends Either[E, A]
  case class Left[E, A](e: E) extends Either[E, A]
  ```

Ok, this is `Algebraic Data Type`

* For the signle effect `Right` or `Left`, we call it as `Product Type` which contains the detailed information of effect.
* For `Either` `Right` and `Left`, we call the inheritance relationship between them as `Sum Type` which can be used to unify the type of effect.
* We call `Product Type` and `Sum Type` together as `Algebraic Data Type`

Let's recall how to make a function to be pure

* Find out what effect the function send to outside
  * The result of division when y is not zero
  * Exception when y is zero

* For each effect, create a corresponding `Product Type` for it

  ```scala
  case class DivResult(v: Double)
  case class ExceptionResult(e: Exception)
  ```

* To unify the type of differnt effects, create `Sum Type` to give the effects a common parent type

  ```scala
  trait Result
  case class DivResult(v: Double) extends Result
  case class ExceptionResult(e: Exception) extends Result
  ```

* Refactor the function to use the `Algebraic Data Type` as return type

  ```scala
  def div(x: Double, y: Double): Result = {
    if(y == 0) 
      ExceptionResult(new Exception("/ by zero"))
    else 
      DivResult(x/y)
  }
  ```

Using this procedure, we can get lots of generic `Algebraic Data Type`, such as Option, Either, IO, Reader, Writer, State etc.
We will talk about which type of function can use them to be pure laterly.

