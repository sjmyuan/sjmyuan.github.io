---
title: Review map and flatMap from code
tags:
  - Scala
---

In this blog, we will talk about `map` and `flatMap` again. they are the most common monadic operations in functional programming. I don't want to tell you what's Monad[^3] here, I just want to talk about these two functions in code level and see why we need them.

# The footstone of FP

Functional Programming[^1] is not Monad, it's just a programming pattern. 

When we use this pattern, we will meet some new problems, Monad is just a key to solve these problems, just like Factory method, Observer in Software design pattern[^2]

The base principle of this pattern is **every function should be pure function**, this is the source of truth, this is the fuze of Monad.

When we try to make every function pure[^1], we found there are lots of duplicated code to compose functions. When we try to reduce the duplication, we found we can extract some common functions. In these functions, there are two most common functions, we call them `map` and `flatMap`.

That's it, we don't need to understand the theory of mathematics and even don't need to understand what's is Monad. it's just a name like cat or mouse.

What we need to understand is why we need them, what's problem they solved. so I think it's a better way to review them from code. 

In the progress of code evolution, we can see **pure function** and **remove duplication** drive this evolution and I think they are the footstone of FP.

# The code without FP

Let's start with a very simple code, it will get 4 numbers from console and do the calculation
$$
\sqrt{x1 \div x2+x3}-x4
$$
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
    
    val result = sub(sqrt(sum(div(x1,x2),x3)),x4)
    println(result)
}
```

We have three test cases

```sh
@ calculation
1
2
3
4
-2.1291713066130296
```

```sh
@ calculation
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
2
-4
5
java.lang.Exception: sqrt should not less than 0
  ammonite.$sess.cmd10$.sqrt(cmd10.sc:2)
  ammonite.$sess.cmd14$.calculation(cmd14.sc:7)
  ammonite.$sess.cmd17$.<init>(cmd17.sc:1)
  ammonite.$sess.cmd17$.<clinit>(cmd17.sc)
```

We can see `div` and `sqrt` are not pure function, let's refactor it using FP.

# The code refactor using FP

## How to make a function pure?

Usually we say a function is pure when it does't have side-effect, but I'd like to say a function is pure when its result can represent all the information it want to return.

In our example, `div` and `sqrt` actually want to return two type of information: normal calculation result and invalid input. But the return type `Double` can only represent normal calculation result(let's ignore null here, we will talk about why we shouldn't use it in another blog), so the function have to use exception to represent the invalid input information.

So we can say the root cause of impure function is its return type can't represent all the information it want to return, it has to use other way to return these information, such as exception, log.

Now we know the problem, the solution is very simple, we just need to reuse or create a data type which can represent all the information. 

Usually we will use an existing data type(such as Option, Either), but let's assume we just build the program from ground, we don't have any existing complex data type.

To create a new data type to represent information, we usually use the following pattern

```scala
sealed trait Info
case class Info1(...) extends Info
case class Info2(...) extends Info
case class Info3(...) extends Info
```

## Let's do it

In our example, we want to create a new data type to represent `double` and `invalid input`, we can implement it like this

```scala
sealed trait Data
case class Normal(v:Double) extends Data
case class InvalidInput(error:String) extends Data
```

Then we can rewrite `div` and `sqrt` like this

```scala
def sqrt(v:Double):Data = 
  if (v <0) InvalidInput("sqrt should not less than 0") else Normal(Math.sqrt(v))

def div(numerator:Double, denominator:Double):Data =
  if (denominator == 0) InvalidInput("denominator should not be zero") else Normal(numerator/denominator)
```

# The first usage scenario

## What's the problem?

Now our `sqrt` and `div` return `Data` which can't be passed into `sum` and `sub` directly, our original expression won't compile

```scala
val result = sub(sqrt(sum(div(x1,x2),x3)),x4)
```

Because `Data` represent `Double` or `InvalidInput`, we need to check it before pass it into a function which only accept `Double`. we can rewrite the expression like this

```scala
val divResult:Data = div(x1,x2)
val result:Data = divResult match {
    case Normal(v) =>
       val sumResult:Double = sum(v, x3)
       val sqrtResult:Data = sqrt(sumResult)
       sqrtResult match {
           case Normal(v1) => sub(v1,x4)
           case InvalidInput(e) => InvalidInput(e)
       }
    case InvalidInput(e) -> InvalidInput(e)
}
```

This code is messy, too much nested level, I don't know why we have to write code like this, I can't see the benefit of using FP. These are the complains from my heart when I saw this code and These are challenges we have to face when we push FP in the team.

## How to fix it?

Ok, let's see if we can make this code better. we can see there is a pattern happend twice

```scala
divResult match {
    case Normal(v) => ...
    case InvalidInput(e) => InvalidInput(e)
}

sqrtResult match {
    case Normal(v1) => ...
    case InvalidInput(e) => InvalidInput(e)
}
```

They all check the content of `Data` type, if it is `InvalidInput`, just return the original value, if it is `Normal`, just pass the value to `sum` or `sub`.

We can extract it to an util function like this

```scala
def computeIfNormal(data:Data)(f:Double=>Double):Data = data match {
    case Normal(v) => Normal(f(v))
    case _ => data
}
```

Then we can rewrite the calculation like this

```scala
val divResult:Data = div(x1,x2)

val sumResult:Data = computeIfNormal(divResult)(result => sum(result, x3))

val sqrtResult:Data = sumResult match {
    case Normal(v) => sqrt(v)
    case e => e
}

val result:Data = computeIfNormal(sqrtResult)(result => sub(result, x4))
```

Hmm, looks better, but it's not good enough to persuade team to agree this refactor. the benefit can't afford the longer code. Let's hold on and talk about it in next chapter. Now let's talk about `computeIfNormal` more.

## That's map

`computeIfNormal(divResult)(result => sum(result, x3))` is a little bit hard to read, it's different from the way our mind works. Our mind like to read the code from left to right, but here we have to read all the expression before we know what it said.

To make it easier to read, we can add a syntax for `computeIfNormal` like this

```scala
object Data {
    implicit DataOps(v:Data){
        def computeIfNormal(f:Double => Double):Data = computeIfNormal(v)(f)
    }
}
```

Then the calculation expression become

```scala
val divResult:Data = div(x1,x2)

val sumResult:Data = divResult.computeIfNormal(result => sum(result, x3))

val sqrtResult:Data = sumResult match {
    case Normal(v) => sqrt(v)
    case e => e
}

val result:Data = sqrtResult.computeIfNormal(result => sub(result, x4))
```

Ok, how about rename `computeIfNormal` to `map`? 

```scala
val divResult:Data = div(x1,x2)

val sumResult:Data = divResult.map(result => sum(result, x3))

val sqrtResult:Data = sumResult match {
    case Normal(v) => sqrt(v)
    case e => e
}

val result:Data = sqrtResult.map(result => sub(result, x4))
```

Ahh, magic happen, it's really what we called `map` in Monad. Let's review what happend

1. We refactor a function to pure function by returning complex data type, such as `div`, `sqrt`
2. We can not compose the `pure function from born` using the complex data type directly, such as `sum`, `sub`
3. To reuse the `pure function from born`, we do lots of pattern match on the complex data type and we got lots of duplicated code.
4. These duplicated code have the same pattern, we extract a common function to do it, such as `computeIfNormal`
5. The common function does the same thing as `map` which come from Monad theory

So we can give a summary

1. We use `map` to reduce duplication
2. We use `map` to compose the function which is `pure function from born`, that is the function which doesn't return the complex data type.

# The second usage scenario

## What's the problem

From the last chapter, we can see there is still an ugly code

```scala
val sqrtResult:Data = sumResult match {
    case Normal(v) => sqrt(v)
    case e => e
}
```

We can't use `map(computeIfNormal)` to simplify it, because `sqrt` is not a `pure function from born`, it return a complex data type, we can't pass it to `map`.

And if we want to compute 
$$
x1 \div x2 \div x3
$$
Or
$$
\sqrt{\sqrt{x1}}
$$
We still can't use `map`. they all have the same pattern: two `pure function after refactor` which return complex data type want to compose.

## How to fix it?

We can create a new function to handle this scenario

```scala
def computeIfNormalForComplexFunction(data:Data)(f:Double=>Data):Data = data match {
    case Normal(v) => Normal(f(v))
    case _ => data
}

object Data {
    implicit DataOps(v:Data){
        def computeIfNormalForComplexFunction(f:Double => Data):Data =    
           computeIfNormalForComplexFunction(v)(f)
    }
}
```

Then the calculation expression become

```scala
val divResult:Data = div(x1,x2)

val sumResult:Data = divResult.map(result => sum(result, x3))

val sqrtResult:Data = sumResult.computeIfNormalForComplexFunction(result => sqrt(result))

val result:Data = sqrtResult.map(result => sub(result, x4))
```

Even more, we can rewirte the expression in one line now

```scala
val result = 
div(x1,x2).map(sum(_,x3)).computeIfNormalForComplexFunction(sqrt(_)).map(sub(_,x4))
```

I think it's good enough to persuade team to agree this refactor now, our code is still clean and we can leverage the benefit of FP.

## That's flatMap

Let's rename `computeIfNormalForComplexFunction` to `flatMap`, then the calculation expression become

```scala
val result = 
div(x1,x2).map(sum(_,x3)).flatMap(sqrt(_)).map(sub(_,x4))
```

Yeah, it is `flatMap` from Monad, Let's review what happend

1. To compose the `pure function after refactor`, we do lots of pattern match on the complex data type and we got duplicated code.
2. These duplicated code have the same pattern, we extract a common function to do it, such as `computeIfNormalForComplexFunction`
3. The common function does the same thing as `flatMap` which come from Monad theory

So we can give a summary

1. We use `flatMap` to reduce duplication
2. We use `flatMap` to compose the function which is `pure function after refacotr`, that is the function which return the complex data type.

# References

[^1]: [Functional Programming](https://en.wikipedia.org/wiki/Functional_programming)
[^2]: [Software design pattern](https://en.wikipedia.org/wiki/Software_design_pattern)
[^3]: [Monad](https://en.wikipedia.org/wiki/Monad_(functional_programming))

