---
title: Say Goodbye to Eff
tags:
- fp
excerpt: ''
date: 2021-07-15 22:43 +0800
---
# What

About 3 years ago, we introduced [Eff](https://github.com/atnos-org/eff) in our system.
It's something like Free Monad but supports multiple effects composing.

With this library, we can compose any number of effects together.
Except for the common monads such as Option, Either, Reader, IO, etc, it's also very easy to create custom ADT.
Actually, we created more than 30 ADTs and used them to separate our business logic and implementation.

Ideally, with the separation of logic and implementation, it should be easy to test.
But there is no mature test framework which supports eff, we have to develop our test framework called eff-test-util which support mock effects and check the calling order.

Eff give us lots of flexibility, we even don't need to think about the intra-architecture like OO,
everything can be effect and we can inject it easily.

But the problem is it's very hard to understand why it can work, which needs lots of FP knowledge.

We have a super team who can overcome the sharp learning curve and deliver products efficiently.
But we can't avoid the movement of people, and the knowledge is also losing with people moving.

At last, most team members can implement the feature correctly, but they don't know why it works, which makes people upset.

Now, team members feel more and more upset and painful about Eff and finally, we decide to decommission it.

To be honest, I feel sad about this decision. This is the most complicated library I learned but also push me to dig into functional programming.
But proof by facts, it's not a proper time to use this library now.

We need to attract people to use Scala first, or Scala may die, not to mention this library.

# Why

## Learning curve

Eff is not friendly to newbies, they need to learn basic Scala knowledge, basic FP knowledge, common Monads, and Monad Transformer first, then they may understand it.
This process usually takes 3 or even more months, which is long enough to fade the passion of learning FP.

The training of Eff spent most of the energy of the team, which prevent us from learning other new things and making our project better.

For example, we don't have time to review the effect composing carefully and introduce mature FP intra-process architecture,
which makes the number of effects explosive and the dependency graph messy(Yeah, we still need to care about dependency in FP).

Our own Eff test framework does not work as well as other frameworks. there is some special syntax, which always scares newbies to add a new test.

We used lots of implicit in our code to make the syntax better, but the readability is low, team members have to master implicit to understand the code.

## People Hiring

We can't hire people with Scala skills, not to mention people with Scala FP skills.

We have to train people, but the sharp learning curve stops us from doing it efficiently.

We have to prove the benefit of Scala to the manager again and again, which makes us tired.

So maybe we should do something differently,
something like [Write Junior Code](https://www.parsonsmatt.org/2019/12/26/write_junior_code.html),
show people how easy to use Scala and FP, abandon the complex library and syntax, give readability the priority.

## Project Maintainability

As mentioned before, we don't have time to think about other things except for Eff.
We developed the test framework, effects to manage http4s,
effects to support NewRelic, effect to support transaction, effect to support log, etc.
New heavy effect always introduces new things to learn.

Proof by facts, FP also needs intra-process architecture, not just function composing.
Now we have more than 30 effects, how do we name them? how do we structure the source code?
how do we isolate the changes and keep our core function stable?

If we still don't have time to think about them, the project will be harder and harder to maintain.

Let's leave Eff and fancy features to interest, keep our project code simple and easy to maintain.

Let's challenge every new feature we want to introduce in our project, can it be mastered in 2 weeks by newbies?
can it be accepted by all team members? did we already add enough new features in this project?

# How

First, we need a replacement of Eff

We need to think about Scala and functional programming from a junior perspective, make the code simple, avoid fancy features.

[Tagless Final](https://blog.shangjiaming.com/scala%20tutorial/tagless-final/) is a proper pattern for a junior developer.
they just need to know the normal OO pattern and be familiar with the usage of Monad and Sync, which is pretty easy to learn.

For dependency injection,
we can use [ReaderT pattern](https://www.fpcomplete.com/blog/2017/06/readert-design-pattern/),
but we are scared of it, Reader and ReaderT need more effort to learn.

We choose the most simple solution, use constructor to pass dependencies everywhere.
you may say it's ugly, but we will say it's easy, every developer can understand it immediately.

Second, we need to give some boring code to allow the team to resolve some common problems by copy-paste.

We use Tagless Final to implement log, transaction, configuration, http client, NewRelic, etc. The team just needs to copy them if they need them in other projects.

We didn't extract them to a separated library, because we did it before for Eff but it's painful to upgrade the dependencies.
We will do it until we think they are stable and the dependency upgrade problem is resolved.

Third, we need to define some rules for Eff decommissioning

Except for the requirement of the library, we discourage implicit which reduces the readability and maintainability of code.

No code improvement. these are critical systems, we need to make sure the Eff decommissioning is simple and easy to find the difference, refactoring can be done in the future.

After decommissioning one effect, we need to deploy production as soon as possible. we don't want a big release with big risk, 
instead, we release it by effect, which can give us feedback quickly and control the risk. 

# Summary

I think Scala is experiencing the same situation as Haskell at least around me.
I love it but I can't force people to spend lots of time learning it, because I can't give them the job.

I agree with [All Languages lead to Scala](https://www.lihaoyi.com/post/FromFirstPrinciplesWhyScala.html#conclusion-all-languages-lead-to-scala),
Scala is a qualified language, I don't worry about its future. but it's too powerful, we need to restrict the fancy features carefully.
If any developer can contribute the code in two weeks, then we can attract more people to use it, and the community can be bigger and bigger.
