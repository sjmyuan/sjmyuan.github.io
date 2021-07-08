---
title: Why we need contravariant?
tags:
  - fp
---

In Scala, Contravariant is one of the most confused concept. In this blog, I will show you why we need it.

# What is Subtyping?

Subtyping is the core feature of OO language, it is inheritance and if A is subtype type of B, they will follow the [subsitution principle](https://en.wikipedia.org/wiki/Liskov_substitution_principle)

> A variable of a given type may be assigned a value of any subtype, and a method with a parameter of a given type may be invoked with an argument of any subtype of that type.

So if type A and B don't follow the subsitution principle, they can not be subtyping relationship.

# How to understand the subsitution of function?

For function, when we say two function have same type, we mean their parameter type and return type are same.

Let's start from simple variable assignment.

If two function variables have same type, it should be able to assign the value directly

```scala
val f1: Int => Int = (x:Int) => x + 1

val f2: Int => Int = f1

f2(1) // 2
```

Say we have a function `f`, and we want to assign it to a function variable `g`.
And let's give an obvious definition of function assignment

> if a function f can be assigned to a function variable g, then we should be able to replaces all the g with f in our code.

Let's discuss the assignment in the following scenarios

## Same return type

### Same parameter number

#### One parameter

* If the parameter of `f` and `g` don't have subtyping relationship, the assignment can't work.

  ```scala
  val f: Int => Int = (x:Int) => x + 1

  val g: String => Int = f1

  g("hello world") // f("hello world") can't work
  ```

* If the parameter of `f` is the parent of parameter of `g`, the assignment can work.

```scala
val f: Any => String = (x:Any) => x.toString

val g: Int => String = f1

g(1) // f(1) can work
```

* If the parameter of `f` is the child of parameter of `g`, the assignment can't work.

```scala
val f: Int => String = (x:Int) => (x+1).toString

val g: Any => String = f1

g("hello world") // f("hello word") can't work
```

#### More than one parameter

### Different parameter number

## Different return type


it is easy to understand the subsitution principle of object, if we have a variable with type A and B is the subtype of A, then we can assign the instance of B to A.

```scala

sealed trait Animal
case object Cat extends Animal
case object Dog extends Animal

val animal:Animal = Cat // success
```

Then how should we understand the subsitution of function? 
We know function can be identified by its signature which is composed by parameter and return type, we can use `Function[Input, Output]` to stand for the function type.
If `Function[Ain, Aout]` and `Function[Bin, Bout]` are function types and the variable of `Function[Ain, Aout]` can be assigned to instance of `Function[Bin, Bout]`, then 

```scala
val f: Ain => Aout = (Bin) => {???}
```

the following expression should work

# How to define a subtyping of generic function?

## What is contravariant? 

# Summary
