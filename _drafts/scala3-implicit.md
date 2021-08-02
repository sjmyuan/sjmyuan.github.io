---
title: Scala3 Implicits
tags:
  - Scala3
---

Scala3 did lots of improvement in Implicits, which is most painful feature in Scala2.

The quick summary is

* Improve the error log to help developer find the correct implict instance.
* Split the implicits feature into given/using and extension to make the feature more clear.

Now let's have a taste of these feature.

## How pass parameter implicitly?

### Scala2 

```scala
def add(x: Int)(implicit y: Int): Int = x + y

implicit val instance: Int = 10

add(1)
add(1)(5)
```

### Scala3

```scala
def add(x: Int)(using y: Int): Int = x + y

given instance: Int = 10

add(1)
add(1)(5)
```

### Difference

* Replace `implicit` with `using` in function parameter
* Replace `implicit val` with `given` in instance definition

With these changes, the code readability is improved and the `implicit` word will not confuse us.


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

printListWithUsing(List(1, 2, 3, 4))
printListWithContextBound(List(1, 2, 3, 4))
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

printListWithUsing(List(1, 2, 3, 4))
printListWithContextBound(List(1, 2, 3, 4))
```

### Difference

Except the `given/using` changes, the context bound is same as Scala2

## How to add function to existing class?

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

1.show()

```

### Scala3

```scala
extension(v: Int){
 def show(): String = v.toString
}

1.toString
```

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

1.show()
List(1, 2, 3).show()
```

### Difference

In Scala2, we usually define a type class first, then use a implicit class to add the function to existing type.
In Scala3, it become straightforward, use a new keyword `extension` to do that. 
The magic thing is all the method in `extension` will be translated to a function, the following two expression are equal

```scala
extension(v: Int) {
  def show():String = v.toString
}

// equals to def show()(v: Int) = v.toString

1.show()
show()(1)
```
