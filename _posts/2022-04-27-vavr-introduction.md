---
title: Vavr Introduction
excerpt: ''
tags:
- Java
- fp
header:
  overlay_image: https://images.shangjiaming.com/barn-images-t5YUoHW6zRo-unsplash.jpeg
  overlay_filter: 0.5
  caption: 'Photo credit: [**Unsplash**](https://unsplash.com)'
date: 2022-04-27 20:47 +0800
---
# What is Vavr?

[Vavr](https://www.vavr.io/) is a  Functional Programming library in Java 8+. The first version, called _Javaslang_, was released on 19h march 2014.

It mainly contains three parts
1. Immutable collections to avoid side-effects
2. ADT([Algebraic data type](https://blog.shangjiaming.com/scala%20tutorial/algebraic-data-type/)) to eliminate side-effects
3. Pattern matching to simplify the usage of ADT

I already created [vavr-examples](https://github.com/sjmyuan/vavr-examples), welcome to clone and play the features.

# How to install Vavr?

* Maven

    ```xml
    <dependencies>
        <dependency>
            <groupId>io.vavr</groupId>
            <artifactId>vavr</artifactId>
            <version>0.10.4</version>
        </dependency>
    </dependencies>
    ```

* Gradle

    ```
    dependencies {
        compile "io.vavr:vavr:0.10.4"
    }
    ```

* Gradel 7 +

    ```
    dependencies {
        implementation "io.vavr:vavr:0.10.4"
    }
    ```

# How to use Vavr?

[Functional Programming](https://blog.shangjiaming.com/scala%20tutorial/what-is-fp/) means constructing program with pure functions.

A function is pure when
* For all input, the same input produces the same output
* No Side-Effects

For example, the following example breaks the first rule

```java
public Integer generateRandomNumber() {
    Random random = new Random();
    return random.nextInt();
}

@Test
public void returnDifferentResultForEachCall() {
    assertThat(generateRandomNumber()).isNotEqualTo(generateRandomNumber());
}
```

If we call `generateRandomNumber` multiple times, it will generate a different `Integer`. We can not avoid this logic, because we need a random integer anyway. But We can move this logic to the boundary of the program to ensure most of the functions are pure. For example, we can pass a function to generate a random integer or generate the random integer in main and then pass it to business logic.

```java
public Integer generateRandomNumber(Supplier<Integer> generator) {
    return generator.get();
}

@Test
public void returnSameResultForEachCall() {
    assertThat(generateRandomNumber(() -> 1)).isEqualTo(generateRandomNumber(() -> 1));
}
```

In the following sections, we will focus on how to use Vavr to avoid or eliminate side-effect.

## Avoid Side-Effect

Is the `append` function pure?

```java
public void append(List<Integer> intList1, List<Integer> intList2) {
    intList1.addAll(intList2);
    return;
}

@Test
public void modifyParameterIsSideEffect() {

    List<Integer> intList1 = new LinkedList<Integer>();
    intList1.add(1);
    intList1.add(2);

    List<Integer> intList2 = new LinkedList<Integer>();
    intList2.add(3);
    intList2.add(4);

    this.append(intList1, intList2);

    assertThat(intList1.size()).isEqualTo(4);
    assertThat(intList2.size()).isEqualTo(2);
}
```

The answer is no. it returns void, but actually, changes the `intList1` which is outside of the function.

We can code like this because the `List` is mutable. To avoid the modification of the outside variable, we need to make the data type immutable.

Vavr redefines the common collection data type to make them immutable.		

### List

We can construct List by some util functions

```java
java.util.List<Integer> javaList = new java.util.LinkedList<Integer>();
javaList.add(1);
javaList.add(2);
javaList.add(3);

List.of(1,2,3);
List.ofAll(javaList);
```

The List is immutable, any changes to the list instance will create a new instance

```java
public List<Integer> append(List<Integer> intList1, List<Integer> intList2) {
    return intList1.appendAll(intList2);
}

@Test
public void modifyParameterWillGenerateNewList() {

    List<Integer> intList1 = List.of(1, 2);

    List<Integer> intList2 = List.of(1, 2);

    List<Integer> result = this.append(intList1, intList2);

    assertThat(result.size()).isEqualTo(4);
    assertThat(intList1.size()).isEqualTo(2);
    assertThat(intList2.size()).isEqualTo(2);
}
```

We can also compare the elements of List easily
```java
@Test
public void listCanCompareElement() {
    assertThat(List.of(1, 2)).isEqualTo(List.of(1, 2));
    assertThat(List.of(1, 2)).isNotEqualTo(List.of(1, 2, 3));
}
```

## Eliminate Side-Effect

### Option

It's very common to return `null` in Java, for example

```java
public Integer getHead(java.util.List<Integer> intList){
  if(intList.size() == 0 ){
     return null;
  } else {
     return intList.get(0);
  }
}

@Test
public void consumerNeedToCheckNull() {
    Integer head = getHead(new LinkedList<Integer>());

    String result = head == null ? "No Element" : head.toString();

    assertThat(result).isEqualTo("No Element");
}
```
But if one function may return `null`, the consumer of the function need always to check if the return value is `null`, that's why there are lots of ways to do nullable checking, such as `@NonNull`, `assert a != null` and `Objects.requireNonNull(a)`.

The reason is a wrapper type always stands for two possible values, `null` or normal value, which is implicit and easy to forget. Even worse if we forget to check `null`, the compiler won't remind us.

In Functional Programming, we'd like to eliminate `null`, then it's very easy to know what's the return value of the function, and don't need to guess if it will return null.

So `null` is a side-effect in Functional Programming, we need to find a way to return it more obviously. Vavr defines `Option` to make the function returning `null` pure.

The above example can be changed to 

```java
public Option<Integer> getHead(java.util.List<Integer> intList) {
    if (intList.size() == 0) {
        return Option.none();
    } else {
        return Option.some(intList.get(0));
    }
}

@Test
public void consumerAlwaysKnowOptionMaybeSomeOrNone() {
    Option<Integer> head = getHead(new java.util.LinkedList<Integer>());

    String result = head.fold(() -> "No Element", (x) -> x.toString());

    assertThat(result).isEqualTo("No Element");
}
```

We can use `Some` to wrap normal value, `None` to replace `null`

```java
Option.some(1);
Option.none();
```

We can compare its value easily

```java
assertThat(Option.some(1)).isEqualTo(Option.some(1));
assertThat(Option.none()).isEqualTo(Option.none());
```

And unwrap it easily

```java
assertThat(Option.some(1).get()).isEqualTo(1);
assertThat(Option.none().getOrNull()).isNull();
```

### Either

Do you think this is a pure function?

```java
public Integer divide(Integer x, Integer y) {
    if (y == 0) {
        throw new Error("The denominator can not be 0");
    } else {
        return x / y;
    }
}
```
The error is a side-effect because we don't know if the function will throw an error just according to its return type, we have to add some comment or use the `throws` keyword to supply more information. But it's still not convenient for the user to handle the error.

To make the function pure, we can use `Either`, then the function can be refactored like this
```java
public Either<Error, Integer> divide(Integer x, Integer y) {
    if (y == 0) {
        return Either.left(new Error("The denominator can not be 0"));
    } else {
        return Either.right(x / y);
    }
}
```

`Either` can wrap data of two different types, take `String` and `Integer` for example

```java
Either<String, Integer> errorOrInteger = Either.left("Error");
Either<String, Integer> errorOrInteger = Either.right(1);
```

It supports comparing values directly

```java
assertThat(Either.right(1)).isEqualTo(Either.right(1));
assertThat(Either.left("Error")).isEqualTo(Either.left("Error"));
```

And can be unwrapped

```java
@Test
public void shouldReturnWrappedValueForRight() {
    Either<String, Integer> intValue = Either.right(1);
    assertThat(intValue.get()).isEqualTo(1);

    Either<String, Integer> intValue2 = Either.left("Error");
    assertThatThrownBy(() -> intValue2.get()).isInstanceOf(NoSuchElementException.class)
            .hasMessageContaining("get() on Left");
}

@Test
public void shouldReturnWrappedValueForLeft() {
    Either<String, Integer> intValue = Either.left("Error");
    assertThat(intValue.getLeft()).isEqualTo("Error");

    Either<String, Integer> intValue2 = Either.right(1);
    assertThatThrownBy(() -> intValue2.getLeft()).isInstanceOf(NoSuchElementException.class)
            .hasMessageContaining("getLeft() on Right");
}
```

### Try

Sometimes we need to invoke other libraries, which may throw an error, for example

```java
public Integer add(String x, String y){
   Integer xInt = Integer.valueOf(x);
   Integer yInt = Integer.valueOf(y);
   return xInt + yInt;
}

@Test
public void stringToIntegerMayThrowError() {
    assertThatThrownBy(() -> {
        add("1", "a");
    }).isInstanceOf(IllegalArgumentException.class);
}
```
We can use regex expression to check if the parameters are integer string, but we also can use `Try` to make it easier

```java
public Try<Integer> add(String x, String y) {
    Try<Integer> xInt = Try.of(() -> Integer.valueOf(x));
    Try<Integer> yInt = Try.of(() -> Integer.valueOf(y));
    return xInt.flatMap(xv -> yInt.map(yv -> xv + yv));
}
@Test
public void tryCanHandleError() {
    assertThat(add("1", "a").getCause()).isInstanceOf(IllegalArgumentException.class);
}
```

`Try` can accept a function, return `Failure` if the function throws an error, and return `Success` if the function return value successfully.

It can also accept a constant value

```java
Try<Integer> = Try.success(1);
Try<Integer> = Try.failure(new Error("Error"));
```

And can be unwrapped
```java
assertThat(Try.success(1).get()).isEqualTo(1);
assertThatThrownBy(() -> {
    Try.success(1).getCause();
}).isInstanceOf(UnsupportedOperationException.class);
assertThat(Try.failure(new Error("Error")).getCause()).isInstanceOf(Error.class);
assertThatThrownBy(() -> {
    Try.faiure(new Error("Error")).get();
}).isInstanceOf(Error.class);
```

# How to code with functions?

In OO, there are lots of popular [design patterns](https://en.wikipedia.org/wiki/Design_Patterns) to organize the code.

In Functional Programing, there are lots of popular [higher order functions](https://en.wikipedia.org/wiki/Higher-order_function) to organize the code. 

In this section, considering we use ADT as the return type to wrap effects, we will call ADT a container to make it easy to understand.

## map

A `map` can apply a lambda function to a container, for example

```java
assertThat(Option.some(1).map(x -> x + 1)).isEqualTo(2);
assertThat(Option.none().map(x -> x + 1)).isEqualTo(Option.none());

assertThat(Either.right(1).map(x -> x + 1)).isEqualTo(Either.right(2));
assertThat(Either.left("Error").map(x -> x + 1)).isEqualTo(Either.left("Error"));

assertThat(Try.success(1).map(x -> x + 1)).isEqualTo(Try.success(2));
assertThat(Try.failure(new Error("Error")).map(x -> x + 1).isFailure()).isTrue();

assertThat(List.of(1,2,3).map(x -> x + 1)).isEqualTo(List.of(2,3,4));
```

Vavr uses the traditional `class method` to implement `map`, we can abstract it to a higher-order function, the pseudocode is

```java
public <M<_>, A, B> M<B> map(M<A> value, Function<A, B> f)
```

The code can't compile in Java, `M<_>` means any type requiring one type parameter, such as `Option<_>`, `Either<String, _>`, `Try<_>`, `List<_>` etc.

## flatMap

A `flatMap` can also apply a lambda function to a container, but not like a `map`, the lambda function will return the type of container. If `M.map` accept function `A->B`, then `M.flatMap` accept function `A -> M<B>`

for example

```java
assertThat(Option.some(1).flatMap(x -> Option.some(x + 1))).isEqualTo(2);
assertThat(Option.some(1).flatMap(x -> Option.none())).isEqualTo(Option.none());
assertThat(Option.none().flatMap(x -> Option.some(x + 1))).isEqualTo(Option.none());
assertThat(Option.none().flatMap(x -> Option.none())).isEqualTo(Option.none());

assertThat(Either.right(1).flatMap(x -> Either.right(x + 1))).isEqualTo(Either.right(2));
assertThat(Either.right(1).flatMap(x -> Either.left("Error"))).isEqualTo(Either.left("Error"));
assertThat(Either.left("Error").flatMap(x -> Either.right(x + 1))).isEqualTo(Either.left("Error"));
assertThat(Either.left("Error").flatMap(x -> Either.left("Error2"))).isEqualTo(Either.left("Error"));

assertThat(Try.success(1).flatMap(x -> Try.success(x + 1))).isEqualTo(Try.success(2));
assertThat(Try.success(1).flatMap(x -> Try.failure(new Error("Error"))).isFailure()).isTrue();
assertThat(Try.failure(new Error("Error")).flatMap(x -> Try.sucess(x + 1)).isFailure()).isTrue();
assertThat(Try.failure(new Error("Error")).flatMap(x -> Try.failure(new Error("Error2"))).isFailure()).isTrue();

assertThat(List.of(1,2,3).flatMap(x -> List.of(x, x))).isEqualTo(List.of(1,1,2,2,3,3,4,4));
```

The pseudocode of `flatMap` is

```java
public <M, A, B> M<B> flatMap(M<A> value, Function<A, M<B>> f)
```

## filter

A `filter` is a shortcut usage of `flatMap`, take `Option` for example
```java
public <A> Option<A> filter(Option<A> value, Function<A, boolean> f) {
	return value.flatMap(x -> {
		return f(x) ? Option.some(x) : Option.none();
	});
}
```
The different containers will have different behavior for `filter`,  for example

```java
assertThat(Option.some(1).filter(x -> x > 1)).isEqualTo(Option.none());
assertThat(Option.some(1).filter(x -> x < 2)).isEqualTo(Option.some(1));
assertThat(Option.none().filter(x -> x > 1)).isEqualTo(Option.none());
assertThat(Option.none().filter(x -> x < 2)).isEqualTo(Option.none());

assertThat(Either.right(1).filter(x -> x > 1)).isEqualTo(Option.some(Either.right(1)));
assertThat(Either.right(1).filter(x -> x < 2)).isEqualTo(Option.none());
assertThat(Either.left("Error").filter(x -> x > 1)).isEqualTo(Option.none());
assertThat(Either.left("Error").filter(x -> x < 2)).isEqualTo(Option.none());

assertThat(Try.success(1).filter(x -> x > 1).isFailure()).isTrue();
assertThat(Try.success(1).filter(x -> x < 2)).isEqualTo(Try.success(1));
assertThat(Try.failure(new Error("Error")).filter(x -> x > 1).isFailure()).isTrue();
assertThat(Try.failure(new Error("Error")).filter(x -> x < 2).isFailure()).isTrue();

assertThat(List.of(1,2,3).filter(x -> x > 1)).isEqualTo(List.of(2,3));
```

## foldLeft

A `foldLeft` is used to fold all the possible effects of the container into one value, for example

```java
assertThat(List.of(1,2,3).foldLeft(0, (acc, ele) -> acc + ele)).isEqualTo(6);
```

But `Option`, `Either`, and `Try` only have the `fold` function in Vavr

```java
assertThat(Option.some(1).fold(() -> "none", x -> x.toString())).isEqualTo("1");
assertThat(Option.none().fold(() -> "none", x -> x.toString())).isEqualTo("none");

assertThat(Either.right(1).<String>fold((e) -> "left", x -> x.toString())).isEqualTo("1");
assertThat(Either.left("Error").<String>fold((e) -> "left", x -> x.toString())).isEqualTo("left");

assertThat(Try.success(1).<String>fold((e) -> "failure", x -> x.toString())).isEqualTo("1");
assertThat(Try.failure(new Error("Error")).<String>fold((e) -> "failure", x -> x.toString()))
                .isEqualTo("failure");
```

But they actually can implement `foldLeft`, take `Option` for example

```java
public <A, B> B foldLeft(Option<A> option, B initial, Function2<B, A, B> f){
	return option.flatMap(x -> f(initial, x)).getOrElse(initial);
}

public <A, B> B fold(Option<A> option, B onNone, Function<A, B> onSome){
	return foldLeft(option, onNone, (acc, ele) -> onSome(ele));
}
```

the pseudocode of higher oder function `foldLeft` is
```java
public <M<_>, A, B> B foldLeft(M<A> value, B initial, Function2<B, A, B> f)
```

# How to unwrap a container easily?

To catch side-effects, we use an abstract interface as the parent type of all function effects, each effect is a child of the interface. For the consumer of the function, to know what's the effect returned, we need to find a way to identify the child of the interface, then take corresponding actions.

For Option, we can use `isSome` and `isNone`

```java
assertThat(Option.some(1).isSome()).isTrue();
assertThat(Option.some(1).isNone()).isFalse();
```

For Either, we can use `isRight` and `isLeft`

```java
assertThat(Either.right(1).isRight()).isTrue();
assertThat(Either.right(1).isLeft()).isFalse();
```
For Try, we can use `isSuccess` and `isFailue`

```java
assertThat(Try.success(1).isSuccess()).isTrue();
assertThat(Try.success(1).isFailure()).isFalse();
```

There will be lots of `if-else` in our code, like

```java
if(v.isSome()){
   ...
} else {
   ...
}
``` 

Vavr supply pattern matching to help us remove these boilerplates.

## Pattern Matching

Vavr borrows pattern matching from [Scala Pattern Matching](https://blog.shangjiaming.com/scala%20tutorial/pattern-match/), which is a very powerful tool.

We can use it to unwrap container

```java
@Test
public void canMatchOption() {
    Integer result =
            Match(Option.some(1)).of(Case($Some($()), x -> x + 10), Case($None(), 2));

    assertThat(result).isEqualTo(11);
}

@Test
public void canMatchEither() {

    Either<String, Integer> eitherValue = Either.right(1);

    String result = Match(eitherValue).of(Case($Right($()), x -> "right"),
            Case($Left($()), x -> "left"));

    assertThat(result).isEqualTo("right");

}

@Test
public void canMatchTry() {
    Try<Integer> tryValue = Try.success(1);

    String result = Match(tryValue).of(Case($Success($()), x -> "success"),
            Case($Failure($()), x -> "failure"));

    assertThat(result).isEqualTo("success");
}

@Test
public void canMatchList() {
    List<Integer> listValue = List.of(1,2,3);

    Integer result = Match(listValue).of(Case($Cons($(), $()), (head, tail) -> head));

    assertThat(result).isEqualTo(1);
}
```

We can check if the value meets the given condition

```java
@Test
public void canCheckValue() {
    Integer intValue = 2;
    String result = Match(intValue).of(Case($(x -> x > 0), x -> "positive"),
            Case($(x -> x < 0), x -> "negative"),
            Case($(x -> x == 0), x -> "zero"));

    assertThat(result).isEqualTo("positive");
}
```

We can also use it like `switch`

```java
@Test
public void canMatchLikeSwitch() {
    Integer intValue = 1;

    String result = Match(intValue).of(Case($(1), "1"), Case($(2), "2"), Case($(), "-1"));

    assertThat(result).isEqualTo("1");
}
```

# Summary

Vavr borrow lots of concepts from Scala, if you are interested in Functional Programming, please try [cats](https://typelevel.org/cats/) in Scala, it supplies more powerful features.