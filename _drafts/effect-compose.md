---
title: How to compose Effect?
tags:
  - Scala
---
In **What is Effect?**[^1], we talk about the definition of Effect and how to use it in pure way.
In this blog, we will talk about how to use two(even more) effects together in pure way.

# What is the problem?

## The first scenario

```scala
def sqrt(v:Double):Option[Double] = 
  if (v <0) None else Some(Math.sqrt(v))

def div(numerator:Double, denominator:Double):Option[Double] =
  if (denominator == 0) None else Some(numerator/denominator)

def sum(left:Double, right:Double):Either[String, Double] = Right(left + right)

def sub(left:Double, right:Double):Either[String, Double] = Right(left - right)
```

$$
\sqrt{x1 \div x2+x3}-x4
$$

```scala
val divResult:Option[Double] = div(x1,x2)
val result:Either[String, Double] = divResult match {
    case Some(v) =>
       val sumResult:Either[String, Double] = sum(v, x3)
       val sqrtResult:Option[Double] = sumResult match {
           case Right(v1) => sqrt(v1)
           case Left(e) => None
       }
       sqrtResult match {
           case Some(v1) => sub(v1,x4)
           case None => Left("error")
       }
    case None => Left("error")
}
```

```scala
type Composed[A] = Either[String, Option[A]]

def map[A](v:Composed[A])(f: A => B):Composed[B] = v match {
    case Right(Some(v1)) => Right(Some(f(v1)))
    case _ => v
}

def flatMap[A](v:Composed[A])(f: A => Composed[B]):Composed[B] = v match {
    case Right(Some(v1)) => f(v1)
    case _ => v
}

def liftOption[A](v:Option[A]):Composed[A] = Right(v)

def liftEither[A](v:Either[String, A]):Composed[A] = v.map(Some(_))
```

```scala
val divResult:Composed[Double] = liftOption(div(x1,x2))
val result:Either[String, Double] = divResult.flatMap(y => {
    val sumResult:Composed = liftEither(sum(y, x3))
    val sqrtResult:Composed = sumResult.flatMap(x=>liftOption(sqrt(x)))
    sqrtResult.flatMap(x=>liftEither(sub(x,x4)))
}) 
```

## The second scenario

# Solutions

## Moand Transformer

## EitherK

# Further discussion

# References
