---
title: 'Trampoline: Eliminate StackOverlfowError'
excerpt: Are you familiar with this error? do you remember the nightmare when writing
  recursive methods? In this post, we will talk about a technique to eliminate StackOverflowError
  based on HotSpot JVM.
tags:
- Java
- fp
header:
  overlay_image: https://images.shangjiaming.com/charles-cheng-jVAu6BbJPro-unsplash.jpeg
  overlay_filter: 0.5
  caption: 'Photo credit: [**Unsplash**](https://unsplash.com)'
date: 2022-07-24 23:14 +0800
---
![image.png](https://images.shangjiaming.com/image-20220712230702-sun3d43.png)

Are you familiar with this error? do you remember the nightmare when writing recursive methods? In this post, we will talk about a technique to eliminate `StackOverflowError` based on HotSpot JVM.

# What is Java Virtual Machine Stack?

To understand this error, we need to know the Java Virtual Machine Stack first.

Java Virtual Machine Stack is also called Java Stack. It is used to store stack frames, which record the current state of the current method in the current thread. When we create a thread, a Java Virtual Machine Stack will be created at the same time and only the corresponding thread can operate it.

A stack frame consists of three parts: local variables, operand stacks, and dynamic links. When the current method **A** invokes another method **B**, a new stack frame of **B** will be pushed into the Java Virtual Machine Stack. After method **B** is complete normally or abruptly, **B**'s stack frame will be popped out, and **A**'s stack frame is on the top again, then it will be used to restore the state of **A** to continue the process.

Say we have a function `factorial`

```java
public Long factorial(Long n) {
    if (n == 1) {
        return 1l;
    }
    return n * factorial(n - 1);
}
```

The Java Virtual Machine Stack of `factorial(4)` looks like

![image.png](https://images.shangjiaming.com/image-20220715105519-how08st.png)

The Java Virtual Machine Stack size of each thread is limited, we can check the default size by

```sh
java -XX:+PrintFlagsFinal -version|grep ThreadStackSize
```

We can also specify the maximum stack size

```sh
java -Xss2M //set the maximum thread stack size to be 2M
```

[The minimum stack size of different OS are](https://github.com/openjdk/jdk/search?p=1&q=_java_thread_min_stack_allowed)

|OS|Default Stack Size|
| ---------------| --------------------|
|Windows<br />|40KB|
|Linux AArch64|72KB|
|Linux RISC-V|72KB|
|Linux s390|32KB|
|Linux ARM|32KB|
|Linux x86|40KB|

# What is StackOverflowError?

According to the [Java Virtual Machine Specification](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.5.2),  If the computation in a thread requires a larger Java Virtual Machine Stack than is permitted, the Java Virtual Machine throws a `StackOverflowError`.

For example, if we call `factorial(10000l)`, there will be a `StackOverflowError`, because it exhausts the Java Virtual Machine Stack before calling `factorial(1l)` which starts to pop the stack frame.

```java
factorial(10000l);

java.lang.StackOverflowError
 at io.github.sjmyuan.trampoline.StackOverflowTest.factorial(StackOverflowTest.java:12)
 at io.github.sjmyuan.trampoline.StackOverflowTest.factorial(StackOverflowTest.java:12)
 at io.github.sjmyuan.trampoline.StackOverflowTest.factorial(StackOverflowTest.java:12)
 at io.github.sjmyuan.trampoline.StackOverflowTest.factorial(StackOverflowTest.java:12)
 at io.github.sjmyuan.trampoline.StackOverflowTest.factorial(StackOverflowTest.java:12)
 at io.github.sjmyuan.trampoline.StackOverflowTest.factorial(StackOverflowTest.java:12)
 .....
```

# Tail call and Tail recursion

A tail call is a method invocation which is the final action of the current method. For example

```java
public Integer add(Integer x, Integer y) {
  return x + y;
}

public Ineger substract(Integer x, Integer y) {
  return add(x, -1*y); // tail call
}
```

In the above code, the invocation of `add(x, <result of -1*y>)` is a tail call.

But in the `factorial` example, `factorial(n-1)` is not a tail call, because the final action is `*`

```java
public Long factorial(Long n) {
    if (n == 1) {
        return 1l;
    }
    Long nextFacorial = factorial(n-1); // not tail call
    return n * nextFactorial; // final action of method
}
```

If the tail call is the same as the current method, we say the current method is tail recursion. For example

```java
public Long fibonacci(Long n, Long a, Long b) { // tail recursion
    if (n == 0) {
        return a;
    }
    if (n == 1) {
        return b;
    }
    return fibonacci(n - 1, b, a + b); // tail call
}
```

# How to eliminate tail recursion?

Tail recursion can be eliminated by a `while` loop. For example

```java
public Long fibonacci(Long n, Long a, Long b) {
    Long nParam = n;
    Long aParam = a;
    Long bParam = b;
    while (true) {
        if (nParam == 0) {
            return aParam;
        }
        if (nParam == 1) {
            return bParam;
        }
        nParam = nParam - 1;
        Long aCurrent = aParam;
        aParam = bParam;
        bParam = aCurrent + bParam;
    }
}
```

We can use the following steps to eliminate any tail recursion

1. Create a local variable for each parameter, like`nParam`, `aParam`, and `bParam`.
2. Wrap the body of the method with a `while(true)` loop, and replace the reference of the parameter with the reference of the corresponding local variable, like lines 5 - 15.
3. Replace the tail call of the current method with local variables assignment, which assigns the parameter of the tail call to the corresponding local variable, like lines 12 - 15.

It's possible to eliminate the tail recursion by the compiler automatically, Scala already implemented it, but Java does not support it yet. One of the reasons is polymorphic, the compiler won't know if the method has been overridden by a child, so it can't use the current method body to eliminate the tail recursion. Even in Scala, it requires the tail recursion method to be private or final, which can't be overridden.

# How to eliminate StackOverflowError?

A method without method invocation won't throw `StackOverflowError`.

Let's focus on the method with method invocation. We know

1. `StackOverflowError` is caused by the limited size of the Java Virtual Machine Stack
2. We can eliminate the tail recursion by a `while` loop that requires fewer stack frames.
3. The heap size is larger than the Java Virtual Machine Stack size

Could we utilize tail recursion and heap to eliminate `StackOverflowError`? The answer is trampoline\.

## What is trampoline?

[Trampoline](https://en.wikipedia.org/wiki/Trampoline_(computing)) is a technique to trade stack for heap. The root cause of `StackOverflowError` is the current method need to wait for the method invocation to complete to release the stack frame. Trampoline allows the current method to release the stack frame immediately after the method invocation because it stores the invocation state in the heap, not in the stack.

The trade-off is we need to pick up the invocation state from the heap to execute, so there is a scheduler in trampoline to do this work and we can implement it by tail recursion.

Not like normal method invocation, the stack of the trampoline will increase and decrease just like a trampoline

![image.png](https://images.shangjiaming.com/image-20220717111510-0gbh82e.png)

## CPS

To apply trampoline easily, we need to convert the method from direct style to [CPS](https://en.wikipedia.org/wiki/Continuation-passing_style) first.

CPS means continuation-passing style, it is a style of programming in which control is passed explicitly in the form of a continuation.

To write a CPS method, we need to pass an extra argument, its type is function and we call it continuation. When the current method completes the computation, it will pass the result to the continuation instead of returning it.

For example, we can rewrite `factorial` to CPS

```java
public void factorial(Long n, Consumer<Long> continuation) {
    if (n == 1) {
        continuation.accept(1l);
        return;
    }
    factorial(n - 1, (Long result) -> continuation.accept(n * result));
}
```

It's a tail recursion now, we can convert it to a `while` loop

```java
public void factorial(Long n, Consumer<Long> continuation) {
    Long nParam = n;
    Consumer<Long> continuationParam = continuation;
    while (true) {
        if (nParam == 1) {
            continuationParam.accept(1l);
            return;
        }
        Long nCurrent = nParam;
        nParam = nParam - 1;

        final Consumer<Long> currentContinuation = continuationParam;
        continuationParam = (Long result) -> currentContinuation.accept(nCurrent * result);
    }
}
```

But it still throws `StackOverflowError`

```java
factorial(10000l, (x) -> {});

java.lang.StackOverflowError
 at java.base/java.lang.Long.longValue(Long.java:1353)
 at io.github.sjmyuan.trampoline.CPSTest.lambda$1(CPSTest.java:28)
 at io.github.sjmyuan.trampoline.CPSTest.lambda$1(CPSTest.java:28)
 at io.github.sjmyuan.trampoline.CPSTest.lambda$1(CPSTest.java:28)
 at io.github.sjmyuan.trampoline.CPSTest.lambda$1(CPSTest.java:28)
 at io.github.sjmyuan.trampoline.CPSTest.lambda$1(CPSTest.java:28)
 at io.github.sjmyuan.trampoline.CPSTest.lambda$1(CPSTest.java:28)
 at io.github.sjmyuan.trampoline.CPSTest.lambda$1(CPSTest.java:28)
 at io.github.sjmyuan.trampoline.CPSTest.lambda$1(CPSTest.java:28)
 ....
```

`continuationParam.accept(1l)` in line 6 thrown the error, because a new `currentConinuation` call will be made in each loop in line 13.

![image](https://images.shangjiaming.com/image-20220723094523-xiynkhq.png)

Not all the CPS methods are tail recursion, for example

```java
public boolean isEven(Long n) {

    if (n == 0)
        return true;
    return isOdd(n - 1);

}

public boolean isOdd(Long n) {

    if (n == 0)
        return false;
    return isEven(n - 1);

}
```

We can rewrite these two methods to CPS

```java
public void isEven(Long n, Consumer<Boolean> continuation) {
    if (n == 0) {
        continuation.accept(true);
        return;
    }
    isOdd(n - 1, (result) -> continuation.accept(result));
}

public void isOdd(Long n, Consumer<Boolean> continuation) {

    if (n == 0) {
        continuation.accept(false);
        return;
    }
    isEven(n - 1, (result) -> continuation.accept(result));
}
```

The above method invocations are tail calls, but `isEven` and `isOdd` are not tail recursion.

We can't eliminate`StackOverflowError`by CPS, and it's easy to make mistakes using CPS, but all the method invocations are tail calls and the execution orders are explicit, which are easier to apply trampoline.

## How to implement trampoline?

A trampoline is a loop that iteratively invokes thunk-returning functions. A [thunk](https://en.wikipedia.org/wiki/Thunk) is a function without argument, like `Supplier` in Java or `Lazy` in Scala

```java
Supplier<Long> thunk = () -> 1l
```

To eliminate the `StackOverflowError`, we need to take over the control of method invocation from JVM. We can wrap the method invocation in a thunk, then the method won't be invoked until we run the thunk

```java
Supplier<Long> thunk = () -> factorial(4l); // create a Supplier instance, won't run factorial(4l)
thunk.get(); // run factorial(4)
```

If we use direct style, we have to run the thunk in the same method, which makes no sense to use it

```java
private Long factorial(Long n) {
    if (n == 1) {
        return 1l;
    }
    Supplier<Long> thunk = () -> factorial(n-1);
    return n * thunk.get();
}
```

If we use CPS, we also need to run thunk in the same method

```java
public void factorial(Long n, Consumer<Long> continuation) {
    if (n == 1) {
        Supplier<Void> thunk = () -> {
            continuation.accept(1l);
            return null;
        };
        think.get();
        return;
    }
    Supplier<Void> thunk = () -> {
        factorial(n - 1, (Long result) -> continuation.accept(n * result));
        return null;
    };
    thunk.get();
}
```

We don't want the current method to run the thunk, and we can see the last action of the CPS method is `thunk.get()`, then we can just return the thunk and decide when to run it by ourself

```java
public Supplier<Void> factorial(Long n, Consumer<Long> continuation) {
    if (n == 1) {
        Supplier<Void> thunk = () -> {
            continuation.accept(1l);
            return null;
        };
        return thunk;
    }
    Supplier<Void> thunk = () -> {
        Supplier<Void> thunkContinuation =
                factorial(n - 1, (Long result) -> continuation.accept(n * result));
        thunkContinuation.get();
        return null;
    };
    return thunk;
}
```

We can't do this in direct style, because other actions depend on the thunk result.

But this method can still throw `StackOverflowError`, because `thunk.get()`will call `thunkContinuation.get()`. We return the tail call of `factorial` by thunk, but miss the tail call in thunk.

To get the tail call in thunk, we need to change the thunk signature from `Supplier<Void>` to `Supplier<Supplier<Supplier<....>>>`. We can't define the recursive type only by `Supplier`, so we involve another type

```java
public class More {
    private Supplier<More> thunk;
    public More(Supplier<More> thunk){
        this.thunk = thunk;
    }
}
```

Then we can return `More` as thunk.

```java
public More factorial(Long n, Function<Long, More> continuation) {
    if (n == 1) {
        return new More(() -> continuation.apply(1l));
    }
    return new More(() -> factorial(n - 1, (Long result) -> new More(() -> continuation.apply(n * result))));
}
```

We also change the signature of `continuation`, there are two reasons

1. `continuation` may also throw `StackOverflowError`, which needs to apply trampoline
2. We can't construct `More` by `Supplier<Void>` when `n == 1`

Then we can iterate thunk to calculate the result

```java
public static void run(More trampoline) {
    run(trampoline.getThunk().get());
}
```

But you may find we can't call `factorial`, because we can't construct `More` in `continuation`, if we want to construct `More`, we need another `More` instance, which is a dead loop.

```java
More trampoline = factorial(4l, x -> new More(y -> new More(z -> ....)))
```

So we need a new class `Done` to signal the end of the computation, it should have the same parent as `More`

```java
public interface Trampoline {
}

public class Done implements Trampoline {

    public Done() {
    }
}

public class More implements Trampoline {
    private Supplier<Trampoline> thunk;

    public More(Supplier<Trampoline> thunk) {
        this.thunk = thunk;
    }
}
```

The `run` method also need to detect the `Done` signal to end the computation

```java
public static void run(Trampoline trampoline) {

    if (trampoline instanceof Done) {
        return;
    }

    run(((More) trampoline).getThunk().get());
}
```

Now the `factorial` become

```java
public Trampoline factorial(Long n, Function<Long, Trampoline> continuation) {
    if (n == 1) {
        return new More(() -> continuation.apply(1l));
    }

    return new More(() -> factorial(n - 1, (Long result) -> new More(() -> continuation.apply(n * result))));
}
```

And we can call it

```java
Trampoline trampoline = factorial(4l, x -> new Done())
run(trampoline);
```

Remember that tail recursion can be eliminated by `while` loop, there is no `StackOverflowError` even we call `factorial(10000l, x -> Done())`

```java
public static void run(Trampoline trampoline) {

    Trampoline trampolineParam = trampoline;

    while (true) {

        if (trampolineParam instanceof Done) {
            return;
        }

        trampolineParam = ((More) trampolineParam).getThunk().get();
    }
}
```

Let's summarize how to apply trampoline to the CPS method

1. Change return type from void to `Trampoline`
2. Change continuation from `Consumer<Long>` to `Function<Long, Trampoline>`
3. Replace all the tail calls with `More`, including the tail call in continuation.
4. Iterate `Trampoline` by `run` method

The stack state of `run(facorial(4))` is

![image](https://images.shangjiaming.com/image-20220723225112-7hoveun.png)

We can see that each method invocation return immediately and there are no accumulated stack frames.

But If we don't replace the tail call with `More` in continuation, the implementation will be

```java
public Trampoline factorial(Long n, Function<Long, Trampoline> continuation) {
    if (n == 1) {
        return new More(() -> continuation.apply(1l));
    }

    return new More(() -> factorial(n - 1,
            result -> continuation.apply(n * result))); // call continuation.apply directly

}
```

It will throw `StackOverflowError`, the stack state of `run(facorial(4))` is

![image](https://images.shangjiaming.com/image-20220723235357-rnwtdw3.png)

We can see that the stack frames are accumulated when calling `continuation3(1)` because it doesn't return thunk, JVM invokes nested `continuation` and store method state in stack automatically, we can't control it by trampoline.

## How to make it easy to use?

There are some pain points for the usage of trampoline

1. CPS is not customary in our daily work
2. Easy to make mistake when replacing tail call with `More`

CPS uses the stack to store the continuation function, which passes the continuation as a method argument. Considering all the methods that will apply trampoline, we can use the heap to store the relationship between result and continuation.

Let's add one more class `FlatMap` like this

```java
public class FlatMap<A, B> implements Trampoline<B> {

    private Trampoline<A> lastResult;

    private Function<A, Trampoline<B>> continuation;

    public FlatMap(Trampoline<A> lastResult, Function<A, Trampoline<B>> continuation) {

        this.lastResult = lastResult;
        this.continuation = continuation;
    }
}
```

We store both results of the last expression and the following continuation in `FlatMap`, and we allow the continuation to convert the expression result to any type.

Because we use trampoline to store the expression result, `Do` and `More` need to be refactored

```java
public interface Trampoline<T> {

}

public class Done<T> implements Trampoline<T> {

    private T result;

    public Done(T result) {
        this.result = result;
    }
}

public class More<T> implements Trampoline<T> {

    private Supplier<Trampoline<T>> thunk;

    public More(Supplier<Trampoline<T>> thunk) {

        this.thunk = thunk;

    }
}
```

The `run` method also needs to handle `FlatMap`

1. If the last result is `Done`, just apply continuation to the result
2. If the last result is `More`, invoke the thunk and create a new `FlatMap` to apply continuation to the thunk result
3. If the last result is `FlatMap`, change `FlatMap(FlatMap(trampoline, g), f) `to `FlatMap(trampoline, x -> FlatMap(g(x), f))`

```java
public static <S> S run(Trampoline<S> trampoline) {

    if (trampoline instanceof Done) {
        return ((Done<S>) trampoline).getResult();
    } else if (trampoline instanceof More) {
        return run(((More<S>) trampoline).getThunk().get());
    } else {

        FlatMap<Object, S> continuation = (FlatMap<Object, S>) trampoline;

        Trampoline<Object> lastResult = continuation.getLastResult();
        Function<Object, Trampoline<S>> continuationFunc = continuation.getContinuation();

        if (lastResult instanceof FlatMap) {

            FlatMap<Object, Object> lastResultContinuation =
                    (Continuation<Object, Object>) lastResult;

            return run(new FlatMap<Object, S>(lastResultContinuation.getLastResult(),
                    x -> new FlatMap<Object, S>(
                            lastResultContinuation.getContinuation().apply(x),
                            continuationFunc)));
        } else if (lastResult instanceof More) {

            return run(new FlatMap<Object, S>(((More<Object>) lastResult).getThunk().get(),
                    continuationFunc));
        } else {
            return run(continuationFunc.apply(((Done<Object>) lastResult).getResult()));
        }
    }
}
```

Now we don't need to convert the method to CPS, we can apply trampoline directly like this

```java
private Trampoline<Long> factorial(Long n) {
    if (n == 1) {
        return new Done<Long>(1l);
    }
    return new FlatMap<Long, Long>(factorial(n - 1), x -> new Done<Long>(n * x));
}
```

Oops! it still throws `StackOverflowError`, why?

```java
Trampoline<Long> trampoline = factorial(10000l);

Trampoline.run(trampoline);

java.lang.StackOverflowError
	at io.github.sjmyuan.trampoline.v3.TrampolineTest.factorialTrampoline(TrampolineTest.java:11)
	at io.github.sjmyuan.trampoline.v3.TrampolineTest.factorialTrampoline(TrampolineTest.java:11)
	at io.github.sjmyuan.trampoline.v3.TrampolineTest.factorialTrampoline(TrampolineTest.java:11)
	at io.github.sjmyuan.trampoline.v3.TrampolineTest.factorialTrampoline(TrampolineTest.java:11)
	at io.github.sjmyuan.trampoline.v3.TrampolineTest.factorialTrampoline(TrampolineTest.java:11)
	at io.github.sjmyuan.trampoline.v3.TrampolineTest.factorialTrampoline(TrampolineTest.java:11)
	at io.github.sjmyuan.trampoline.v3.TrampolineTest.factorialTrampoline(TrampolineTest.java:11)
	at io.github.sjmyuan.trampoline.v3.TrampolineTest.factorialTrampoline(TrampolineTest.java:11)
	at io.github.sjmyuan.trampoline.v3.TrampolineTest.factorialTrampoline(TrampolineTest.java:11)
	at io.github.sjmyuan.trampoline.v3.TrampolineTest.factorialTrampoline(TrampolineTest.java:11)
        ....
```

The reason is we call `factorial(n - 1)` directly

```java
new FlatMap<Long, Long>(factorial(n - 1), x -> new Done<Long>(n * x))
```

Before we create the instance of `FlatMap`, we need to wait for the return of `factorial(n - 1)`, which will create another `FlatMap` and wait for the return of `factorial(n - 2)`. The stack frames are accumulated.

We can use `More` to avoid this scenario.

```java
new FlatMap<Long, Long>(new More<Long>(() -> factorial(n - 1)),
    x -> new Done<Long>(n * x));
```

The most important thing here is to wrap all the method invocation with `More`.

But it's still not convenient to construct the trampoline class, we can add some utility methods

```java
public static <A> Trampoline<A> of(A v) {
    return new Done<A>(v);
}

public static <A> Trampoline<A> suspend(Supplier<Trampoline<A>> thunk) {
    return new More<A>(thunk);
}


<B> Trampoline<B> flatMap(Function<T, Trampoline<B>> continuation);
```

`flatMap` is a method implemented by each class

```java
// Done and More
public <B> Trampoline<B> flatMap(Function<T, Trampoline<B>> continuation) {
    return new FlatMap<T, B>(this, continuation);
}

//FlatMap
public <C> Trampoline<C> flatMap(Function<B, Trampoline<C>> nextContinuation) {
    return new FlatMap<A, C>(lastResult,
            x -> Trampoline.suspend(() -> continuation.apply(x)).flatMap(nextContinuation));
}
```

We will make the constructor of `FlatMap` to be protected, then we can apply the strict rule to the usage of `FlatMap`to avoid the direct call when constructing it.

With these methods, `factorial` can be cleaner

```java
public Trampoline<Long> factorial(Long n) {
    if (n == 1) {
        return Trampoline.of(1l);
    }
    return Trampoline.suspend(() -> factorial(n - 1))
            .flatMap(x -> Trampoline.of(n * x));
}
```

# Summary

Trampoline is an important technique to eliminate `StackOverflowError` in functional programming. It's also essential knowledge to understand the implementation details of functional programming libraries.

According to the implementation, trampoline is a [Free Monad](https://blog.shangjiaming.com/scala%20tutorial/free-monad/), the missed part is the`map`

```java
public <B> Trampoline<B> map(Function<T, B> continuation) {
    return new FlatMap<T, B>(this, x -> Trampoline.suspend(() -> Trampoline.of(continuation.apply(x))));
}
```

Most of the above contents are explanations of [Stackless Scala With Free Monads](https://blog.higher-order.com/assets/trampolines.pdf) in Java, but there are some points I don't understand

1. The `StackOverflowError` caused by the left-leaning tower of FlatMaps in section **4.3 An easy thing to get wrong**

    I tried to reproduce this scenario by

    ```java
    @Test
    public void tooManyLeftAssociateContinuationWillNotThrowError() {
        Trampoline<Long> trampoline = new Done<Long>(1l);
        for (int i = 1; i < 50000; i++) {
            trampoline = new FlatMap<Long, Long>(trampoline, x -> new Done<Long>(x));
        }

        assertThat(Trampoline.run(trampoline)).isEqualTo(1);
    }
    ```

    But it didn't throw `StackOverflowError`, and I also draw the stack state, it won't blow up the stack unless some continuation blows up it. It's not the problem of left-associate FlatMaps, it's the problem of continuation which is out of our control.

    ![image.png](https://images.shangjiaming.com/image-20220718131933-thox8qo.png)
2. The flatMap implementation in **4.3 An easy thing to get wrong**

    The flatMap implementation in the article is

    ```scala
    def flatMap [B ](
        f: A => Trampoline [B ]): Trampoline [ B] =
            this match {
                case FlatMap (a , g) =>
                    FlatMap (a , (x: Any ) = > g (x) flatMap f)
                case x => FlatMap (x , f )
            }
    ```

    It throws `StackOverflow` in the following scenario

    ```java
    @Test
    public void tooManyFlatMapWillThrowError() {
        Trampoline<Long> trampoline = Trampoline.of(1l);
        for (int i = 1; i < 50000; i++) {
            trampoline = trampoline.flatMap(x -> Trampoline.of(x));
        }

        assertThat(Trampoline.run(trampoline)).isEqualTo(1);
    }
    ```

    The stack state is

    ![image](https://images.shangjiaming.com/image-20220724010127-quhn138.png)

    We can fix it by suspend `continuation.apply(x)`($g(x)$ in the article implementation)

    ```java
    //FlatMap
    public <C> Trampoline<C> flatMap(Function<B, Trampoline<C>> nextContinuation) {
        return new FlatMap<A, C>(lastResult,
                x -> Trampoline.suspend(() -> continuation.apply(x)).flatMap(nextContinuation));
    }
    ```

    The stack state is

    ![image](https://images.shangjiaming.com/image-20220720234240-778teua.png)

I already put all the code in [trampoline-example](https://github.com/sjmyuan/trampoline-example), welcome to review and let me know if there is any mistake, hope this post can help you to understand Trampoline.
