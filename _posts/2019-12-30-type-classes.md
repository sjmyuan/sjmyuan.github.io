---
title: Type Classes
tags:
  - Scala
categories:
  - Scala Tutorial
---

In this blog, we will talk about `Type Class` in Scala and try to find out why we need this pattern.

# Question

Say we already have a number of models in our code

```scala
case class Age(value: Int)
case class Person(name: String, age: Age)
case class Point(x: Int, y: Int)
```

One day, we get a new requirement which need to add a `def add(delta: Int)` method to all the models 

What should we do?

# Solution

In Java, if we want to add a `add` function to a class, we may just modify the class like this

```scala
case class Age(var value: Int){
  def add(delta: Int):Unit = 
    value += delta
}
```

This is not ideal in FP, we change the `Age` class to be mutable and modify the existing code which will increase the likelihood of errors.

Could we both keep the existing models unchanged and add the `add` function?

In Scala, we can use `implicit class` to do this

```scala
object Age {
  implicit class AgeOps(v: Age){
    def add(delta: Int): Age = Age(v.value + delta)
  }
}

object Person {
  implicit class PersonOps(v: Person){
    def add(delta: Int): Person = Person(v.name, v.age.add(delta)) // Age alreay has add function
  }
}

object Point {
  implicit class PointOps(v: Point){
    def add(delta: Int): Point = Point(v.x + delta, v.y + delta)
  }
}
```

```sh
$ val age = Age(18)
$ age.add(1) // Age(19)

$ val person = Person("Job", Age(18))
$ person.add(1) // Person("Job", Age(19))

$ val point = Point(1, 1)
$ point.add(1) // Point(2, 2)
```

Now we add the `add` function to the existing models, what if there is another function which want to invoke the `add` function?

After all we have the `add` function with same parameter on all the existing models, it make sense to have a function like this

```scala
def processAdd[A](a: A, delta: Int): A = {
  a match {
    case x: Age => x.add(delta).asInstanceOf[A]
    case x: Person => x.add(delta).asInstanceOf[A]
    case x: Point => x.add(delta).asInstanceOf[A]
    case _ => throw new Exception(s"Can not process ${a}")
  }
}
```

```sh
$ processAdd(Age(18), 1) // Age(19) 
$ processAdd(Person("Job", Age(18)), 1) // Person("Job", Age(19))
$ processAdd(Point(1, 1), 1) //Point(2, 2)
```

# Refactor

Let's see which part can be improved in the above code

1. `Age`, `Person` and `Point` have a function called `add`, but there is no hard rule to restrict them to use the same function name. For example, `Age` can change `add` to `addInt` without any error.
2. `processAdd` is ugly, it throw exception, use `asInstanceOf` and has duplicated code

How should we restrict the signature(including name) of `add` function on different models? 

The root cause of this problem is each model implement their own `implicit class`, maybe we can try to use type parameter on one `implicit class` like this

```scala
object AddSyntax {
  implicit class AddOps[A](v: A){
    def add(delta: Int): A = ???
  }
}
```

But how should we implement the function body? 

Different model has different implementation and we can't predict if there are new model using this function.

Seems what we can do now is to leave a hook here to allow the consumer of this function to decide the implementation

```scala
object AddSyntax {
  implicit class AddOps[A](v: A)(implicit f: (A, Int) => A){
    def add(delta: Int): A = f(v, delta)
  }
}
```

Then for each model, they just need to implement an implicit function `(A, Int) => A` like this

```scala
implicit def ageAddFunction(age: Age, delta: Int) = Age(age.value + delta)
implicit def personAddFunction(person: Person, delta: Int): Person = Person(person.name, person.age.add(delta)) // Age alreay has add function
implicit def pointAddFunction(point: Point, delta: Int): Point = Point(point.x + delta, point.y + delta)
```

```sh
$ val age = Age(18)
$ age.add(1) // Age(19)

$ val person = Person("Job", Age(18))
$ person.add(1) // Person("Job", Age(19))

$ val point = Point(1, 1)
$ point.add(1) // Point(2, 2)
```

Looks good for now, we have an unified signature for `add` function and we don't need to care about how each model implement the function `(A, Int) => Int` in the `implicit class`.

But what if we want to add another new function `sub`?

```sh
$ val age = Age(18)
$ age.sub(1) // Age(17)

$ val person = Person("Job", Age(18))
$ person.sub(1) // Person("Job", Age(17))

$ val point = Point(1, 1)
$ point.sub(1) // Point(0, 0)
```

Based on the above discussion, we need to implement it like this
```scala
object SubSyntax {
  implicit class SubOps[A](v: A)(implicit f: (A, Int) => A){
    def sub(delta: Int): A = f(v, delta)
  }
}

implicit def ageSubFunction(age: Age, delta: Int) = Age(age.value - delta)
implicit def personSubFunction(person: Person, delta: Int): Person = Person(person.name, person.age.sub(delta)) // Age alreay has sub function
implicit def pointSubFunction(point: Point, delta: Int): Point = Point(point.x - delta, point.y - delta)
```

But in terms of the implicit scope rule, we can't use `add` and `sub` together, 
because we need to import `sub` implicit function and `add` implicit function in one scope. 
Unfortunately they have the same signature and compiler will raise error. 

For example

```scala
implicit def ageSubFunction(age: Age, delta: Int) = Age(age.value - delta)
implicit def ageAddFunction(age: Age, delta: Int) = Age(age.value + delta)
```

`ageSubFunction` and `ageAddFunction` have different name but same signature, the compiler will raise error when finding them in one scope.

How should we avoid the signature conflict? 

We can't use object to wrap them here, because we need to import them together anyway. Then we only have two options: class or trait.

Let's try class first

```scala
class AddInterface[A] {
  def add(value: A, delta: Int): A
}

class SubInterface[A] {
  def sub(value: A, delta: Int): A
}
```

```scala
object AddSyntax {
  implicit class AddOps[A](v: A)(implicit addInstance: AddInterface[A]){
    def add(delta: Int): A = addInstance.add(v, delta)
  }
}

object SubSyntax {
  implicit class SubOps[A](v: A)(implicit subInstance: SubInterface[A]){
    def sub(delta: Int): A = subInstance.sub(v, delta)
  }
}
```

```scala
implicit val ageAddInstance = new AddInterface[Age] {
  override def add(age: Age, delta: Int): Age = Age(age.value + delta)
}

implicit val personAddInstance = new AddInterface[Person] {
  override def add(person: Person, delta: Int): Person = Person(person.name, person.age.add(delta)) // Age alreay has add function
}

implicit val pointAddInstance = new SubInterface[Point] {
  override def add(point: Point, delta: Int): Point = Point(point.x + delta, point.y + delta)
}

implicit val ageSubInstance = new SubInterface[Age] {
  override def sub(age: Age, delta: Int): Age = Age(age.value - delta)
}

implicit val personSubInstance = new SubInterface[Person] {
  override def sub(person: Person, delta: Int): Person = Person(person.name, person.age.sub(delta)) // Age alreay has sub function
}

implicit val pointSubInstance = new SubInterface[Point] {
  override def sub(point: Point, delta: Int): Point = Point(point.x - delta, point.y - delta)
}
```

```sh
$ val age = Age(18)
$ age.sub(1).add(1) // Age(18)

$ val person = Person("Job", Age(18))
$ person.sub(1).add(1) // Person("Job", Age(18))

$ val point = Point(1, 1)
$ point.sub(1).add(1) // Point(1, 1)
```

It works, we can use `add` and `sub` together now! 

But wait, seems class here doesn't give us more value than trait, let's use trait replace class

```scala
trait AddInterface[A] {
  def add(value: A, delta: Int): A
}

trait SubInterface[A] {
  def sub(value: A, delta: Int): A
}
```

Now we refactor the first part, let's see if we can get a better `processAdd` based the above refactor

```scala
def processAdd[A: AddInterface](a: A, delta: Int): A = a.add(delta)
```

Pretty simple! We use context bound here which only allow the type with `add` function to use `processAdd`. 
In this way, we don't need to throw exception and check the type of parameter one by one.

# Summary

* To unify the signature of `add` function, we involve `implicit class` with type parameter and hook function

  ```scala
  object AddSyntax {
    implicit class AddOps[A](v: A)(implicit f: (A, Int) => A){
      def add(delta: Int): A = f(v, delta)
    }
  }
  ```

* To avoid the signature conflict of hook functions, we involve trait to wrap them

  ```scala
  trait AddInterface[A] {
    def add(value: A, delta: Int): A
  }
  ```

* To generate the instance of `AddOps` for given type, we need to create a implicit instance of hook trait 

  ```scala
  implicit val ageAddInstance = new AddInterface[Age] {
    override def add(age: Age, delta: Int): Age = Age(age.value + delta)
  }
  ```

This is `Type Class`, it consist of three parts:

* Interface

  It unify group of operations which can be applied to existing models without changing them.

  ```scala
  trait DoSomething[A] {
    def doSomething(a:A)
  }
  ```

* Instances

  It implements the real logic of `DoSomething` for given type

  ```scala
  implicit val someTypeDoSomething = new DoSomething[SomeType] {
    def doSomething(a: SomeType) = ???
  }
  ```

* Consumer

  It is the code which want to use the `DoSomething` operations of given type, it can be an `implicit class` or a function

  ```scala
  implicit class DoSomethingOps[A: DoSomething](v: A) {
    def doSomething(a: A) = implicitly[DoSomething[A]].doSomething(a)
  }
  ```

  ```scala
  def processDoSomething[A: DoSomething](a: A) = ???
  ```

`Type Class` is heavily used by the Scala FP library. Usually the library will supply the `Interface`, `Consumer` and some of the `Instances` of popular data models.
When we want to use the library, we need to implement the `Instances` of our custom data models or define our own `Consumer`.

For example, `Show` is a `Type Class` of `Cats`. 
`Cats` already defined the `Show Interface` and `Show Syntax(Consumer)`.
If we want our class `Age` to support `Show`, we need to do it like this.

```scala
object Age {
  implicit val ageShow = new Show[Age] {
    override def show(age: Age): String = age.value.toString
  }
}
```

```sh
$ val age = Age(18)
$ age.show // "18"
```
