---
title: Say Goodbye to Eff
tags:
  - fp
---

# What

About 2 years ago, we involve a library called [Eff](https://github.com/atnos-org/eff) in our system. 
It's something like Free Monad but support unlimited effect.

With this library, we can compose arbitrary effects together.
Except the common monads, it's also very easy to create custom ADT.

Actually we created more than 20 ADTs and used them to seperate our business logic and io implementation.

Ideally with the seperation of logic and implementation, it should be easy to test,
but there is no mature framework support eff, we have to develop our own test framework called eff-test-util.

It give us lots of flexibility, we even didn't think about the intra-architecture carefully, we thought it was just function composition and we can do it by eff easily.

But the problem is it's very hard to understand why it can work, it need lots of fp knowledge.

We have a good team who can overcome the sharp learning curve and deliver product efficiently.

But we can't avoid the movement of people, the knowledge is also losing with people moving.

At last, most of team members can implement the feature correctly, but they don't know why which make people upset.

Team feel more and more painful on Eff and finally we decide to decommission it.

To be honest, I feel sad about this decision, this is the most complicated library I learnt but also push me dig into functional programming.
But proofs by fact, it's not a correct time to use this library. 
We need to attract people to use Scala first, or Scala will die, not to mention this library. 

# Why

## Learning curve

Eff is not friendly to newbies, they need to learn Scala basic, FP basic and Common Monads first, then can understand Eff.
This process usually take 3 or even more months and hard to get the positive feedback, which is long enough to fade the passing of learning FP.

The usage of Eff spend most of the energy of team, we can't focus on the intra-process architecture designing, which make our code worse and worse.

There is no mature test framework for Eff, we have to build a totally new one by ourself, which always scare newbies to add a new test.

At last, most of team member can deliver product on this code base carefully, but don't understand the code they are writing.

## People Hiring

We can't hire people with Scala skills, not to mention people with Scala FP skills. 

We have to train people, but the learning curve stop us from doing it efficiently.

We have to prove the benefit of Scala again and again to manager, which make us tired.

So maybe we should do something differently, something like [Write Junior Code](https://www.parsonsmatt.org/2019/12/26/write_junior_code.html), show people how easy to use Scala and FP, abandon the library like Eff.

## Project Maintainability

As mentioned before, we don't have time to think about the intra-process architecture.

Team always think how to implement this by Eff, not how should we design this part better.

So we even didn't use the layered architecture in our repo, just compose more than 30 effects by Eff.

You may say this is functional programming, compose the function! 
but we still need a strategy to organize the source code, we still need to isolate the domain logic.
for big project, we can't simple implement our logic by 2 or 3 functions, we need hundreds or even thousands of them,
how can you manage them easily without any strategy?

This is big problem of functional programming, 
we don't have a set of common and mature patterns to follow easily, 
different project have different pattern, different people use different pattern. It's a disaster for big group.

If we don't change now, the project will be harder and harder to add new feature.

# How

We need to think about Scala and functional programming from junior perspective, make the code simple, avoid fancy feature.

We think Tagless Final is a proper pattern for junior developer, they just need to know the normal OO pattern and familiar with the usage of Monad and Sync, which is pretty easy to learn.

We should also give the solution of common problem, such as log, transaction, configuration, http client.

They exist in almost all the projects, but hard to be implemented in functional programming.

The nature of these problem is dependency injection, We can use [ReaderT pattern](https://www.fpcomplete.com/blog/2017/06/readert-design-pattern/), but we are scared of it, Reader and ReaderT need more effort to learn.

We choose the most simple solution, use construct parameter to pass it everywhere. you may say it ugly, but I will say it's easy, every developer can understand it.

Except the requirement of library, we discourage implicit which reduce the readability and maintainability of code.

# Summary

This is a blog about decommissioning Eff, but I talk lots of Scala.

I think Scala are experiencing the same situation of HaskKell at least around me, I love it but I can't force people to spend lots of time to learn it, because I can't give them job.

I totally agree with [All Languages lead to Scala](https://www.lihaoyi.com/post/FromFirstPrinciplesWhyScala.html#conclusion-all-languages-lead-to-scala), it's too powerful, what we need to do is to restrict the fancy features carefully. If any developer can contribute the code in two weeks, then we can involve more people to use it.
