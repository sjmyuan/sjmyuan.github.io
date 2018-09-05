---
title: The Problem of List Concatenation
tags:
  - Scala
---

In this blog we will talk about the problem of list concatenation and give a rough introduction to Trampoline.[^6]

Before we discuss more about this topic, we need to anwser two questions

* What is Continuation Passing Style?
* What is StackOverflow?

Let's try to understand them in the following two sections.

# What is Continuation Passing Style?

>In functional programming, **continuation-passing style** (**CPS**) is a style of programming in which control is passed explicitly in the form of a continuation.[^2]

Let's see a traditional function to calculate the Pythagorean theorem.[^4]

```scala
def pythageorean(x:Double,y:Double):Double = {
    Math.sqrt(x*x+y*y)
}
```

```sh
$ pythageorean(3,4)
res1: Double = 5.0
```

Let's see the CPS of this function

```scala
def multiply(x:Double,y:Double,continuation:Double=>Double):Double = {
    continuation(x*y)
}

def sqrt(x:Double,continuation:Double=>Double):Double = {
    continuation(Math.sqrt(x))
}

def add(x:Double,y:Double,continuation:Double=>Double):Double = {
    continuation(x+y)
}

def pythageorean(x:Double,y:Double,continuation:Double=>Double):Double = {
    multiply(x,x,vx=>{
        multiply(y,y,vy=>{
            add(vx,vy,vxy=>{
                sqrt(vxy,continuation)
            })
        })
    })
}
```

```sh
$ pythageorean(3,4,x=>x)
res6: Double = 5.0
```

To write a CPS function, we only add one extra argument called **continuation** and ensure this argument will process the return value of function. In this way, we make something explicit

* Procedure returns

  We need to call continuation explicitly

* Intermediate values

  We will give them name when call continuation

* Order of argument evaluation

  We can control the order of argument evaluation by continuation

* Tail calls[^5]

  This is the main benefit we can get

  For non-recursion function, it will always be tail calls, because we need to use continuation to process return value.

  ```scala
  def add(x:Double,y:Double,continuation:Double=>Double):Double = {
      continuation(x+y)
  }
  ```

  For recursive function, we can make a tail-recursion easily

  ```scala
  def sum(n:BigDecimal):BigDecimal = {
      if(n == 0) 0
      else n+sum(n-1)
  }
  ```

  ```scala
  def sum(n:BigDecimal, continuation:BigDecimal=>BigDecimal):BigDecimal = {
      if(n==0) continuation(0)
      else sum(n-1, x=> continuation(x+n))
  }
  ```

# What is StackOverflow?

Usually, we may encounter stack overflow in two scenarios

* Non Tail-Recursion

  ```scala
  object Test {
      def isOdd(v:Int):Boolean = {
          if( v==0 ) 
              false
          else 
              isEven(v-1)
      }
      def isEven(v:Int):Boolean = {
          if(v ==0) 
              true
          else 
              isOdd(v-1)
      }
  }
  
  ```

  ```bash
  $ Test.isOdd(10000000)
  java.lang.StackOverflowError
    ammonite.$sess.cmd0$Test$.isEven(cmd0.sc:6)
    ammonite.$sess.cmd0$Test$.isOdd(cmd0.sc:3)
    ammonite.$sess.cmd0$Test$.isEven(cmd0.sc:6)
    ammonite.$sess.cmd0$Test$.isOdd(cmd0.sc:3)
  ```

* Deeply Nested Function

  ```scala
  object Test {
      def nestedFunctions(n:Int):Int => Int ={
          Range(0,n).foldLeft[Int => Int](identity)((acc,e)=>{
              x:Int => e+acc(x)
          })
      }
  }
  ```

  ```sh
  $ val f = Test.nestedFunctions(10000)
  $ f(1)
  java.lang.StackOverflowError
    ammonite.$sess.cmd3$Test$.$anonfun$nestedFunctions$3(cmd3.sc:4)
    ammonite.$sess.cmd3$Test$.$anonfun$nestedFunctions$3(cmd3.sc:4)
    ammonite.$sess.cmd3$Test$.$anonfun$nestedFunctions$3(cmd3.sc:4)
    ammonite.$sess.cmd3$Test$.$anonfun$nestedFunctions$3(cmd3.sc:4)
    ammonite.$sess.cmd3$Test$.$anonfun$nestedFunctions$3(cmd3.sc:4)
  ```

In theory, if the compiler do nothing for the recursive function, we will always get a **StackOverflowError**, because the program always need to use the stack to store the **parameter** and **local variable**, but the size of stack is fixed. 

In this diagram we can understand it clearly.[^3]

![](https://i.stack.imgur.com/SHTah.jpg)

We know Scala compiler already did some optimization for tail-recursive function. To avoid the **StackOverflowError**, we should make our recursive function to be tail-recursive and avoid deeply nested function.

# The Problem of List Concatenation

We know List is a recursive data type in Scala, we can give its definition roughly like this

```scala
sealed trait List[+A]
case object Nil extends List[Nothing]
case class ::[A](head:A, tail:List[A]) extends List[A]
```

If we want to concatenate two list, we can do it like this

```scala
def concat[A](left:List[A],right:List[A]):List[A] = {
    left match {
        case Nil => right
        case head::tail => head::concat(tail,right)
    }
}
```

But this implementation has a problem, it will throw **StackOverflowError** when the left list is too long

```sh
$ concat[Int](List.fill(100000)(0),Nil)
java.lang.StackOverflowError
  scala.collection.immutable.Nil$.equals(List.scala:433)
  ammonite.$sess.cmd31$.concat(cmd31.sc:3)
  ammonite.$sess.cmd31$.concat(cmd31.sc:4)
  ammonite.$sess.cmd31$.concat(cmd31.sc:4)
```

The call stack looks like this

![](https://ws1.sinaimg.cn/large/006tNbRwly1fuxumawf0ij30kv07t0so.jpg)

According to the previous section, we can use CPS to convert this function to tail-recursion like this

```scala
def concat[A](left:List[A],right:List[A],continuation:List[A]=>List[A]):List[A] = {
    left match {
        case Nil => continuation(right)
        case head::tail => concat(tail,right, x=> continuation(head::x))
    }
}
```

But this tail-recusion still throw a **StackOverflowError**

```sh
$ concat[Int](List.fill(100000)(0),Nil,identity)
java.lang.StackOverflowError
  ammonite.$sess.cmd34$.$anonfun$concat$1(cmd34.sc:4)
  ammonite.$sess.cmd34$.$anonfun$concat$1(cmd34.sc:4)
  ammonite.$sess.cmd34$.$anonfun$concat$1(cmd34.sc:4)
  ammonite.$sess.cmd34$.$anonfun$concat$1(cmd34.sc:4)
```

Why this happen? let's see what's the call stack of this function

![](https://ws4.sinaimg.cn/large/006tNbRwly1fuxv6pa2h7j30kv07tmx6.jpg)

On the left side, we optimize the tail-recursion and it won't throw the StackOverflowError, but we found the tail-recursive function compose the continuation function in every call and we got a deeply nested function in the final call.

When we begin to evaluation the continuation function, we actually invoke a deeply nested function. According to the previous section, it will throw the StackOverflowError.

How to avoid the deeply nested function? or How to convert the nested function to tail-recursion?

# Nested function to Tail-recursion

Let's say we have three functions

```scala
def f1(x:Int):Int = x+1
def f2(x:Int):Int = f1(x)+2
def f3(x:Int):Int = f2(x)+3
```

How do we change **f3** to a tail-recursive function? To do that, we can use a data type to store the function information and loop the data type using tail-recursion

```scala
sealed trait Action[A]
case class Done[A](v:A) extends Action[A]
case class Doing[A](x:Action[A], f:A=>Action[A]) extends Action[A]

def evaluate[A](x:Action[A]):A = {
    x match {
        case Done(v) => v
        case Doing(v,f) => v match {
            case Done(v1) => evaluate(f(v1))
            case Doing(v1,f1) => evaluate(Doing[A](v1,y=>Doing(f1(y),f)))
        }
    }
}
```

```scala
def f1M(x:Int):Action[Int] = Doing[Int](Done(x),v=>Done(v+1))
def f2M(x:Int):Action[Int] = Doing[Int](f1M(x),v=>Done(v+2))
def f3M(x:Int):Action[Int] = Doing[Int](f2M(x),v=>Done(v+3))
```

```sh
$ evaluate(f3M(100))
res46: Int = 106
```

The basic idea here is we use **Action** to store the call stack and it will break the call chains, then re-evaluate them one by one in an outer loop.

The knowledge of **Action** is more than what we can see in above code, it use the technique called **Trampoline**,[^1][^6] we will talk about it more detail in another blog.

# Reference

[^1]: [Stackless Scala with Free Monads](http://blog.higher-order.com/assets/trampolines.pdf)
[^2]: [Continuation passing style](https://en.wikipedia.org/wiki/Continuation-passing_style)
[^3]: [What is a StackOverflow](https://stackoverflow.com/questions/214741/what-is-a-stackoverflowerror)
[^4]: [Pythagorean theorem](https://en.wikipedia.org/wiki/Pythagorean_theorem)
[^5]: [Tail Calls](https://en.wikipedia.org/wiki/Tail_call)
[^6]: Steven E. Ganz and Daniel P. Friedman and Mitchell Wand, Trampolined Style, in International Conference on Functional Programming, ACM Press, 1999, pp. 18–27.

