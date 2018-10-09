---
title: Implicits and Variances
tags:
  - Scala
---

We have talked about Implicits[^1] and Variances[^2] detailedly, In this blog we will talk about an edge case when use them together.

Let's say we have a data type called `Data` and we also have a syntax sugar to convert `Any` to `Data`

```scala
trait Data[A]
case class Data1[A](v:A) extends Data[A]
object Data{
    implicit def any2Data(v:Any):Data[Any] = Data1(v)
}
```

Then we want to use it like this

```sh
$ val a:Data[Int] = 1
cmd22.sc:5: type mismatch;
 found   : Int(1)
 required: ammonite.$sess.cmd22.Data[Int]
val a:Data[Int] = 1
                  ^
Compilation Failed
```

The implicit function `any2Data` doesn't work. 

How about we change `Data` to be `Covariant`?

```scala
trait Data[+A]
```

```sh
$ val a:Data[Int] = 1
cmd22.sc:5: type mismatch;
 found   : Int(1)
 required: ammonite.$sess.cmd22.Data[Int]
val a:Data[Int] = 1
                  ^
Compilation Failed
```

Oops, same error! Can't we use the `any` implicit function on `Int`? Let's try `Contravariant`

```scala
trait Data[-A]
```

```sh
$ val a:Data[Int] =1
a: Data[Int] = Data1(1)
```

It works! Actually compiler will always try to find the exact implicit function. In `Covariant` and `Invariant`, the `Data[Any]` can't be treated as `Data[Int]`, so compiler won't use the implicit function. In `Contravariant`, `Data[Any]` is the child of `Data[Int]`, so compiler can use the implicit function to fix the compile error.

In practice we won't define the implicit function like the previous example, the following code is a better way

```scala
object Data{
    implicit def a2Data[A](v:A):Data[A] = Data1(v)
}
```

# References

[^1]: [Implicits](https://blog.shangjiaming.com/scala%20tutorial/implicit/)
[^2]: [Scala Variance in Inheritance](https://blog.shangjiaming.com/scala-variance-in-inheritance/)

