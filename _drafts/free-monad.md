---
title: Free Monad
tags:
  - Scala
categories:
  - Scala Tutorial
---

When we talk about Free Monad, almost everyone just tell us it can split the definition and implementation of program, and easy to do test.
But this is just the resut of using Free Monad, no one tell us why we need it and how it was created.

In this blog, we will try to find out the motivation of Free Monad from code level and infer it by ourself.

# Question

Let's begin with a simple question, how do you make this function pure?

```scala
def copy(from:String, to:String):Unit = {
  val content = Source.fromFile(from).mkString
  val file = new File(to)
  val bw = new BufferedWriter(new FileWriter(file))
  bw.write(content)
  bw.close()
}
```

In [Algebraic Data Type](https://blog.shangjiaming.com/scala%20tutorial/algebraic-data-type/), we talk about how to make a function pure, let's try it on the `copy` function.

The purpose of this function is to copy one file to another location. 
According to [What is Functional Programming?](https://blog.shangjiaming.com/scala%20tutorial/what-is-fp/),There are 2 effect in this function

* Read file
* Write file

So we can define an ADT like this

```
sealed trait FileOps[A]
case class ReadFile(path:String) extends FileOps[String]
case class SaveFile(path:String, content:String) extends FileOps[Boolean]
```

Then the `copy` can be refined to be pure by the ADT

```scala
def copy(from:String, to:String):FileOps[Boolean] = {
  ReadFile(from)
  SaveFile(to, ???)
}
```
There are two problems here:

* We don't know the content of `SaveFile`.

  In the non-pure `copy`, we got the content from the return value of `Source.fromFile`, but this is a function with `Side-Effect`, we replace it with `ReadFile` effect.
  The `ReadFile` effect is just a message to the outside of function and won't do the real io work, so we can't get the file content from it.

* The `ReadFile` effect is ignored by the return expression.

  Not like the function in [Algebraic Data Type](https://blog.shangjiaming.com/scala%20tutorial/algebraic-data-type/) which just return A effect or B effect and won't return them together.
  For pure `copy` function, we expect the return type contains two effects: `ReadFile` and `SaveFile`, not only the `SaveFile`
  And these two effects should happens in order, we can't imagine a copy operation do save operation first, it can't save anything before read the content.

According to these problems, we know our ADT can't make the `copy` function pure, we need the ADT not only represent multiple effects, but also the order of effects. 

# Solution

Obviously the simplest way to represent ordered elements is to use Array like model

```scala
case class OrderedEffect[A, B](effect1:FileOps[A], effect2:FileOps[B]) extends FileOps[B]
```

But `effect1` and `effect2` are not independent, `effect2` may use the return value of `effect1`.
To represent the dependent relationship, we can define `effect2` always depends on `effect1`, then we can use a call back function to represent it.

```scala
case class OrderedEffect[A, B](effect1:FileOps[A], effect2Callback: A => FileOps[B]) extends FileOps[B]
```

This effect have two meanings:

* The evaluated value of this effect is B which can be a primitive data or the return value of non-pure operations represented by this effect.

* When we evaluate this effect, we need to evaluate `effect1` first, then pass the return value to `effect2Callback` function, then evaluate the returned `effect2`

Here we use `evaluate effect` to mean run the non-pure operations represented by the effect. 

Then our ADT becomes this

```
sealed trait FileOps[A]
case class ReadFile(path:String) extends FileOps[String]
case class SaveFile(path:String, content:String) extends FileOps[Boolean]
case class OrderedEffect[A, B](effect1:FileOps[A], effect2Callback: A => FileOps[B]) extends FileOps[B]
```

The `copy` function can be made pure like this

```scala
def copy(from:String, to:String):FileOps[Boolean] = {
  OrderedEffect[String, Boolean](ReadFile(from), (content:String) => SaveFile(to, content))
}
```

Looks all good for now, but according to [Monad](https://blog.shangjiaming.com/scala%20tutorial/scala-monad/), our ADT need `map` and `flatMap` to remove duplication.

Let's try

```scala
def flatMap[A, B](fa: FileOps[A])(f: A => FileOps[B]): FileOps[B] = fa match {
  case ReadFile(path) => 
    val content = Source.fromFile(path).mkString
    f(content)
  case SaveFile(path, content) => try {
    val file = new File(path)
    val bw = new BufferedWriter(new FileWriter(file))
    bw.write(content)
    bw.close()
    f(true)
  } catch {
    case e => 
      f(false)
  }
  case OrderedEffect(effect1, f1) =>
    flatMap(flatMap(effect1)(f1))(f)
}
```

```scala
def map[A, B](fa: FileOps[A])(f: A => B): FileOps[B] = fa match {
  case ReadFile(path) => 
    val content = Source.fromFile(path).mkString
    f(content)
    ???
  case SaveFile(path, content) => try {
    val file = new File(path)
    val bw = new BufferedWriter(new FileWriter(file))
    bw.write(content)
    bw.close()
    f(true)
    ???
  } catch {
    case e => 
      f(false)
      ???
  }
  case OrderedEffect(effect1, f1) => 
    OrderedEffect(effect1, (x:A) => map(f1(x))(f))
}
```

Except for `Side-Effect`, `flatMap` looks good, we can give a definite implementation. 

We got a problem in `map`, we don't know which effect should be returned when processing `ReadFile` and `SaveFile`.

* For `ReadFile`,
  * Could we return `ReadFile` again? No, we can't read a file twice, it will fall in the dead loop, we will read file for ever.
  * Could we return `SaveFile`? No, we don't know where to save and we even don't what's the real type of `B`.

* For `SaveFile`
  * Could we return `ReadFile`? No, which file should we read?
  * Could we return `SaveFile` again? No, we can't save a file twice, it will fall in the dead loop, we will save a file for ever.

Seems the only solution is to return `OrderedEffect`, but `f: A => B` is a pure function, can't pass to parameter with type `f: A => FileOps[B]`.
If we can convert `B` to `FileOps[B]`, it may work.

Let's add an effect like `Some` to our ADT

```
case class NoEffect[A](v: A) extends FileOps[A]
```

Then we can implement `map` like this

```scala
def map[A, B](fa: FileOps[A])(f: A => B): FileOps[B] = fa match {
  case ReadFile(path) => 
    OrderedEffect(fa, (x:A) => NoEffect(f(x)))
  case SaveFile(path, content) =>
    OrderedEffect(fa, (x:A) => NoEffect(f(x)))
  case NoEffect(v) => NoEffect(f(v))
  case OrderedEffect(effect1, f1) => 
    OrderedEffect(effect1, (x:Any) => map(f1(x))(f))
}
```
Ok, now `map` is implemented and pure. let's go back and see how to remove the `Side-Effect` of `flatMap`.

Luckily we can do it easily, because the signature of `flatMap` looks very similar to the definition of `OrderedEffect`.

```scala
def flatMap[A, B](fa: FileOps[A])(f: A => FileOps[B]): FileOps[B] = fa match {
  case NoEffect(v) => f(v)
  case _ => OrderedEffect(fa, f)
}
```

Let's put them together.

```scala
sealed trait FileOps[A]
case class ReadFile(path:String) extends FileOps[String]
case class SaveFile(path:String, content:String) extends FileOps[Boolean]
case class OrderedEffect[A, B](effect1:FileOps[A], effect2Callback: A => FileOps[B]) extends FileOps[B]
case class NoEffect[A](v: A) extends FileOps[A]

def map[A, B](fa: FileOps[A])(f: A => B): FileOps[B] = fa match {
  case ReadFile(path) => 
    OrderedEffect(fa, (x:A) => NoEffect(f(x)))
  case SaveFile(path, content) =>
    OrderedEffect(fa, (x:A) => NoEffect(f(x)))
  case NoEffect(v) => NoEffect(f(v))
  case OrderedEffect(effect1, f1) => 
    OrderedEffect(effect1, (x:Any) => map(f1(x))(f))
}

def flatMap[A, B](fa: FileOps[A])(f: A => FileOps[B]): FileOps[B] = fa match {
  case NoEffect(v) => f(v)
  case _ => OrderedEffect(fa, f)
}
  

def copy(from:String, to:String):FileOps[Boolean] = {
  OrderedEffect[String, Boolean](ReadFile(from), (content:String) => SaveFile(to, content))
}
```

Please recall the purpose of Functional Programming, we are not to remove the `Side-Effect` totally, we just want to centralize the `Side-Effect`.
Here what we want is not get an ADT for `copy`, we want to copy a file to another location, which mean we want the `Side-Effect` happens eventually.

So we need to evaluate the effect of ADT, the intuitional implementation looks like this

```scala
def run[A](fa: FileOps[A]): A = fa match {
  case ReadFile(path) => 
    Source.fromFile(path).mkString
  case SaveFile(path, content) => try {
    val file = new File(path)
    val bw = new BufferedWriter(new FileWriter(file))
    bw.write(content)
    bw.close()
    true
  } catch {
    case e => 
      false
  }
  case OrderedEffect(effect1, f1) => 
    f1(run(effect1))
  case NoEffect(v) => v
}
```

Then our main function may looks like this

```scala
def main(args: Array[String]):Unit = {
  val program: FileOps[Boolean] = copy('a.txt', 'b.txt')

  run(program)
}
```

# Refactor

Assume we also want to make this function pure

```scala
def div(x:Double, y:Double): Double = {
  if(y == 0){
   println(s"/ by zero")
   throw new Exception("/ by zero")
  }else{
   x/y
  }
} 
```

According to the previous section, we may define our ADT like this

```scala
trait Response[+A]
case class Success[+A](v: A) extends Response[A]
case class Error(message: String) extends Response[Nothing]
case class Log(message: String) extends Response[Unit]
case class ResponseOrderedEffect[A, B](effect1: Response[A], effect2Callback: A => Response[B]) extends Response[B]
case class ResponseNoEffect[A](v: A) extends Response[A]
```

Then the `div` function can be pure

```scala
def div(x: Double, y: Double): Response[Double] = {
  if(y ==0 ) {
    ResponseOrderedEffect(Log("/ by zero"), _ => Error("/ by zero"))
  }else{
    Success(x/y)
  }
}
```

And the `map` and `flatMap` of `Response` can be implemented like this

```scala
def flatMap[A, B](fa: Response[A])(f: A => Response[B]): Response[B] = fa match {
  case ResponseNoEffect(v) => f(v)
  case _ => ResponseOrderedEffect(fa, f)
}
```

```scala
def map[A, B](fa: Response[A])(f: A => B): Response[B] = fa match {
  case Success(v) => 
    Success(f(v))
  case Error(e) => 
    Error(e)
  case Log(m) =>
    OrderedEffect(fa, (x:A) => NoEffect(f(x)))
  case ResponseNoEffect(v) =>
    ResponseNoEffect(f(v))
  case OrderedEffect(effect1, f1) => 
    ResponseOrderedEffect(effect1, (x:A) => map(f1(x))(f))
}
```

Then evaluate the `Response`

```scala
def run[A](fa: Response[A]): A = fa match {
  case Success(v) => 
    v
  case Error(e) => 
    throw new Exception(e)
  case Log(m) =>
    println(m)
  case ResponseOrderedEffect(effect1, f1) => 
    f1(run(effect1))
  case ResponseNoEffect(v) => v
}
```

We found both the `Response` ADT and `FileOps` ADT have the effect `NoEffect` and `OrderedEffect`.
And the behavior of these two effects in `map`, `flatMap` and `run` are same.
Could we remove the duplication?

In terms of the usage of `OrderedEffect`, it can be used on any effect to represent the effect order, so we may create an special ADT to extract this part.

The intuitional implementation is

```scala
trait EffectRelation[F[_], A]
case class OrderedEffect[F[_], A, B](fa: F[A], f: A => F[B]) extends EffectRelation[F, B]
case class NoEffect[F[_], A](a: A) extends EffectRelation[F, A]
```

Here we just use `F[_]` to replace the inital ADT, but there is a restriction on `f: A => F[B]`.
The `f` return `F[_]` which cannot represent the order of effect, then the implementation of `f` will be very painful.

So let's do a minor change, change `f: A => F[B]` to `f: A => EffectRelation[F, B]`, then `f` will be more flexible.

```scala
trait EffectRelation[F[_], A]
case class OrderedEffect[F[_], A, B](fa: F[A], f: A => EffectRelation[F, B]) extends EffectRelation[F, B]
case class NoEffect[F[_], A](a: A) extends EffectRelation[F, A]
```

Because we don't know the exact type of `F`, we need a callback function to help us evaluate the `EffectRelation`, it may looks like this

```scala
def run[F[_], A](fa: EffectRelation[A], runF: F[A] => A): A = fa match {
  case OrderedEffect(effect1, f1) => 
    run(f1(runF(effect1)), runF)
  case NoEffect(v) => v
}
```

But the function can not be compiled, `runF` here need an input `F[A]`, the type of `effect1` may be not `F[A]`(most of time it's not `F[A]`), it may be any type.
So for `runF`, it should be able to process any type, not only `A` in `run`. if we define a function for `runF`, it should looks like this

```scala
def runF[F[_], A](fa: F[A]): A = ???
```

What's the type of this function? Here we need to use the [Type lambdas](https://underscore.io/blog/posts/2016/12/05/type-lambdas.html) of Scala

```scala
({ type T[A] = F[A] => A })#T
```

Then then the `run` becomes


```scala
def run[F[_], A](fa: EffectRelation[A], runF: ({ type T[C] = F[C] => A })#T): A = fa match {
  case OrderedEffect(effect1, f1) => 
    run(f1(runF(effect1)))
  case NoEffect(v) => v
}
```

We can also define `map` and `flatMap` for `EffectRelation` which are just parts of the `map` and `flatMap` of `FileOps` and `Response`.

```scala
def map[F[_], A, B](fa:EffectRelation[F[_], A])(f: A => B): EffectRelation[F[_], B] = fa match {
  case OrderedEffect(effect1, f1) =>
    OrderedEffect(effect1, (x:Any) => map(f1(x))(f))
  case NoEffect(v) => NoEffect(f(v))
}

def flatMap[F[_], A, B](fa:EffectRelation[F[_], A])(f: A => EffectRelation[F[_],B]): EffectRelation[F[_], B] = fa match {
  case OrderedEffect(effect1, f1) =>
    OrderedEffect(effect1, (x:Any) => flatMap(f1(x))(f))
  case NoEffect(v) => f(v)
}
```

After extract the `EffectRelation` ADT, the `FileOps` just need to define `ReadFile` and `SaveFile` effects 

```scala
sealed trait FileOps[A]
case class ReadFile(path:String) extends FileOps[String]
case class SaveFile(path:String, content:String) extends FileOps[Boolean]

def runFileOps[A](fa: FileOps[A]): A = fa match {
  case ReadFile(path) => 
    Source.fromFile(path).mkString
  case SaveFile(path, content) => try {
    val file = new File(path)
    val bw = new BufferedWriter(new FileWriter(file))
    bw.write(content)
    bw.close()
    true
  } catch {
    case e => 
      false
  }
}
```

To use `EffectRelation` to represent the effect order, we need to convert the `FileOps` to `EffectRelation`.

```scala
sealed trait FileOps[A]
case class ReadFile(path:String) extends FileOps[String]
case class SaveFile(path:String, content:String) extends FileOps[Boolean]

type FileOpsEffectRelation[A] = EffectRelation[FileOps, A]

def readFile(path:String): FileOpsEffectRelation[String] = 
  OrderedEffect[FileOps,String, String](ReadFile(path), (content:String) => NoEffect(content))

def saveFile(path:String, content:String): FileOpsEffectRelation[Boolean] = 
  OrderedEffect[FileOps,Boolean, Boolean](SaveFile(path, concent), (success:Boolean) => NoEffect(success))
```

Then `copy` becomes

```scala
def copy(from:String, to:String):FileOpsEffectRelation[Boolean] = for {
  content <- readFile(from)
  success <- saveFile(to, content)
} yield success
```

And `main` will looks like this

```scala
def main(args: Array[String]):Unit = {
  val program: FileOpsEffectRelation[Boolean] = copy('a.txt', 'b.txt')

  run(program, runFileOps)
}
```

Sometimes we also want to make the `runF` pure, so we may define an ADT `G[_]` to achieve this.
Then the type of `runF` will become

```scala
({ type T[A] = F[A] => G[A] })#T
```

And the `run` of `EffectRelation` becomes

```scala
def run[F[_], G[_], A](fa: EffectRelation[A], runF: ({ type T[C] = F[C] => G[C] })#T): G[A] = fa match {
  case OrderedEffect(effect1, f1) => 
    run(f1(runF(effect1)))
  case NoEffect(v) => v
}
```

This function can't compile now, because if `f1` requires `C`, `runF` returns `G[C]` which is mismatched. To make the code work, `G[_]` need to be a monad here.

```scala
def run[F[_], G[_]: Monad, A](fa: EffectRelation[A], runF: ({ type T[C] = F[C] => G[C] })#T): G[A] = fa match {
  case OrderedEffect(effect1, f1) => 
    val fa1:G[EffectRelation[A]] = runF(effect1)
    fa1.flatMap((x:EffectRelation[A]) => run(f1(x), runF))
  case NoEffect(v) => v
}
```

Actually this function still support `runF` with type 

```scala
({ type T[A] = F[A] => A })#T
```

Because we can give `A` an alias `Id[A]` to change its kind to `G[_]` and it's a inborn monad.

```scala
type Id[A] = A 
```

We can also give the type of `runF` an alias to make it more readable

```scala
type ~>[F[_],G[_]] = ({ type T[A] = F[A] => G[A] })#T
```

And the `run` of `EffectRelation` becomes

```scala
def run[F[_], G[_]: Monad, A](fa: EffectRelation[A], runF: F ~> G): G[A] = fa match {
  case OrderedEffect(effect1, f1) => 
    val fa1:G[EffectRelation[A]] = runF(effect1)
    fa1.flatMap((x:EffectRelation[A]) => run(f1(x), runF))
  case NoEffect(v) => v
}
```

Now let's do a simple rename work

* `EffectRelation` => `Free` 
* `OrderedEffect` => `Impure` 
* `NoEffect` => `Pure` 

Then we can get the definition of `Free` monad

```scala
trait Free[F[_], A]
case class Impure[F[_], A, B](fa: F[A], f: A => Free[F, B]) extends Free[F, B]
case class Pure[F[_], A](a: A) extends Free[F, A]

def run[F[_], G[_]: Monad, A](fa: Free[A], runF: F ~> G): G[A] = fa match {
  case Impure(effect1, f1) => 
    val fa1:G[Free[A]] = runF(effect1)
    fa1.flatMap((x:Free[A]) => run(f1(x), runF))
  case Pure(v) => v
}

def map[F[_], A, B](fa:Free[F[_], A])(f: A => B): Free[F[_], B] = fa match {
  case Impure(effect1, f1) =>
    Impure(effect1, (x:Any) => map(f1(x))(f))
  case Pure(v) => Pure(f(v))
}

def flatMap[F[_], A, B](fa:Free[F[_], A])(f: A => Free[F[_],B]): Free[F[_], B] = fa match {
  case Impure(effect1, f1) =>
    Impure(effect1, (x:Any) => flatMap(f1(x))(f))
  case Pure(v) => f(v)
}
```

Here we don't have any restriction for `F[_]` which mean it can not be a functor or monad. What if we required it's a functor?
We found we can make `Impure` simpler, because we can apply `f` directly to `fa` with `map`, then the type will be `F[Free[F, B]]` and `Free` monad become

```scala
trait Free[F[_], A]
case class Impure[F[_], A](fa: F[Free[F, A]]) extends Free[F, A]
case class Pure[F[_], A](a: A) extends Free[F, A]

def run[F[_], G[_]:Monad, A](fa: Free[A], runF: F ~> G): G[A] = fa match {
  case Impure(f1) => 
    val fa1: G[Free[F, A]] = runF(f1)
    fa1.flatMap(x => run(x)(runF))
  case Pure(v) => Monad[G].pure(v)
}

def map[F[_]:Functor, A, B](fa:Free[F[_], A])(f: A => B): Free[F[_], B] = fa match {
  case Impure(f1) =>
    val fa1: F[Free[F, B]] = f1.map((x:Free[F, A]) => map(x)(f) )
    Impure(fa1)
  case Pure(v) => Pure(f(v))
}

def flatMap[F[_]:Functor, A, B](fa:Free[F[_], A])(f: A => Free[F[_],B]): Free[F[_], B] = fa match {
  case Impure(f1) =>
    val fa1: F[Free[F, B]] = f1.map((x:Free[F, A]) => flatMap(x)(f) )
    Impure(fa1)
  case Pure(v) => f(v)
}
```

# Summary

Returning multiple effects in order is a common requirement when making a function pure, so we extract an ADT `EffectRelation` to help one ADT to achieve this.
The definition of `EffectRelation` can be different depend on the restriction of target ADT.

* For the ADT without any restriction, we call the implementation `Freer` monad
  ```scala
  trait Free[F[_], A]
  case class Impure[F[_], A, B](fa: F[A], f: A => Free[F, B]) extends Free[F, B]
  case class Pure[F[_], A](a: A) extends Free[F, A]
  ```

* For the ADT without functor requirement, we call the implementation `Free` monad
  ```scala
  trait Free[F[_], A]
  case class Impure[F[_], A](fa: F[Free[F, A]]) extends Free[F, A]
  case class Pure[F[_], A](a: A) extends Free[F, A]
  ```

Actually, cats implement the `Freer` monad, but they call it `Free`.
