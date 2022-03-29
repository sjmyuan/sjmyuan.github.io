---
title: Let's write junior code in Scala
tags:
  - Scala
  - Team
---

What make us excited when coding? Refactoring the code, involving a new technology or using a fantastic solution to resolve a hard problem?

All these things can prove our ability, but team have to maintain them, it's our responsibility to teach team.

If our team is pretty stable and all the team members are senior developers, there is nothing to concern. 

But what if it is not true? how do we ensure the team can still maintain the code very well? 

Unfortunately, this is my team's situation. My team already worked on Scala more than 5 years, we use functional programming heavily and would like to try any new technology, all team members are happy. But with more and more fantastic work we did, we found it's harder and harder to continue it, finally we decide to apply some limitation to the code to make the junior developer can contribute the code easily, we call it wring junior code.

In this post, I will share why we did this decision and how to write the junior code.

# Why Junior Code?

## Recruitment

We tried to hire people with our required the techniques, such as functional programming, cats, fs2, http4s, circe, doobie, etc. But it's harder than we thought.

In the past 5 years, we got 18 new team members, only 2 of them have Spark experience and 3 of them become senior Scala developer.

Most of the new team members are from other tech stack, Java, JavaScript or Ruby for example.

So we have to train our new team members.

## Training
Considering most of new team members don't have Scala experience, the purpose of training is  to help them to be senior Scala developer. 

Why senior Scala developer? Because we used advanced libraries, such as eff, fs2, http4s, etc. there are different architecture in different repo, such as Cake Pattern, Tageless Final, Free Monad, etc. Our team member have to master all the advanced knowledge before they can contribute qualified code.

Did I mention the training time? Ideally it should be like other languages, the team members can contribute qualified code in few weeks. Could you become a senior Java developer from graduate in few weeks? The answer is definitely no for most of people, that's the problem.

We prepared more than 20 hours sessions and workshops, which cover the knowledge from basic Scala and Functional Programming knowledge to the advanced library usage.

According to the feedback of new team members, we can split the knowledge into two parts

1. Junior knowledge, which can be mastered in few weeks.
   * Scala language
   * Functional programming fundamental knowledge
   * Simple monad, like Option, Either, IO
   * Simple function composition, like map, flatMap, filter, reduce, foldLeft
   * Library with simple interface, like cats, http4s, circe, doobie

2. Senior knowledge, which need several months to be understood.
   * Implicit
   * Free Monad
   * Complicated Monad, like Reader, State
   * Monad Transformer, like ReaderT
   * Complicated function composition, like traverse and sequence 
   * Library with complicated interface, like eff, fs2, monix
   * Architecture, like Cake pattern, ReaderT pattern and Tagless Final

Unfortunately, we used lots of Senior knowledge in our system, that mean our training can not make our new team members working on our repo shortly.

## Possible Solution

There are 3 possible solutions

1. We can bless one day there are enough people with our required Scala skills, then we don't need to worry about this problem anymore. But we know it's impossible if we do nothing.

2. We can switch to another language stack, like TypeScript or Java. But there are more than 50 existing systems writhen by Scala and most of our senior people like Scala and Functional Programming, if we switch language, we need to spend huge effort to rewrite the systems and may lose most of the senior people. This is what we can't afford.

3. Only use the Junior knowledge, then it will be easier both for new team members and existing members, we can utilize the benefit of Scala and Functional Programming, and don't need to worry about the training too much.


We chose solution 3. We know there are always complicated problem which need senior knowledge, we can move it to a framework with simple interface. ideally we can make it open source, then contribute to the Scala ecosystems.

# How to write junior code in Scala?

## Avoid implicit

`implicit` is a powerful tool, but it is very easy to be abused.

Most of time, it is `implicit` which make the code hard to read and maintain.
It’s also the biggest blocker for new team members to learn scala.

For example

```scala
import IntInstances._

val a:Int = "1"
if(a.isGreaterThan(0)){
  println(s"The number is greater than 0")
}
```

Could you understand this code? Why a Int can accept String? Where the function `isGreaterThan` come from?

How about this one

```scala
def toInt(String str):Int = a.toInt
def isGreaterThan(Int x, Int y):Boolean = x > y

val a:Int = toInt("1")
if(isGreaterThan(a, 0)){
  println(s"The number is greater than 0")
}
```

This one is not fancy, but most of developers can understand what it is doing.

`implicit` is not a required technique to write better code. 

So unless framework requires implicit instances, let's not use it.

## Unified Monad

The return type of function is very important in Functional Programming, it should be able to handle all the side effect.

If a function may return null value, we can choose Option. If a function may return error, we can choose Either. If a function may interact with API or database, we can choose IO or Task.

How about we have two functions with different return type? for example

```scala
def readInput():Option[String] = ???
def toInt(input:String):Either[String, Int] = ???
```

If we want to compose these two functions, we need unify their return type

```scala
for{
input <- readInput().toEither("There is no input")
value <- toInt(input)
} yield value
```

That mean if we have lots of different return type in our project, we need to remember how to convert them to each other. it need more knowledge but give us less benefit.

We can use an unified return type in our whole project, then all the functions can be composed directly. 

But we need to choose the return type carefully

* The return type should be powerful enough to handle all possible effect. for example, we can not choose Either if we need to interact with API or database. IO and Task are good candidate.
* The interface of return type should be easy to use. for example, Free Monad or ReaderT is complicated, we need to learn how to lift other Monad into them.

//The Unified Monad should have simple interface, we used a Monad called eff before, it's very powerful, can compose any effect together.
//But we have to replace it with IO after using it 2 years, because only few peoples can understand it properly, most of the team members can't do a qualified change to it.

## OO architecture with pure functions

We tried lots of architectures in our projects. 

At the beginning we used Cake pattern, 

```scala
trait Application with Service1 with Service2 with Service3 {
  ???

  def run = ???
} 

val application = new Application with Service1Implementation with Service2Implementation with Service3Implementation {}

application.run
```

then moved to eff which support to compose all Monad together, 

```scala
type Stack = Fx.fx4[Application, Service1, Service2, Service4, IO]

class Application[M: HasService1: HasService2: HasService3, A] {
  def runApplication[M: HasApplication, U: HasService1: HasService2: HasService3] = ???
}

val application = new Application[Stack]().runService1().runService2().runService3()

application.unsafeRunSync()
```

and after two years we embrace Tagless Final, 

```scala
trait Application[M[_]] {
  ???

  def run:M[Unit] = ???
}

class ApplicationImplementation[M[_]:Monad](service1: Service1[M], service2: Service2[M], service3: Service3[M]){
  ???

  def run:M[Unit] = ???
}

val application = new ApplicationImplementation[IO](new Service1Implementation[IO](), new Service2Implementation[IO](), new Service3Implementation[IO]())

application.run.unsafeRunSync()
```

then we come back to traditional OO architecture with pure functions. 

```scala

trait Application {
  ???

  def run:IO[Unit] = ???
}

class ApplicationImplementation(service1: Service1, service2: Service2, service3: Service3){
  ???

  def run:IO[Unit] = ???
}

val application = new ApplicationImplementation(new Service1Implementation(), new Service2Implementation(), new Service3Implementation())

application.run.unsafeRunSync()
```

Obviously, the last one is pretty easy to understand by non-Scala developer, the only thing they need to learn is how to use IO.

Actually, for eff architecture, we even spend 3 months to decommission it from all related projects. Because it need supper senior knowledge of functional programming, after some senior team members left, only few people can understand and maintain it. Although it make everything pure and easy to compose, it's still not a good time to continue to use it.

We use the traditional OO architecture to group the pure functions, there is no mutable property in it, all the data processed by function is still immutable.

Most of developers are familiar with the architecture, they just need to focus on how to make the function pure.

With this way, we can utilize the advantage of functional programming, and make our new team members happy.

## Move senior code to framework

We still need the senior code, there are always complicated problem, such as NewRellic integration, log with transaction id, global context, etc. sometimes there is no mature framework in Scala to solve them, then we need to build our own framework.

There is only one rule for the framework, the interface should be easy to use and verified. apart from that, we can write any fantastic code, even involve side effect.

For example, if we want to print log with transaction id, we may need to use MDC to store the transaction id of each thread, which is definitely side effect. But if there is no other better solution and the framework can be tested very well, we can just adopt it.

We'd better make it to be open source framework, same problem may be also happening in other teams.

With this way, we can contribute to the ecosystem and make senior team members happy.

# Summary

This is the lesson we learnt in Scala, [it also happened in some Haskell team](https://www.parsonsmatt.org/2019/12/26/write_junior_code.html).

Make our junior developer happy, it is the key point to continue our business and enlarge the community.