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

Just like what we said in **Review map and flatMap from code**[^3] , we want to reduce the boilerplate

```scala
if (fileOpsInterpreter(saveDataToCSV(???,???)))
   fileOpsInterpreter(saveDataToCSV(???,???))
else
   false
```

Then we may refine the fileOpsInterpreter like this

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

def flatMap[B](v:FileOps[A])(f:A=>FileOps[B]):FileOps[B] = v match {
  case ReadFile(path) => f(fileOpsInterpreter(v))
  case SaveFile(path, content) => {
    val result = fileOpsInterperter(v)
    if(result) f(result) else ???
  }
}
```

Then we can refine `saveCSVFiles` like this

```scala
def saveCSVFiles(file1Content:String, file2Content:String, file3Content:String):Boolean = {
  fileOpsInterpreter(
    flatMap(fileOpsInterpreter(saveDataToCSV("data1.csv",file1Content)))(
    _ => flatMap(fileOpsInterpreter(saveDataToCSV("data2.csv",file2Content)))(
      _ => fileOpsInterpreter(saveDataToCSV("data3.csv",file3Content))
    )))
}
```

Oh yeah, We got a `flatMap`. But we find the `flatMap` and `fileOpsInterpreter` is not pure and there is one place we are not sure how to implement in `flatMap`.

# The type of Effect

In **Review map and flatMap from code**[^3] we talk about how to create an effect to make function pure and we know an effect need to have map and flatMap to make it easy to use. 

In this section we will talk about the type of effect and how to implement their map and flatMap.

It looks very simple to implement map and flatMap for **Data** effect, I call this effect as **Data-Effect** which contains some data will be used by the outside of function. But there is another type of effect, I call it as **Action-Effect** which describe an action which need to be interpreted by the outside of function.

## Data Effect

Data-Effect is a wrapper of data, such as Option, Either, List etc. the implementation of map and flatMap for these effect looks simple and obvious, just need to unwrap the effect and apply function on the data. Let's take a look the impementation of Option.

```scala
def map[A,B](option:Option[A])(f:A=>B):Option[B] = option match {
    case Some(v) => Some(f(v))
    case None => None
}

def flatMap[A,B](option:Option[A])(f:A=>Option[B]):Option[B] = option match {
    case Some(v) => f(v)
    case None => None
}
```

Even Reader monad is a Data-Effect, its data is a function, let's see its implementation of map and flatMap

```scala
case class Reader[A, B](run: A => B)
def map[A, B, C](reader: Reader[A, B])(f: B=>C):Reader[A, C] = 
	Reader((a:A) => f(reader.run(a)))
def flatMap[A, B, C](reader: Reader[A, B])(f: B=>Reader[A, C]): Reader[A, C] =
    Reader((a:A) => f(reader.run(a)).run(a))
```

## Action Effect

Let's see the following effect first

```scala
case User(name:String, age:Int)
trait HttpClient[A]
case class GetUser(name:String) extends HttpClient[User]
case class CreateUser(user:User) extends HttpClient[Int]
```

Can we implement map and flatMap for this effect?



# References

[^1]: [Side-Effect](https://en.wikipedia.org/wiki/Side_effect_(computer_science))
[^2]: [Extensible Effects: an alternative to Monad Transformers](http://okmij.org/ftp/Haskell/extensible/index.html)
[^3]: [Review map and flatMap from code](https://blog.shangjiaming.com/review-map-flatmap-from-code/#)

