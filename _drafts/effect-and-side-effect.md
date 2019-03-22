---
title: What is Effect?
tags:
  - Scala
---

In the world of Functional Programming, we are familiar with the Side-Effect[^1], but we rarely mentioned Effect. In this blog, we will try to explain it from code.

# What is Effect?

**Extensible Effects: an alternative to Monad Transformers**[^2] give some definition of Effect

> An effect is most easily understood as an interaction between a sub-expression and a central authority that administers the global resources of a program.
>
> An effect can be viewed as a message to the central authority plus enough information to resume the suspended calculation.​ 

It's a little bit long and hard to understand for me, I would like to say **An effect is just the return value of function**

In the world of FP, function is the first class member and it should be pure. A function is assumed to interact with the outside world only by the input and return value, if it build connection with outside by other way, we say these ways are **Side-Effect**.

There are two type of interaction can be done between function and the outside of function.

* Receive message from outside

  A function usually uses input to receive message, but we also can implement it by return value, we will talk about it later.

* Return message to outside
  A function without Side-Effect have to return message to outside by return value, the message returned by function is **Effect**.

# Interpreter

All effects returned by function should have an interpreter which can be used to understand the effect by outside. 

For example, when we invoke a function which return Option, we always need a pattern-match to interpret the return value

```scala
def div(v1:Double, v2:Double):Option[Double] = if(v2==0) None else Some(v1/v2)

def optionInterpreter(v:Option[Double]):String = v match {
  case Some(v1) => s"The result is ${v1}"
  case None => "The denominator can not be zero"
}
```

```sh
$ optionInterpreter(div(1,2))
The result is 0.5
$ optionInterpreter(div(1,0))
The denominator can not be zero
```

Let's see another example about file operations, we have an Effect to stand for different type of file operations

```scala
sealed trait FileOps[A]
case class ReadFile(path:String) extends FileOps[String]
case class SaveFile(path:String, content:String) extends FileOps[Boolean]
```

We also have some functions which return the FileOps effect

```scala
def getConfiguration:FileOps[String] = ReadFile("./config.xml")
def saveDataToCSV(name:String, content:String):FileOps[String] = SaveFile(s"./tmp/${name}",content)
```

These functions are pure function, but pure function can't help us to create/read a file, so we need an interpreter to do these things

```scala
def fileOpsInterpreter[A](v:FileOps[A]):A = v match {
  case ReadFile(path) => Source.fromFile(path).mkString
  case SaveFile(path, content) => try {
    val file = new File(path)
    val bw = new BufferedWriter(new FileWriter(file))
    bw.write(content)
    bw.close()
    true
  } catch {
    case e => false
  }
}
```

Then our final program will be

```sh
$ fileOpsInterpreter(getConfiguration)
pool_zide=1
port=8080

$ fileOpsInterpreter(saveDataToCSV("data1.csv","name,id\nJon,1001"))
true
```

How about save multiple csv files?

```scala
def saveCSVFiles(file1Content:String, file2Content:String, file3Content:String):Boolean = {
  if (fileOpsInterpreter(saveDataToCSV("data1.csv",file1Content)))
    if(fileOpsInterpreter(saveDataToCSV("data2.csv",file2Content)))
       fileOpsInterpreter(saveDataToCSV("data3.csv",file3Content))
    else
       false
  else 
    false
}
```

we want to reduce the boilerplate

```scala
if (fileOpsInterpreter(saveDataToCSV(???,???)))
   fileOpsInterpreter(saveDataToCSV(???,???))
else
   false
```

Then we may add a flatMap like this

```scala
def flatMap[B](v:FileOps[A])(f:A=>FileOps[B]):FileOps[B] = v match {
  case ReadFile(path) => f(fileOpsInterpreter(v))
  case SaveFile(path, content) => {
    val isSuccess = fileOpsInterperter(v)
    if(isSuccess) f(result) else ???
  }
}
```

Then we can refine `saveCSVFiles` like this

```scala
def saveCSVFiles(file1Content:String, file2Content:String, file3Content:String):Boolean = {
  fileOpsInterpreter(
    flatMap(saveDataToCSV("data1.csv",file1Content))(
    _ => flatMap(saveDataToCSV("data2.csv",file2Content))(
      _ => saveDataToCSV("data3.csv",file3Content)
    )))
}
```

But we found the `flatMap` and `fileOpsInterpreter` are not pure and there is one place we are not sure how to implement in `flatMap`(we don't know which effect should be returned when the SaveFile operation failed).

At this stage, we can say we can't implement a pure `flatMap` for `FileOps` effect. Then what should we do? Can't we use the effect whose interpreter looks like `FileOps` ?

# A special Effect

As we can see in the last section, we need to solve two problem to make the `flatMap` of `FileOps` pure

* Side-Effect introduced by `fileOpsInterpreter`
* Definite effect returned by `flatMap`

Just like what we said in **Review map and flatMap from code**[^3] , we can create an effect to remove the Side-Effect

```scala
sealed trait SpecialEffect[A]
case class FlatMap[A, B](v:FileOps[A], f:A=>FileOps[B]) extends SpecialEffect[B]
```

The `flatMap` of `FileOps`, `interpreter` of `SpecialEffect` can be implemented like this

```scala
def flatMap[B](v:FileOps[A])(f:A=>FileOps[B]):SpecialEffect[B] = FlatMap(v, f)
def interpreter[A](v:SpecialEffect[A]):A = v match {
  case FlatMap(v1,f1) => fileOpsInterpreter(f1(fileOpsInterpreter(v1)))
}
```

The `flatMap` is pure. but it is not a `flatMap` now, because its return type is not `FileOps`. Seems it's a conflict, we make it pure but we can't make it follow the pattern of `flatMap`. 

Let's try a different way, if it is not possible to implement a `flatMap` for `FileOps` and we can use `SpecialEffect` to make the ideal `flatMap` pure, how about put the `FileOps` in `SpecialEffect`? then we can get a `flatMap` for `SpecialEffect` which contains all the information of `FileOps`. Let's implement a function to do this

```scala
def lift[A](v:FileOps[A]):SpecialEffect[A] = FlatMap(v, ???)
```

Yeah, we don't know how to implement the `f` of `FlatMap`, our idea is the `f` should be an identity function which won't do anything for the `v:FileOps`, but there is no way to use `FileOps` to stand for identity. 

So we have to refine the `SpecialEffect` like this

```scala
sealed trait SpecialEffect[A]
case class Pure[A](v:A) extends SpecialEffect[A]
case class FlatMap[A, B](v:FileOps[A], f: A=>SpecialEffect[B]) extends SpecialEffect[B]
```

Then the `lift` can be implemented like this

```scala
def lift[A](v:FileOps[A]):SpecialEffect[A] = FlatMap(v, x => Pure(x))
```

Cool, we wrapped the `FileOps` with `SpecialEffect` without changeing it, let's see the `flatMap` and `interpreter`

```scala
def flatMap[B](v:SpecialEffect[A])(f:A=>SpecialEffect[B]):SpecialEffect[B] = v match {
  case Pure(v1) => f(v1)
  case FlatMap(v1, f1) => FlatMap(v1, x => flatMap(f1(x))(f))
}

def interpreter[A](v:SpecialEffect[A]):A = v match {
  case Pure(v1) => v1
  case FlatMap(v1,f1) => interpreter(f1(fileOpsInterpreter(v1)))
}
```

Then the `saveCSVFiles` can be refined like this

```scala
def saveCSVFiles(file1Content:String, file2Content:String, file3Content:String):Boolean = {
  interpreter(
    flatMap(lift(saveDataToCSV("data1.csv",file1Content)))(
    _ => flatMap(lift(saveDataToCSV("data2.csv",file2Content)))(
      _ => lift(saveDataToCSV("data3.csv",file3Content))
    )))
}
```

Ok the only thing is implement the if-else logic



# References

[^1]: [Side-Effect](https://en.wikipedia.org/wiki/Side_effect_(computer_science))
[^2]: [Extensible Effects: an alternative to Monad Transformers](http://okmij.org/ftp/Haskell/extensible/index.html)
[^3]: [Review map and flatMap from code](https://blog.shangjiaming.com/review-map-flatmap-from-code/#)

[^4]: [Free Monad](https://typelevel.org/cats/datatypes/freemonad.html)

