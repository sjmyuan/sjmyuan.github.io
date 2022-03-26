---
title: Let's write junior code in Scala
tags:
  - Scala
---

Our team have been using Scala about 5 years, there some lesson we learnt, the most important one is writing junior code. 

TL,DR

* We got problem hiring people with Scala skills. 
* We can't train people efficiently
* It's hard to switch tech stack
* Writing junior code can save us

# Why Junior Code?

## People

It's always hard to hire people with Scala skills, our requirements are

* Scala
* Functional programming
* Cats ecosystem

But the best people we can hire is people with Spark Scala, which still have a big gap with our requirements.

Usually we have to hire Java developers and train them by ourselves, but it's still a challenge.

## Training

We can't train people efficiently, there too many things to learn.

It's easy to teach Scala basic knowledge and common Monad like Option, Either. but people will always stuck at Free Monad, Monad Transformer and implicit.

It usually need more than 3 month to train people to make them contribute qualified code.

And most of time, before the newbies can contribute code, some senior team members will roll off.

## Possible Solution

We can bless there are enough people with Scala skills and we don't need to worry about it anymore, but it's impossible for now.

We also can switch to another language stack, like TypeScript or Java, but there are more than 50 systems writhen by Scala and most of our senior people like Scala, if we switch language, we need to rewrite the systems and lose most of the senior people.

So the possible solution is starting write junior code to make sure it

* Friendly for Java Developer.
* Easy to train, for example, after several weeks training, A Java Developer can contribute qualified code.
* Enlarge the Scala community in long term.

Also there are always complicated problem which need senior people to write fancy code, we can move it to a framework with simple interface. ideally we can make it open source, then contribute to the Scala ecosystems.

# How to write junior code in Scala?

## Avoid implicit

According to our experience, implicit is the most hard part for beginner.

Although it can make our code clean and simple, but we need more time to understand how it works.

It's not a required technique to write better code. 

So unless supply implicit instance to framework, let's not define implicit function/instance/class by ourself, let's use the raw code to implement the feature.

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
