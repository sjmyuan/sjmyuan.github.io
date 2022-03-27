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

There are too many things to learn for a Java developer. we can't make a junior Scala developer to be Senior Scala developer quickly.  

Like other new languages, it's always easy to master the basic knowledge, such as Scala primitive feature and simple Monad(Option/Either/Try).

But it's hard to master the advanced Functional programming concept and library, such as Free Monad, Monad Transformer.

The purpose of the training is to make a Java developer meet our requirements of Scala shortly, considering the new technologies we using in project, thats mean they need to be a Senior Scala shortly.

We prepared sessions and workshops, but it doesn't work, our team members still need at least 3 months to do qualified contribution.
Some techniques even need more time, such as Free Monad, Monad Transformer, fs2 and eff.

During the training, some senior members will roll off, less and less members master some technologies, such as eff, we have to plan some story to decommission this library.

If we can train newbies efficiently, the knowledge should not be lost and we don't need to spend extra effort to decommission some fantastic technology.

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
