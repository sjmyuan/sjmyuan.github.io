---
title: Vavr Introduction
excerpt: ""
tags:
- Java
- fp
header:
  overlay_image: https://images.shangjiaming.com/alfons-morales-YLSwjSy7stw-unsplash.jpeg
  overlay_filter: 0.5
  caption: 'Photo credit: [**Unsplash**](https://unsplash.com)'
---

# What is Vavr?

[Vavr](https://www.vavr.io/) is a library for Functional Programming in Java. The first version, called _Javaslang_, was released on 19h march 2014.

It mainly contains three tings
1. Immutable collections to avoid side-effects
2. ADT(Algebraic data type) to wrap side-effects
3. Pattern matching to simplify the usage of ADT

I already created [vavr-examples](https://github.com/sjmyuan/vavr-examples), welcome to clone it and play the features.

# How to install it?

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

# What is Functional Programming?

[Functional Programming](https://blog.shangjiaming.com/scala%20tutorial/what-is-fp/) means construct program with pure functions.

A function is pure when
* For all input, same input produce same output
* No Side-Effect

# How to make a function pure?

## Avoid Side-Effect

Some times we will modify the parameter directly to return data, for example
```java
class IntOperation {
	public static void append(List<Integer> intList1, List<Integer> intList2){
		intList1.appendAll(intList2);
		return;
	}
}
List<Integer> intList1 = new LinkedList<Integer>();
intList1.add(1);
intList1.add(2);

List<Integer> intList2 = new LinkedList<Integer>();
intList2.add(3);
intList2.add(4);

IntOperation.append(intList1, intList2);

assertThan(intList1.size()).isEqualTo(4);

```

The function `append` is definitely not pure function, it return void but actually change the `intList1` outside of the function.

We can do this, because the `List` is mutable. To avoid the modification of the outside variable, we need to make data structure immutable.

Vavr redefine the common collection data structure to make them immutable.

### List

We can construct List by some util functions

```java
java.util.List<Integer> javaList = new java.util.LinkedList<Integer>()
javaList.add(1)
javaList.add(2)
javaList.add(3)

List.of(1,2,3)
List.ofAll(javaList)
```

The List is immutable, any changes to the list instance will create a new instance

```java
class IntOperation {
	public static void append(List<Integer> intList1, List<Integer> intList2){
		return intList1.appendAll(intList2);
	}
}
List<Integer> intList1 = List.of(1,2);

List<Integer> intList2 = List.of(3,4);

List<Integer> result = IntOperation.append(intList1, intList2);

assertThat(intList1.size()).isEqualTo(2);
assertThat(result.size()).isEqualTo(2);
```

We can also compare the elements of List easily
```java
asserThat(List.of(1,2,3).isEqualTo(List.of(1,2,3));
```

### Stream

## Wrap Side-Effect

### Option

It's very common to return `null` in Java, for example
```java
//TODO
```
But if one function may return null, the consumer of the function need always to check if the return value is null, that's why there are lots of ways to do this, like `@NotNull`, `assert(a != null)` and `Object.requireNotNull(a)`.

The reason is a non-primitive type always stands for two possible value, `null` or normal value, which is implicit and easy to forgot.

In Functional Programming, we'd like to ensure the non-primitive type only stands for the normal value, then it's very easy to know what's the return type of the function, don't need to guess if it will return null.

So `null` is a side-effect in Functional Programming, we need to find a way to return it obviously. Vavr defines Option to make the function returning `null` pure.

The above example can be changed to 

```java
//TODO
```

We can construct `Option` from normal value or `null`

```java
Option.some(1)
Option.none()
```

We can compare its value directly

```java
asserThat(Option.some(1)).isEqualTo(Option.some(1))
asserThant(Option.none()).isEqualTo(Option.none())
```

We can also un-wrap the value

```java
Option.some(1).get
Option.none().getOrNull
```

### Either

### Try

# What are the coding pattern?

## map

## flatMap

## filter

## foldLeft

# How to check wrapped value easily?

## Pattern Match
