---
title: List and Option
excerpt: ''
tags:
- Java
- fp
header:
  overlay_image: https://images.shangjiaming.com/martin-sanchez-ycG0A6DlvOk-unsplash.jpeg
  overlay_filter: 0.5
  caption: 'Photo credit: [**Unsplash**](https://unsplash.com)'
date: 2022-07-07 21:46 +0800
---
We already introduced Vavr in [Vavr Introduction](https://blog.shangjiaming.com/vavr-introduction/). In my daily work, there are lots of scenarios to use `List` and `Option` together, we will talk about some common scenarios in this post.

# List to Option

Sometimes, we'd like to get the first element of `List`

```java
List<Integer> list = List.of(1,2,3);
Integer head = list.get(0); // head = 1
```

But the `List` may be empty, then we have to check the size of `List` before getting the first element

```java
Integer head = 0; // default value
if(list.size() > 0) {
  head = list.get(0); // head = 1
}
```

In the above code, we set the default value of head to be 0. What if we don't know the default value when we get the first element? 

We can pass the `List` until we know the default value, or give a special default value first, then replace it with a reasonable default value. For example

```java
public Integer processHead(List<Integer> list) {
  Integer head = 2; // reasonable default value
  if(list.size() > 0){
    head = list.get(0) + 1;
  }
  return head;
}

public Integer processHead(Integer head) {
  if(head == -1) { // special default value
    return 2; // reasonable default value
  }

  return head + 1;
}
```

To avoid the size checking and special default value, we can use `List::headOption` to get the first element

```java
List<Integer> list = List.of(1,2,3);
Option<Integer> head = list.headOption(); // Option.Some(1)

public Integer processHead(Option<Integer> head) {
  return head.map(x -> x + 1).getOrElse(2);
}
```

`List::headOption` will return `Option.Some` if `List.size() > 0`, return `Option.None` if `List.size() == 0`. Then`Option` allow us to give a reasonable default value by `getOrElse` until we finish all the operations.

# Option to List

Say we have a function `sum` to calculate the sum of `List<Integer>`. What if we only have an `Option` now? Could we convert an `Option` to `List`?

We may implement it like this

```java
public Integer sum(List<Integer> list) {
   return list.fold(0, (acc, ele) -> acc + ele); 
}

Option<Integer> option = Option.some(1);

List<Integer> convertedList = option.isDefined() ? List.of(option.get()) : List.empty(); // List(1)

Integer sumValue = sum(convertedList); // 1
```

Vavr supplies an easier way to do this

```java
Option<Int> option = Option.some(1);

Int sumValue = sum(List.ofAll(option)); // 1
```

`List::ofAll` can convert any `Iterable` to List, luckily`Option` is a child of `Iterable`.

# List<Option> to List

Say we have a list of `Option<Integer>`, we need to filter out `None` and get the `Integer` out of `Option`. For example

```java
//current
List<Option<Integer>> currentList = List.of(Option.some(1), Option.none(), Option.some(2), Option.none());

//expected
List<Integer> expectedList = List.of(1, 2);
```

The obvious implementation is

```java
List<Integer> expectedList = currentList.filter(Option::isDefined).map(Option::get); // List(1, 2)
```

It can work, but we have to use `filter` and `map` always together. And the `Option::get` may throw exceptions, if we add some operations between `filter` and `map` in the future, there may be an exception thrown.

The ideal way is to perform `filter` and `map` as an atomic operation. `flatMap` can achieve this.

```java
List<Integer> expectedList = currentList.flatMap(x -> x); // List(1, 2)
```

Why can it work? Vavr extends the implementation of `flatMap`, it can accept a function `A -> Iterable<B>`, and `Option` is a child of `Iterable`.

```java
default <U> List<U> flatMap(Function<? super T, ? extends Iterable<? extends U>> mapper)
```

We can not only convert `List<Option<A>> `to `List<A>`, but also apply a function `A -> Option<B>` to `List<A>`directly, for example

```java
List<Integer> list = List.of(1, 2, 3, 4);

public Option<Integer> isOdd(Integer v) {
   if(v % 2 == 0){
      return Option.none();
   }
   return Option.some(v);
}

List<Integer> oddList = list.flatMap(this::isOdd); // List(1, 3)
```

Not all the functional programming libraries support this, like [cats](https://typelevel.org/cats/). Because it doesn't follow the definition of Monad strictly. But it's very convenient in practice.

# List<Option> to Option<List>

Say we have a list of id, we require to fetch the user name by id, then return all the names or nothing if there is any error.

```java
public Option<String> fetchName(Integer id) {
   return Option.some("name" + id);
}

List<Integer> ids = List.of(1, 2, 3, 4, 5, 6);

Option<List<String>> names = ids.map(this::fetchName); // compile error, can't assign List<Option<String>> to Option<List<String>>
```

How could we switch the position of `Option` and `List`? We can do it like this

```java
Option<List<String>> names = 
   ids.map(this::fetchName).foldLeft(Option.some(List.empty()), (acc, ele) -> {
      if(ele.isEmpty() || acc.isEmpty()){
         return Option.none();
      }

      return acc.map(list -> list.append(ele.get()));
   });
```

But it will be easier to use`Option::traverse` and `Option::sequence` here.

```java
Option<List<String>> names = Option.traverse(ids, this::fetchName);

Option<List<String>> names = Option.sequence(ids.map(this::fetchName));
```

# Summary

We talked about the common scenarios to use `List` and `Option` together, these scenarios can also work for `Either` and `Try`, feel free to clone [vavr-examples](https://github.com/sjmyuan/vavr-examples) to try them.
