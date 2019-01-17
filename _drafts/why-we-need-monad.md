---
title: Why We Need Monad?
tags:
  - Scala
---

Monad should be the most popular word we heared in functional programming. we spent lots of time to learn and understand it. Most of time we will end with the category theory.

> Monad is a function on endor-functor

What does it means??? 

We have to open the mathematics books and learn Group, Semi-Group, Monoid, Functor, Apply, Applicative, Monad.

Why we need to learn them? we are developer, I just want to know why we have to use it in funtional programming, what problem it can resolved, I don't want to learn the therory.

That's the key point, we always just learn the solution but don't know what's the problem for this solution.

In this blog, we will discuss the functional programming from the problems we meet when we use it and see how to resovle them, then we will understand why we need the Monad.

# The Headstone of Functional Programming

I don't want to talk about what is FP and why we need FP here, I just want to talk about the basic rule of FP

> Every expression should be pure in Functional Programming

To write a pure function, we need to use the result to stand for all the state of this function, let's look at the following example

```scala
def string2Int(v:String):Int = v.toInt
```

Is it a pure function? 

Of course not, it will throw an exception when you invoke it like this

```scala
string2Int("hello world")
```

How do we make it pure? or How do we make an impure function to pure?

To answer this question, let's look at the defination of pure function

> The result only depend on the parameter, there is no other effect except the result

So to make it pure, the result of this function should be able to stand for two type of message

* The int value
* The error message

And should be able to identify if it is a value or error message. Let's see how to implement it in OO

```scala
trait Result
case class IntResult(v: Int) extends Result
case class ErrorResult(error: String) extends Result

def string2Int(v:String):Result={
    try{
        IntResult(v.toInt)
    } catch {
        case e => ErrorResult(e)
    }    
}
```

Now the `string2Int` is a pure function, we can use it like this

```scala
string2Int('1')
string2Int('hello world')
```

Everything looks good for now, let's see how to use it in a more complicated case.

Say we have a txt file which have the following content

```
1897634
2345698
```

We want to read the content of this file and print the value of `first line number/second line number`.

How will we implement it using pure function(forget Monad for now)? maybe like this

```scala
trait Result
case class IntResult(v: Int) extends Result
case class StringResult(v: String) extends Result
case class ErrorResult(error: String) extends Result

def readLine(path:String,index:Int):Result = {....}

def string2Int(s:String):Result = {
    try{
        IntResult(v.toInt)
    } catch {
        case e => ErrorResult(e)
    }
}
def calcDivision(left:Int,right:Int):Result = {
    try{
        IntResult(left/right)
    } catch {
        case e => ErrorResult(e)
    }
}

def void main(args:Array[String]) = {
    val firstLine:Result = readLine("test.txt", 0)
    val secondLine:Result = readLine("test.txt", 1)
    
    val result:Result = firstLine match {
        case StringResult(l1) => {
            secondLine match {
                case StringResult(l2) =>{
                    val firstInt = string2Int(l1)
                    val secondInt = string2Int(l2)
                    firstInt match {
                        case IntResult(n1) =>{
                            secondInt match {
                                case IntResult(n2) =>{
                                    calcDivision(n1,n2)
                                }
                                case e4 => ErrorResult("Failed to parse second line to integer")
                            }
                        }
                        case e3 => ErrorResult("Failed to parse first line to integer")
                    }
                }
                case e2 => ErrorResult("Failed to read the second line")
        	}
        }
        case e1 => ErrorResult("Failed to read the first line")
    }
    
    println(result)
}
```

The code is messy,  there are so many `pattern-match` and seems they have the common pattern

```scala
result match {
    case TargetPattern(v) => doSomething(v)
    case EverythingElse => returnErrorObject
}
```

Can we make the main function more clean? Let's start from the common pattern, we found we can define functions according to this pattern like this

```scala
def processIntResult(r:Result,f:Int=>Result):Result = {
    r match {
        case IntResult(i) => f(i)
        case _ => ErrorResult(s"Can't process $r as IntResult")
    }
}
def processStringResult(r:Result,f:String=>Result):Result = {
    r match {
        case StringResult(s) => f(s)
        case _ => ErrorResult(s"Can't process $r as StringResult")
    }
}
```

Then the main function can be refined like this

```scala
def void main(args:Array[String]) = {
    val firstLine:Result = readLine("test.txt", 0)
    val secondLine:Result = readLine("test.txt", 1)
    
    val result = processStringResult(firstLine, l1 => {
        processStringResult(secondLine, l2 => {
            val firstInt = string2Int(l1)
            val secondInt = string2Int(l2)
            processIntResult(firstInt, n1 => {
                processIntResult(secondInt, n2 => {
                    calcDivision(n1,n2)
                })
            })
        })
    })
    println(result)
}
```

Hmm, looks better. but the nested level is too deep, I don't want to count the bracket. The signatures of `processIntResult` and `processStringResult` are messy, it looks more like a method of class `Result`, why don't we just put it in the body of `Result`? then we can just type `Result.processIntResult(f)` and `Result.processStringResult(f)`, this looks more clean and we can use it in chain like `Result.processIntResult(f1).processStringResult(f2)`, not like the ugly one

`processIntResult(processResult(Result,f1),f2)`.

Let's do it now

```scala
trait Result{
    def processIntResult(f:Int=>Result):Result
    def processStringResult(f:String=>Result):Result
}
case class IntResult(v: Int) extends Result{
    def processIntResult(f:Int=>Result):Result = f(v)
    def processStringResult(f:String=>Result):Result = ErrorResult("Can't process Int as String")
}
case class StringResult(v: String) extends Result{
    def processIntResult(f:Int=>Result):Result = ErrorResult("Can't process String as Int")
    def processStringResult(f:String=>Result):Result = f(v)
}
case class ErrorResult(error: String) extends Result{
    def processIntResult(f:Int=>Result):Result = this
    def processStringResult(f:String=>Result):Result = this
}
```

Then the main function can be refined like this

```scala
def void main(args:Array[String]) = {
    val firstInt:Result = readLine("test.txt", 0).processStringResult(string2Int(_))
    val secondInt:Result = readLine("test.txt", 1).processStringResult(string2Int(_))
    val result = firstInt.processIntResult(n1=> {
                secondInt.processIntResult(n2=>calcDivision(n1,n2))
            })
    println(result)
}
```

This code look more clean, but we can make it better. the following part looks ugly in this code

```scala
firstInt.processIntResult(n1=> {
     secondInt.processIntResult(n2=>calcDivision(n1,n2))
})
```

`firstInt` and `secondInt` are independent, we have to nest them here, could we have a function like this?

```
def process2IntResult(first:Result, second:Result, f:(Int, Int) => Result):Result
```

Of course we can! let's see the following implementation

```scala
def process2IntResult(first:Result, second:Result, f:(Int, Int) => Result):Result = {
    (first, second) match {
        case (IntResult(n1), IntResult(n2)) => f(n1,n2)
        case _ => ErrorResult("Failed to process two int result.")
    }
} 
```

Then the main function can be refined like this

```scala
def void main(args:Array[String]) = {
    val firstInt:Result = readLine("test.txt", 0).processStringResult(string2Int(_))
    val secondInt:Result = readLine("test.txt", 1).processStringResult(string2Int(_))
    val result = process2IntResult(firstInt,secondInt,calcDivision(_, _))
    println(result)
}
```





