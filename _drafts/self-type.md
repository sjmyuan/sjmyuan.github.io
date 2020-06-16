---
title: Self-types
tags:
  - Scala
categories:
  - Scala Tutorial
---

In this blog, we will talk about [Self-types](https://docs.scala-lang.org/tour/self-types.html) which is not used very often,
but it's an important concept to understand Cake Pattern, so it's worth to do a simple introduce.

# Nested trait/class definition

Sometimes we want to define a trait/class in another trait/class, like this

```scala
trait Animal {
  val name: String
  trait Action {
    def doAction: Unit = println(s"${name} is running")
  }
}

object Main {
  def main(): Unit = {
    val dog = new Animal { val name = "Dog" }
    val dogAction = new dog.Action {}
    dogAction.doAction // Dog is running

    val cat = new Animal { val name = "Cat" }
    val catAction = new cat.Action {}
    catAction.doAction // Cat is running
  }
}
```

## Problem

What if the inner trait want to access a variable/function in outter trait,
but it has the same name with one variable/function of inner trait?

```scala
trait Animal {
  val name: String
  trait Action {
    val name: String
    def doAction: Unit = println(s"${name} is ${this.name}")
  }
}

object Main {
  def main(): Unit = {
    val dog = new Animal { val name = "Dog" }
    val dogAction = new dog.Action { val name = "running" }
    dogAction.doAction // running is running

    val cat = new Animal { val name = "Cat" }
    val catAction = new cat.Action { val name = "running" }
    catAction.doAction // running is running
  }
}
```

We can see the output is not what we expected, the `Action` can't access the `name` in `Animal`

## Solution

To solve the problem, we need a reference of `Animal` in `Action`, `Self-types` can do this

```scala
trait Animal { self =>
  val name: String
  trait Action {
    val name: String
    def doAction: Unit = println(s"${self.name} is ${this.name}")
  }
}

object Main {
  def main(): Unit = {
    val dog = new Animal { val name = "Dog" }
    val dogAction = new dog.Action { val name = "running" }
    dogAction.doAction // Dog is running

    val cat = new Animal { val name = "Cat" }
    val catAction = new cat.Action { val name = "running" }
    catAction.doAction // Cat is running
  }
}
```

We get the correct output now, the magic is made by

```scala
self =>
```

This line give an alias of `this` in `Animal`, which can be referred directly in `Action`. The alias can be any valid identifier.

# Trait mix in

## Problem

## Solution

# Summary
