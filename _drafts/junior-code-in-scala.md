---
title: Write Junior Code in Scala
tags:
  - Scala
  - Team
---

What make us excited when coding? Refactoring the code, involving a new technology or using a fantastic solution to resolve a hard problem?

These things are harder, and can prove our ability. But we should not forget all team members will maintain them, not only us.

If our team is pretty stable and all the team members are senior developers, there is nothing to concern. 

But what if it is not true? how do we ensure the team can still maintain the code very well? 

Unfortunately, this is my team's situation. My team already worked on Scala more than 5 years, we use Functional Programming in Scala heavily and would like to try any new technologies. But with more and more fantastic work we did, we found it's harder and harder to continue the business, finally we decide to write junior code in Scala.

In this post, I will share the reason for the decision and show you how to write junior code in Scala.

# Why Junior Code?

## Recruitment

We tried to hire people with our required techniques, such as Functional Programming in Scala, [Cats](https://typelevel.org/cats/), [Fs2](https://fs2.io/#/), [http4s](https://http4s.org/), [circe](https://circe.github.io/circe/), [doobie](https://tpolecat.github.io/doobie/), [eff](https://atnos-org.github.io/eff/), etc. But it's harder than we thought.

In the past 5 years, we got 18 new team members, only 2 of them have Spark experience and 3 of them become senior Scala developers.

Most of the new team members are from other tech stack, for example, Java, JavaScript or Ruby.

So we have to train our new team members.

## Training

Considering most of new team members don't have Scala experience, the purpose of training is to help them to be senior Scala developer. 

Why senior Scala developer? Because we used lots of advanced libraries, such as eff, Fs2, etc, and there are different architectures in different repos, such as [Cake pattern](https://medium.com/rahasak/scala-cake-pattern-e0cd894dae4e), [ReaderT pattern](https://www.fpcomplete.com/blog/2017/06/readert-design-pattern/), [Tagless Final](https://okmij.org/ftp/tagless-final/), etc. Our team members have to master all the advanced knowledge to maintain the existing systems.

Did I mention the training time? Ideally it should only take few weeks to train new team members, then they can contribute qualified code. But could you become a senior Java developer from graduate in few weeks? The answer is definitely no, no matter how good is the training, The brain need time to understand the knowledge, that's the problem.

We prepared more than 20 hours sessions and workshops, which cover the knowledge from Functional Programming in Scala to the advanced library usage.

According to the feedback of new team members, we can divide the knowledge into two parts

1. Junior knowledge, which can be mastered in few weeks.

   * Scala language
   * Functional Programming fundamental concept
   * Simple Monad, like Option, Either, IO
   * Simple high-order function, like map, flatMap, filter, reduce, foldLeft
   * Library with simple interface, like Cats, http4s, circe, doobie

2. Senior knowledge, which need several months to understand.

   * Implicit
   * Free Monad
   * Complicated Monad, like Reader, State
   * Monad Transformer, like ReaderT, WriterT, StateT
   * Complicated high-order function, like traverse and sequence 
   * Library with complicated interface, like eff, Fs2
   * Architecture, like Cake pattern, ReaderT pattern, Tagless Final

We used lots of Senior knowledge in our systems, that means our training can not help our new team members to understand and maintain the existing systems quickly.

## Possible Solutions

There are 3 possible solutions

1. We can bless one day there are enough developers with our required skills, then we don't need to worry about the business anymore. But we know it's impossible if we do nothing.

2. We can switch to other languages, like TypeScript or Java. But there are more than 50 existing systems written by Scala and most of our senior developer like Functional Programming in Scala. If we switch language, we need to spend huge effort to rewrite the systems and most of the senior developer may run. This is what we can't afford.

3. Write junior code, it only need the Junior knowledge, then it will be easier for both new team members and existing members, we can not only utilize the benefit of Functional Programming in Scala, but also save the effort of training.


Option 3 is a good balance between technique excellence and business continuation.

# How to write junior code in Scala?

## Avoid implicit

`implicit` is a powerful tool, but it is very easy to be abused.

Most of time, it is `implicit` which make the code hard to read and maintain.
It’s also the biggest blocker for new team members to learn Scala.

For example

```scala
import IntInstances._

val a:Int = "1"
if(a.isGreaterThan(0)){
  println("The number is greater than 0")
}
```

Could you understand this code? Why an Int variable can accept a String value? Where does the function `isGreaterThan` come from?

How about this one

```scala
def toInt(String str):Int = a.toInt
def isGreaterThan(Int x, Int y):Boolean = x > y

val a:Int = toInt("1")
if(isGreaterThan(a, 0)){
  println("The number is greater than 0")
}
```

This one is not fancy, but most of developers can understand it.

`implicit` is not a required technique to write better code. 
Unless library requires implicit instances or we're pretty sure what we are doing, let's not use it.

## Unified Monadic Return Type

The monadic return type of function is very important in Functional Programming, it should be able to handle all the [side-effects](https://en.wikipedia.org/wiki/Side_effect_(computer_science)).

For example, if a function may return null value, we can return Option. If a function may return error, we can return Either. If a function may interact with API or database, we can return IO or Task.

What if we have two functions with different monadic return type? for example

```scala
def last(list: List[String]):Option[String] = ???
def toInt(x:String):Either[String, Int] = ???
```

If we want to compose these two functions, we need to convert Option to Either 

```scala
for{
  x <- last(List("1")).toRight("The list is empty")
  value <- toInt(x)
} yield value
```

~~That means if we have lots of different monadic return type in our project, we need to remember how to convert them to each other. it need more knowledge but give us less benefit.~~

To avoid the conversion among different monadic return types, we can use an unified monadic return type in the whole project, then all the functions can be composed directly. 

But we need to choose the monadic return type carefully

1. It should be powerful enough to handle all potential side-effects. for example, we can not use Either if we need to interact with API or database. IO and Task are good candidates.
2. Its interface should be easy to use. for example, Free Monad and ReaderT are too complicated, we need more time to understand them.

## OO architecture with pure functions

We tried lots of architectures in our projects. 

At the beginning we used Cake pattern, 

```scala
trait Application  { self: Service1 with Service2 with Service3 => 
  def run = ???
} 

val application = new Application with Service1Implementation with Service2Implementation with Service3Implementation {}

application.run
```

then moved to eff which support extensible effects, 

```scala

trait ApplicationEffect[A]

case object Start extends ApplicationEffect[Nothing]

type Stack = Fx.fx4[ApplicationEffect, Service1Effect, Service2Effect, Service3Effect, IO]

implicit class Service1InterpretationOps[R, A](eff: Eff[R, A]) {
  def runService1[U: Member.Aux[Service1Effect, R, ?]: HasIO, A](): Eff[U, A] = ???
}

implicit class Service2InterpretationOps[R, A](eff: Eff[R, A]) {
  def runService2[U: Member.Aux[Service2Effect, R, ?]: HasIO, A](): Eff[U, A] = ???
}

implicit class Service3InterpretationOps[R, A](eff: Eff[R, A]) {
  def runService3[U: Member.Aux[Service3Effect, R, ?]: HasIO, A](): Eff[U, A] = ???
}

implicit class ApplicationInterpretationOps[R, A](eff: Eff[R, A]) {
  def runApplication[U: Member.Aux[ApplicationEffect, R, ?]: HasService1Effect: HasService2Effect: HasService3Effect, A](): Eff[U, A] = ???
}

val application = Eff.send[Stack](Start).runApplication().runService1().runService2().runService3()

application.unsafeRunSync()
```

and after two years we embrace Tagless Final, 

```scala
trait Application[M[_]] {
  def run:M[Unit] = ???
}

class ApplicationImplementation[M[_]:Monad](service1: Service1[M], service2: Service2[M], service3: Service3[M]){
  def run:M[Unit] = ???
}

val application = new ApplicationImplementation[IO](new Service1Implementation[IO](), new Service2Implementation[IO](), new Service3Implementation[IO]())

application.run.unsafeRunSync()
```

then we come back to traditional OO architecture with pure functions. 

```scala

trait Application {
  def run:IO[Unit] = ???
}

class ApplicationImplementation(service1: Service1, service2: Service2, service3: Service3){
  def run:IO[Unit] = ???
}

val application = new ApplicationImplementation(new Service1Implementation(), new Service2Implementation(), new Service3Implementation())

application.run.unsafeRunSync()
```

Obviously, the last one is pretty easy to understand for non-Scala developer, the only thing they need to learn is how to use IO.

For eff architecture, we even spend 3 months to decommission it from all related projects, because it's hard to understand, and after some senior team members left, only few people can understand and maintain it.

Most of developers are familiar with OO architectures, they just need to focus on how to make the function pure.
And we just use the OO architecture to group the pure functions, there is no mutable property in it, all the data are immutable.

With this way, we can not only utilize the advantage of Functional Programming, but also make our new team members happy.

## Move senior code to library

We still need the senior code, because there are always complicated problems, such as NewRellic integration, log with transaction id, global context, etc. Sometimes there is no mature library in Scala to solve them, then we need to build our own library.

There is only one rule for the library, the interface should be easy to use. Apart from that, we can do what we want to do, even involve side-effects.

For example, if we want to print log with transaction id, we may need to use MDC to store the transaction id of each thread, which is definitely side-effect. But if there is no other better solution and the library can be tested very well, we can just adopt the side-effect.

We'd better open source the library, same problem may be also happening in other teams.

With this way, we can contribute to the ecosystem and make our senior team members happy.

# Summary

This is the lesson we learnt in Scala, [it also happened in some Haskell team](https://www.parsonsmatt.org/2019/12/26/write_junior_code.html).
We always need to balance business and the technique excellence to achieve success as one team. 