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

There are lots of Monad can catch the side effect, such as Option, Either, Try, IO, Task etc. We may use different Monad in different service, that means we need to convert them to each other, it need more knowledge to do that.

We can use an unified Monad in our whole project to simplify the convert process, for example, we can use IO to catch all the side effect, then we just need to care about how to catch the side effect, don't need to care how to compose the current service with other service.

The Unified Monad should have simple interface, we used a Monad called eff before, it's very powerful, can compose any effect together.
But we have to replace it with IO after using it 2 years, because only few peoples can understand it properly, most of the team members can't do a qualified change to it.

## OO architecture with pure function

We can still use functional programming to utilize its advantage, but we need a way to group our functions properly.

The traditional OO architectures like three-layers, port-adapter and onion are good options, most of developers are familiar with them, they just need to focus on how to make the function pure.

Not like Java, our service will only have functions, we will use ADT to define the immutable data which is a required tech to make function pure.

## Move senior code to framework

There are always complicated problem, such as NewRellic, transaction id, global context. there is no mature framework in Scala ecosystem to solve them easily.

Senior people need to solve them to unblock team, we can create an open source framework to do that. There is no limitation in the framework even side effect, only one rule is the interface should be easy.

Then we can contribute to the ecosystem and make senior people happy.

# Summary

We already took the actions, it reduce the concern of HR, DM and Team Lead, they just need to hire Java Developer.

Team members also feel more happy, because the feedback loop is shorter now, they can do a better contribution quickly.

We can utilize most of the techniques in OO to refine our architecture and code, not only pure function and fancy code.

To get a better community, let's write junior code.
