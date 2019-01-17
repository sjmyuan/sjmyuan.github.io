---
title: a-special-tail-recursion
tags:
  - Scala
---

In this blog, we will talk about the **evaluate** function mentioned in last blog [The Problem of List Concatenation](https://blog.shangjiaming.com/the-problem-of-list-concatenation/). 

Let's review the definition of this function

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

This is a tail-recursive function, we should be safe to use it, right?

Let's see the follwoing example

```scala
def buildStack:Action[Int] = Range(0,10000).foldLeft(Done(0))((acc,e)=>{
    Doing(acc,x=>Done(x+e))
})

```



```scala
f(v1) -> f1(y) -> f2(y) -> f3(y) .... fn(y)
Done(v1,f1)
f1 -> Doing(f2(v2),_)
f2 -> Doing(f3(v3),_)
f3 -> Doing(f4(v4),_)
f4 -> Done(1)

Doing(Doing(v1,f1),f2) -> Doing(v1,Doing(f1,f2))

```

