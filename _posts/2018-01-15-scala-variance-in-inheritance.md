---
layout: post
title: Scala Variance in Inheritance
excerpt: ""
tags: Scala
---
# Contents
{:.no_toc}

* Toc
{:toc}

# What is variance in Scala?

The variance defines the inheritance relationships of parameterized types. Scala has there types of variance

* Covariant

  Given we have a parameterized type T[+A]

  When type C is subtype  of type P

  Then T[C] is subtype of T[P]

* Contravariant

  Given we have a parameterized type T[-A]

  When type C is subtype of type P

  Then T[P] is subtype of T[C]

* Invariant

  Given we have a parameterized type T[A]

  When type C is subtype of type P

  Then T[P] and T[C] are unrelated

# The scenario I want to talk about

I want to talk about the behaviour of variance in inheritance of parameterized types.

Let's say we have the following types

~~~scala
trait Parent[+T]
case class Child1[T](v:T) extends Parent[T]
case class Child2[+T](v:T) extends Parent[T]
~~~

Is there any different between Child1 and Child2?

Let's do some experiments

* Assign Child[Int] to Parent[Any]

  * without type parameter

    ~~~ bash
    @ val parent1:Parent[Any] = Child1(1)
    parent1: Parent[Any] = Child1(1)

    @ val parent2:Parent[Any] = Child2(1)
    parent2: Parent[Any] = Child2(1)
    ~~~

  * with type parameter

    ~~~ bash
    @ val parent1:Parent[Any] = Child1[Int](1)
    parent1: Parent[Any] = Child1(1)

    @ val parent2:Parent[Any] = Child2[Int](1)
    parent2: Parent[Any] = Child2(1)
    ~~~

  We can see they all follow the covariant rule, the difference between Child1 and Child2 has no impact to Parent

* Assign Child[Int] to Child[Any]

  * without type parameter

    ~~~ bash
    @ val child1:Child1[Any] = Child1(1)
    child1: Child1[Any] = Child1(1)

    @ val child2:Child2[Any] = Child2(1)
    child2: Child2[Any] = Child2(1)
    ~~~

    There is no different behaviour

  * with type parameter

    ~~~ bash
    @ val child1:Child1[Any] = Child1[Int](1)
    cmd14.sc:1: type mismatch;
     found   : ammonite.$sess.cmd1.Child1[Int]
     required: ammonite.$sess.cmd1.Child1[Any]
    Note: Int <: Any, but class Child1 is invariant in type T.
    You may wish to define T as +T instead. (SLS 4.5)
    val child1:Child1[Any] = Child1[Int](1)
                                        ^
    Compilation Failed

    @ val child2:Child2[Any] = Child2[Int](1)
    child2: Child2[Any] = Child2(1)
    ~~~

    Child1 follow the invariant rule, Child2 follow the covariant rule, just like their definition

You may ask what about this

~~~ scala
case class Child3[-T](v:T) extends Parent[T]
~~~

Unfortunately you can't define this type, there will be an error

~~~ bash
contravariant type T occurs in covariant position in type [-T]AnyRef
~~~

# Conclusion

According to the above experiments, we can get a conclusion: 

The variance of type parameter has no relation with the inheritance of parameterized type, it was just decided by the current type
