---
title: Pattern Matching
tags:
  - Scala
categories:
  - Scala Tutorial
---

Pattern matching is a powerful tool in Scala. it has a common syntax like

```scala
v match {
    case v1 if ??? => ???
    case v2 => ???
    case _ => ???
}
```



In this chapter we will discuss how to use it in different scenarios

## How to match literal value?

Let's say we have an int variable `v:Int` , we want to implement the following logic

- if `v==1` then print `This is 1`
- if `v==2` then print `This is 2`
- for the others just print `I don't care about this number`

Usually, we can do it by `if-else`

```scala
if(v==1)
    println("This is 1")
else if (v==2)
    println("This is 2")
else
    println("I don't care about this number")
```

We also can do it by `pattern-matching`

```scala
v match {
    case 1 => println("This is 1")
    case 2 => println("This is 2")
    case _ => println("I don't care about this number")
}
```

In this block we just give the value to each branch directly and use `_` to cover the rest of values. `_` is a powerful symbol in scala, it has different meanings in different scenario, we will talk about it detailedly in other chapter. In this block, it stand for `any value of any type`.

Pattern matching also support the `if-guard`, the previous example can also be implemented like this

```scala
v match {
    case _ if v == 1 => println("This is 1")
    case _ if v == 2 => println("This is 2")
    case _ => println("I don't care about this number")
}
```

The meaning of `case _ if v == 1`is if any value of any type equals to 1 then do something.

There is also an `OR` logic in pattern matching, try following example

```scala
v match {
    case 1|2 => println("This is 1 or 2")
    case _ => println("I don't care about this number")
}
```

## How to match variable?

This scenario is an edge case, but it's very interesting. Let's say we have a function like this

```scala
def dosomething(v:Int,given:Int):Unit ={
    .....
}
```

We want this function to print a log `got it`when `v==given`, how do we implement it using pattern matching?

Could we do it like this?

```scala
def dosomething(v:Int, given:Int):Unit ={
    v match {
        case given => println("got it")
        case _ => println("no idea")
    }
}
```

When we run this code, we will find this function print `got it`for any given parameter, Why?

The meaning of `case given => println("got it")`here is assign any values of any type to `given` then do something.

The compiler treat `given` as a new variable which contains the value of `v`, not the value of parameter `given`. To make compiler do the right thing, we need to use ```given`   ``   like this

```scala
def dosomething(v:Int, given:Int):Unit ={
    v match {
        case `given` => println("got it")
        case _ => println("no idea")
    }
}
```

## How to match type?

Somethimes we want to do differnt thing for different type, for example

* If the type is Int, then print `This is Int`
* If the type is String, then print `This is String`
* For other types, just print `no idea`

We can implement it using pattern matching like this

```scala
def dosomething(v:Any)={
    v match {
        case _:Int => println("This is Int")
        case _:String => println("This is String")
        case _ => println("no idea")
    }
}
```

We see `case _` agagin ! the last one has the same meaning as previous examples. for `case _:Int=>`, it means do something for any values of Int type.

It's pretty simple, right? How about change Any to List[Any]? does this code work well?

```scala
def dosomething(v:List[Any])={
    v match {
        case _:List[Int] => println("This is Int")
        case _:List[String] => println("This is String")
        case _ => println("no idea")
    }
}
```

Opps, we get a wrong answer!

```bash
$ dosomething(List("A","B","C"))
This is Int
```

Actually when we compile this code, we will get a warning

```sh
Warning:(43, 19) non-variable type argument String in type pattern List[String] (the underlying of List[String]) is unchecked since it is eliminated by erasure
          case _: List[String] => "String"
```

Scala is a JVM language, the Scala code will be translated to Java code. The type patameter in Java Generics will be erased. So for higher kind type in Scala, we can't match the inner type.

## How to match case class?

case class plays an important role in Scala, it give us a bunch of utility methods and sugar syntaxs. we will have a look at how it works in pattern matcing.

Unlike the common class, we can create a case class instance by class name

```scala
case class MyClass(id:Int)
val myClass = MyClass(1)
```

In pattern matching, we also can get the content of case class by class name

```scala
def extractId(v:Any)={
    v match {
        case MyClass(id) => println(s"The id is ${id}")
        case _ => println("No idea")
    }
}
```

In this example, `case MyClass(id)=>` will do three things

* check if the type of v is `MyClass`
* assign the id of `MyClass` to variable `id`
* do some logic

We must give the same number of variable as the number of constructor parameter, or we will get a compile error.

We also can apply the type matching and if-guard on the case class matching

```scala
case class MyClass(info:Any)
def extractIntInfo(v:MyClass) = {
    v match {
        case MyClass(id:Int) if id > 0 => println(s"The id is ${id}")
        case _ => println("No idea")
    }
}
```

```sh
$ extractIntInfo(MyClass(1))
The id is 1
$ extractIntInfo(MyClass(-1))
No idea
$ extractIntInfo(MyClass("hello"))
No idea
```

Usually we will use nested case class, how do we extract the information of nested case class? Let's see a example

```scala
case class Name(name:String)
case class Age(age:Int)
case class Address(address:String)
case class Person(name:Name,age:Age,address:Address)
def getAddress(p:Person):String = {
    ???
}
```

We can implement `getAddress` like this

```scala
def getAddress(p:Person):String = {
    p match {
        case Persion(_,_,Address(a)) => println(s"The address is ${a}")
        case _ => println("No idea")
    }
}
```

You may noticed we use `_` for `name` and `age` value, because we don't care about these two value. for `Address`, we just use the non-nested case class matching directly.

What if we want to get `Address`and `Address.address` together? Try the following example

```scala
def getAddress(p:Person):String = {
    p match {
        case Persion(_,_,v@Address(a)) =>
        	println(s"The Address is ${v}, Address.address is ${a}")
        case _ => println("No idea")
    }
}
```

We can bind the nested pattern to a variable using `@`

Beside the previous examples, we may get a case class like this

```scala
case class MyClass(info:String*)
```

`info:String*`is a variable argument list, its length is not fixed, how should we match it? 

Let't try this example

```scala
def getInfo(v:MyClass):List[String] = {
    v match {
        case MyClass(x@_*) => x.toList
        case _ => List()
    }
}
```

```sh
$ getInfo(MyClass("hello","world"))
res2: List[String] = List("hello", "world")
$ getInfo(MyClass())
res3: List[String] = List()
```

`x@_*` will assign the variable parameter list to `x`

## How to match collection?

In this part, we will talk about two topics: Tuple and List

### Tuple

Tuple is simple, we can just treat it as a special case class which has no class name. we can do pattern matching like this

```scala
def extractInfo(v:(Any,Any)) = {
    v match {
        case (k:Int,v:String) => println(s"Key:${k},Value:${v}")
        case _ => println("No idea")
    }
}
```

```sh
$ extractInfo((1,"ok"))
Key:1,Value:ok
$ extractInfo((1,2))
No idea
$ extractInfo(("hello","world"))
No idea
```

### List

Scala List is very differnt from the List in Java, it's a recursion data structure, let's see how can we play it with pattern matching.

We can get the head of List

```scala
def getHead(l:List[Int]) = {
    l match{
        case head::tail => println(s"The head is ${head}")
        case _ => println("No idea")
    }
}
```

```sh
$ getHead(List(1,2,3))
The head is 1
$ getHead(List())
No idea
```

We also can iterate every element in List

```scala
def iterate(l:List[Int]):Unit = {
    l match{
        case head::tail => 
          println(head)
          iterate(tail)
        case _ => 
          println("end")
    }
}
```

```sh
$ iterate(List(1,2,3))
1
2
3
end
$ iterate(List())
end
```

If we know the exact length of List, we also can get all the elements at the same time

```scala
def getThreeElements(l:List[Int]) = {
    l match {
        case List(e1,e2,e3) => 
        	println(e1)
            println(e2)
            println(e3)
        case _ => println("No idea")
    }
}
```

```sh
$ getThreeElements(List(1,2,3))
1
2
3
$ getThreeElements(List(1,2))
No idea
$ getThreeElements(List())
No idea
```

In these examples, List has two pattern in pattern matching

* `case head::tail=>`

  Iterate the element of List from left to right

* `case List(e1,e2,e3) =>`

  Get all the elements of List if the length of List is equal to 3

## How to match regex?

Sometimes we need to check if the given string follows some pattern(integer etc.) or extract some information from given string, let's see how to implement them using pattern matching.

Try this example to check if the input string is an integer

```scala
def isInt(s:String) ={
    val intRegex = "\\d*".r
    s match {
        case intRegex() => println(s"${s} is an integer")
        case _ => println(s"${s} is not an integer")
    }
}
```

```sh
$ isInt("1")
1 is an integer
$ isInt("hello")
hello is not an integer
```

Try this example to extract the name for string `Name=bob`

```scala
def getName(s:String) = {
    val nameRegex = "Name=(.*)".r
    s match {
        case nameRegex(n) => println(s"The name is ${n}")
        case _ => println("No idea")
    }
}
```

```sh
$ getName("Name=bob")
The name is bob
$ getName("Wow!")
No idea
```

The regex in pattern matching will always try to extract something, so even you don't have group in your regex expression you also need a `()` following the regex.

## The dark magic of unapply

The source of truth for pattern matching is `unapply` , the pattern matching is just a sugar syntax of `unapply`. 

For this simple code

```scala
v match{
    case MyClass(a) => ???
}
```

We can rewrite it using `unapply`

```scala
val aOption:Option[A]=MyClass.unapply(v)
if(aOption.nonEmpty){
    val a = aOption.get
    ???
}
```

The declaration of `unapply`

* extract single value

  ```scala
  object MyClass{
      def unapply(v:MyClass):Option[A]
  }
  ```

  `A` can be any type

* extract multiple values

  ```scala
  object MyClass{
      def unapply(v:MyClass):Option[(A1,A2,A3,A4)]
  }
  ```

* extract unfixed values(the number of values is not fixed)

  ```scala
  object MyClass{
      def unapplySeq(v:MyClass):Option[Seq[A]]
  }
  ```

Let's see some declaration of `unapply` we used in this chapter, they have been implemented by Scala.

* case class

  ```scala
  case class MyClass(id:Int)
  object MyClass{
      def unapply(v:MyClass):Option[Int] = Some(v.id)
  }
  ```

* List

  ```scala
  object :: {
      def unapply[A](v:List[A]):Option[(A,List[A])] = {
          if(v.isEmpty) None
          else Some(v.head,v.tail)
      }
  }
  ```

  ```scala
  object List{
      def unapplySeq[A](v:List[A]):Option[Seq[A]] = Some(v)
  }
  ```

We can define a custom extractor by `unapply`, it will give us more power.

Try this example, extract the even numbers from List

```scala
object Even{
    def unapply(v:List[Int]):Option[List[Int]] = Some(v.filter(_%2==0))
}

def getEven(v:List[Int]):List[Int] = {
    v match {
        case Even(l) => l
        case _ => Nil
    }
}
```

```sh
$ getEven(List(1,2,3,4,5))
res44: List[Int] = List(2, 4)
$ getEven(List())
res45: List[Int] = List()
```