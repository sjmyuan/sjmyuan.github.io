---
title: Monad
tags:
- Scala
categories:
- Scala Tutorial
date: 2020-01-30 20:03 +0800
---
In this blog we will talk about Monad which is the core concept of FP.
We won't touch any category theory which belong to mathematics.
We will try to find out the motivation of Monad from code level.

# Question

Let's start from a very simple question, how do you implement the following formula in Scala?

![](https://tva1.sinaimg.cn/large/006tNbRwly1gber8n8qoej306x00zq2s.jpg)

Obviously we need four functions here

```scala
def sqrt(v:Double):Double = 
  if (v <0) throw new Exception("sqrt should not less than 0") else Math.sqrt(v)

def div(numerator:Double, denominator:Double):Double =
  if (denominator == 0) throw new Exception("denominator should not be zero") else numerator/denominator

def sum(left:Double, right:Double):Double = left + right

def sub(left:Double, right:Double):Double = left - right
```

Then we can implement the main function like this

```scala
def calculation:Unit = {
    val x1 = readDouble
    val x2 = readDouble
    val x3 = readDouble
    val x4 = readDouble
    val x5 = readDouble

    val divResult1: Double = div(x1, x2)

    val divResult2: Double = div(divResult1, x3)

    val sumResult: Double = sum(divResult2, x4)

    val sqrtResult: Double = sqrt(sumResult)

    val subResult: Double = sub(sqrtResult, x5)
    
    println(subResult)
}
```

Let's do some test

```sh
@ calculation
1
1
2
3
4
-2.1291713066130296
```

```sh
@ calculation
1
1
0
2
3
java.lang.Exception: denominator should not be zero
  ammonite.$sess.cmd11$.div(cmd11.sc:2)
  ammonite.$sess.cmd14$.calculation(cmd14.sc:7)
  ammonite.$sess.cmd16$.<init>(cmd16.sc:1)
  ammonite.$sess.cmd16$.<clinit>(cmd16.sc)
```

```sh
@ calculation
1
1
2
-4
5
java.lang.Exception: sqrt should not less than 0
  ammonite.$sess.cmd10$.sqrt(cmd10.sc:2)
  ammonite.$sess.cmd14$.calculation(cmd14.sc:7)
  ammonite.$sess.cmd17$.<init>(cmd17.sc:1)
  ammonite.$sess.cmd17$.<clinit>(cmd17.sc)
```

The last two test cases throw execptions, they are caused by `div` and `sqrt` which are not pure functions.

Then we get anther question, how do we make these functions pure?

# Solution

In [Algebraic Data Type](https://blog.shangjiaming.com/scala%20tutorial/algebraic-data-type/), we already talked about how to make a function pure.

Let's follow that steps to make `div` and `sqrt` pure.

1. Find out what effect the function send to outside

    These two functions send two effects to outside
    * The result of calculation
    * Exception when the input parameter is invalid

2. For each effect, create a corresponding `Product Type` for it

    ```scala
    case class Normal(v: Double)
    case class InvalidInput(error: String)
    ```
3. To unify the type of different effects, create `Sum Type` to give these effects a common parent type

    ```scala
    sealed trait Data
    case class Normal(v: Double) extends Data
    case class InvalidInput(error: String) extends Data
    ```
4. Refactor the function to use the `Algebraic Data Type` as return type

    ```scala
    def sqrt(v:Double):Data = 
      if (v <0) InvalidInput("sqrt should not less than 0") else Normal(Math.sqrt(v))

    def div(numerator:Double, denominator:Double):Data =
      if (denominator == 0) InvalidInput("denominator should not be zero") else Normal(numerator/denominator)
    ```

Then we can refactor the main function using pure `div` and `sqrt` like this

```scala
def calculation:Unit = {
    val x1 = readDouble
    val x2 = readDouble
    val x3 = readDouble
    val x4 = readDouble
    val x5 = readDouble
    
    val divResult1: Data = div(x1,x2)

    val divResult2: Data = divResult1 match {
      case Normal(x) => div(x, x3)
      case InvalidInput(e) => InvalidInput(e)
    }

    val sumResult: Data = divResult2 match {
      case Normal(x) => Normal(sum(x, x4))
      case InvalidInput(e) => InvalidInput(e)
    }

    val sqrtResult: Data = sumResult match {
      case Normal(x) => sqrt(x)
      case InvalidInput(e) => InvalidInput(e)
    }

    val subResult: Data = sqrtResult match {
      case Normal(x) => Normal(sub(x, x5))
      case InvalidInput(e) => InvalidInput(e)
    }

    subResult match {
      case Normal(x) => println(x)
      case InvalidInput(e) => throw new Exception(e)
    }
}
```

Please recall what we said in [What is Functional Programming?](https://blog.shangjiaming.com/scala%20tutorial/what-is-fp/)

> Functional Programming means construct most parts of programs using only pure function and centralize the parts with Side-Effect(usually we will put them in main function)

So except the last expression and `readDouble` in `calculation`, our program is pure, we answered the question.

Now let's see if we can make our code more clean and easy to be maintained.

# Refactor

To be honest, comparing with the original code, the FP code is ugly for me. There are so many `pattern-matching`, seems I need the following code everywhere

```scala
case Normal(x) => ???
case InvalidInput(e) => InvalidInput(e)
```

Let's see if we can remove the duplication.

Except the expression with `Side-Effect`, there are 4 expressions use it and the only different part is the branch of `Normal`

```scala
//divResult2
case Normal(x) => div(x, x3)

//sumResult
case Normal(x) => Normal(sum(x, x4))

//sqrtResult
case Normal(x) => sqrt(x)

//subResult
case Normal(x) => Normal(sub(x, x5))
```

`sumResult` and `subResult` have similar structure, they all need a function `Double => Double`. We can pass `sum/sub` by parameter and extract a common function like this

```scala
def computeIfNormal(data:Data)(f:Double=>Double):Data = data match {
    case Normal(v) => Normal(f(v))
    case InvalidInput(e) => InvalidInput(e)
}
```

`divResult2` and `sqrtResult` have similar structure, they all need a function `Double => Data`. We can pass `div/sqrt` by parameter and extract a common function like this

```scala
def computeIfNormalForComplexFunction(data:Data)(f:Double=>Data):Data = data match {
    case Normal(v) => f(v)
    case InvalidInput(e) => InvalidInput(e)
}
```

Then the `calculation` can be refactored like this

```scala
def calculation:Unit = {
    val x1 = readDouble
    val x2 = readDouble
    val x3 = readDouble
    val x4 = readDouble
    
    val divResult1: Data = div(x1,x2)

    val divResult2: Data = computeIfNormalForComplexFunction(divResult1)(x:Double => div(x, x3))

    val sumResult: Data = computeIfNormal(divResult2)(x:Double => sum(x, x4))

    val sqrtResult: Data = computeIfNormalForComplexFunction(sumResult)(sqrt)

    val subResult: Data = computeIfNormal(sqrtResult)(x: Double => sub(x, x4))

    subResult match {
      case Normal(x) => println(x)
      case InvalidInput(e) => throw new Exception(e)
    }
}
```

Looks better now, but the name of these two common functions are too long, let's give them a shorter name

```scala
def map(data: Data)(f: Double => Double): Data = data match {
    case Normal(v) => Normal(f(v))
    case InvalidInput(e) => InvalidInput(e)
}
```

```scala
def flatMap(data: Data)(f: Double => Data): Data = data match {
    case Normal(v) => f(v)
    case InvalidInput(e) => InvalidInput(e)
}
```

Then the `calculation` can be refactored like this

```scala
def calculation:Unit = {
    val x1 = readDouble
    val x2 = readDouble
    val x3 = readDouble
    val x4 = readDouble
    
    val divResult1: Data = div(x1,x2)

    val divResult2: Data = flatMap(divResult1)(x:Double => div(x, x3))

    val sumResult: Data = map(divResult2)(x:Double => sum(x, x4))

    val sqrtResult: Data = flatMap(sumResult)(sqrt)

    val subResult: Data = map(sqrtResult)(x: Double => sub(x, x4))

    subResult match {
      case Normal(x) => println(x)
      case InvalidInput(e) => throw new Exception(e)
    }
}
```

Hmm, there is another minor problem for me.

When we want to invoke `map`/`flatMap`, the idea in our mind is to find a way to apply the given function to the target data,
we will first get a target data then choose `map`/`flatMap` to apply given function. 

But our current code use wrong order, it choose `map`/`flatMap` first, then pass target data and given function.
The order make me uncomfortable, could we change it?

Thanks `Implicit`, we can do it!!

```scala
object Data {
  def map(data: Data)(f: Double=>Double): Data = data match {
      case Normal(v) => Normal(f(v))
      case InvalidInput(e) => InvalidInput(e)
  }

  def flatMap(data: Data)(f: Double=>Data): Data = data match {
      case Normal(v) => f(v)
      case InvalidInput(e) => InvalidInput(e)
  }

  implicit DataOps(v: Data){
      def map(f: Double => Double): Data = Data.map(v)(f)
      def flatMap(f: Double => Data): Data = Data.flatMap(v)(f)
  }
}
```

Then the `calculation` become

```scala
def calculation:Unit = {
    val x1 = readDouble
    val x2 = readDouble
    val x3 = readDouble
    val x4 = readDouble
    
    val divResult1: Data = div(x1,x2)

    val divResult2: Data = divResult1.flatMap(x:Double => div(x, x3))

    val sumResult: Data = divResult2.map(x:Double => sum(x, x4))

    val sqrtResult: Data = sumResult.flatMap(sqrt)

    val subResult: Data = sqrtResult.map(x: Double => sub(x, x4))

    subResult match {
      case Normal(x) => println(x)
      case InvalidInput(e) => throw new Exception(e)
    }
}
```

Perfect! With minor changes, our program is pure now and looks similar to the original code and easy to understand.

# Summary

Let's review what happened in the last section.

1. We make a function pure by returning algebraic data type, such as `div`, `sqrt`
2. We can not compose the refactored function with other functions directly(including refactored one and the unchanged one).
   For example, we can't compose `div` with `sqrt`, `sum` and `sub` directly, because it returns `Data` but other functions accept `Double`
3. To do the function compose, we do lots of pattern match on the algebraic data type and we got lots of duplicated code.
4. These duplicated code have the similar pattern, we extract some common functions to do it, such as `computeIfNormal` and `computeIfNormalForComplexFunction`.
5. To make the code clean, We give these common functions shorter names `map` and `flatMap`
6. To follow the thinking mode of human, we use `Implicit` to change the syntax of `map` and `flatMap`.

So we can give a summary
1. We use `map` and `flatMap` to remove duplication
2. We use `map` to compose the unchanged function which doesn't return the algebraic data type, such as `sum`, `sub`.
2. We use `flatMap` to compose the refactored function which return the algebraic data type, such as `div`, `sqrt`.

Actually we can make the `map` and `flatMap` more generic to support the generic algebraic data type, we call the generic version as `Monad`.

```scala
trait Monad[F[_]] {
  def map[A, B](fa: F[A])(f: A => B): F[B]
  def flatMap[A, B](fa: F[A])(f: A => F[B]): F[B]
}
```

It's a type class, you can review [Type Classes](https://blog.shangjiaming.com/scala%20tutorial/type-classes/) to see why we need a type class for `Monad`.
