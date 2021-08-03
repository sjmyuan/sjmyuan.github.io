---
title: Scala3 Implicit
tags:
- Scala3
date: 2021-08-03 22:22 +0800
---

Scala3 did lots of improvement in Implicit, which is the most painful feature in Scala2.

A quick summary is

* Improve the error log to help the developer find the correct implicit instance.
* Split the implicit feature into given/using and extension to make the feature more clear.

Now let's have a taste of these features.

## How to pass parameters implicitly?

### Scala2 

```scala
def add(x: Int)(implicit y: Int): Int = x + y

implicit val instance: Int = 10 // :Int = 10

add(1) // :Int = 11
add(1)(5) // :Int = 6
```

### Scala3

```scala
def add(x: Int)(using y: Int): Int = x + y

given instance: Int = 10

add(1) // :Int = 11
add(1)(using 5) // :Int = 6
```

### Difference

* Replace `implicit` with `using` in function parameter
* Replace `implicit val` with `given` in instance definition
* Use `using` explicitly when giving explicit value

With these changes, the code readability is improved. 
At least we can search `given` to find the implicit instances and search `using` to find the implicit parameters. 


## How to use context bound?

### Scala2

```scala
trait Show[A] {
 def show(x: A): String
}

def printListWithUsing[A](list: List[A])(implicit show: Show[A]): String = list.map(x => show.show(x)).mkString(";")
def printListWithContextBound[A:Show](list: List[A]): String = list.map(x => implicitly[Show[A]].show(x)).mkString(";")

implicit val intShow: Show[Int] = new Show[Int] {
  def show(x: Int): String = x.toString 
}

printListWithUsing(List(1, 2, 3, 4)) // :String = "1;2;3;4"
printListWithContextBound(List(1, 2, 3, 4)) // :String = "1;2;3;4"
```

### Scala3

```scala
trait Show[A] {
 def show(x: A): String
}

def printListWithUsing[A](list: List[A])(using show: Show[A]): String = list.map(x => show.show(x)).mkString(";")
def printListWithContextBound[A:Show](list: List[A]): String = list.map(x => summon[Show[A]].show(x)).mkString(";")

given intShow: Show[Int] with {
  def show(x: Int): String = x.toString 
}

printListWithUsing(List(1, 2, 3, 4)) // :String = "1;2;3;4"
printListWithContextBound(List(1, 2, 3, 4)) // :String = "1;2;3;4"
```

### Difference

* Replace `implicitly` with `summon`
* Replace `new class` with `with`

  Not like Scala2, we don't need to new the implicit class explicitly, all the members of class will be wrapped by `with` keyword.

  ```scala
  implicit val intShow: Show[Int] = new Show[Int] { ... } // Scala2
  given intShow: Show[Int] with { ... } // Scala3
  ```

## How to add function to an existing class?

### Scala2

```scala
trait Show[A] {
 def show(x: A): String
}

implicit class ShowOps[A: Show](v: A) {
  def show(): String = implicitly[Show[A]].show(v)
}

implicit val intShow: Show[Int] = new Show[Int] {
  def show(x: Int): String = x.toString 
}

implicit def listShow[A: Show]: Show[List[A]] = new Show[List[A]] {
  def show(v: List[A]): String = v.map(_.show()).mkString(";") 
}

1.show() // :String = "1"
List(1, 2, 3).show() // :String = "1;2;3"
```

### Scala3

```scala
trait Show[A] {
 extension (v: A) {
   def show(): String
 }
}

given intShow: Show[Int] with {
  extension (v: Int) {
    def show(): String = v.toString()
  }
}

given listShow[A: Show]: Show[List[A]] with {
  extension (v: List[A]) {
    def show(): String = v.map(_.show()).mkString(";")
  }
}

1.show() // :String = 1
List(1, 2, 3).show() // :String = "1;2;3"
```

### Difference

In Scala2, we usually define a type class first, then use an implicit class to add the function to the existing type.

In Scala3, it becomes straightforward, use a new keyword `extension` to do that. 

The magic thing is all the methods in `extension` will be translated to functions, the following two expressions are equal

```scala
extension(v: Int) {
  def demo():String = v.toString
}

// equals to def demo(v: Int)() = v.toString

1.demo() // :String = 1
demo(1)() // :String = 1
```

## Summary

According to my experience, Scala3 implicit is easier than Scala2, but there are still complicated rules for implicit finding, will talk about them later.

You can find the demo code here

* [Scala2](https://gist.github.com/sjmyuan/b17cccaecea669d88e65b9c89f3efeb5)
* [Scala3](https://gist.github.com/sjmyuan/e588de4ec27b735d8cda97050237fd8d)

