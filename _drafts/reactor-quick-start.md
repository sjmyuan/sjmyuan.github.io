---
title: Reactor Quick Start
tags:
- Java
categories:
- Reactor
header:
  overlay_image: TODO
  overlay_filter: 0.5
  caption: 'Photo credit: [**Unsplash**](https://unsplash.com)'
---

> [Reactor](link) is a fully non-blocking reactive programming foundation for the JVM, with efficient demand management (in the form of managing “backpressure”).

Seems it's very hard, but we can only use 20% of its feature to implement almost 80% work.

In my current project, we don't touch `backpressure` any more, most of the operations we used are

* (Flux|Mono).just
* (Flux|Mono).map
* (Flux|Mono).flatMap
* (Flux|Mono).onErrorResume
* (Flux|Mono).filter

So don't be scared about it, after reading this blog, you can also use it easily in you project.

# How to install?

Add the following configuration to your `pom.xml`. we use `bom` here, so we don't need to care about the version and dependency consistency.

```xml
<dependencyManagement> 
    <dependencies>
        <dependency>
            <groupId>io.projectreactor</groupId>
            <artifactId>reactor-bom</artifactId>
            <version>2020.0.16</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>io.projectreactor</groupId>
        <artifactId>reactor-core</artifactId> 
    </dependency>
    <dependency>
        <groupId>io.projectreactor</groupId>
        <artifactId>reactor-test</artifactId> 
        <scope>test</scope>
    </dependency>
</dependencies>
```

# How to do test?

Reactor supplied a test library to help us to test Flux|Mono, we can use `StepVerifier` in this library to 

* Verify the elements and their order

  ```java
    StepVerifier.create(Flux.fromArray(new Integer[] {1, 2, 3, 4, 5})).expectNext(1)
        .expectNext(2).expectNext(3).expectNext(4).expectNext(5).verifyComplete();
  ```

* Verify error 
   ```java
    StepVerifier.create(Flux.error(new Exception("some error")))
            .verifyErrorMessage("some error");
   ```
* Verify complete 

  ```java
    StepVerifier.create(Flux.empty()).verifyComplete();
  ```

In the following sections, we will use the `StepVerifier` to verify the examples.

# What's the basic concept we need to know?

There are two basic concepts in reactor

* Flux
  A sequence of data, there are 0 or more elements

* Mono
  A sequence of data, there is 0 or 1 element

# What we need to know to do most of work?

In this section, I will use some concept to group the knowledge

* Boxing
  Put the data into Flux|Mono

* Transforming
  Transform the current data to another data

* Peeking
  Do something for each element but not modify the data

* Unboxing
  Get the data out of Flux|Mono

## Boxing

### How to box String?

```java
    @Test
    public void canBeCreatedFromString() {
        StepVerifier.create(Flux.just("hello world")).expectNext("hello world").verifyComplete();
    }
```

```java
    @Test
    public void canBeCreatedFromString() { //TODO add test in mono example
        StepVerifier.create(Mono.just("hello world")).expectNext("hello world").verifyComplete();
    }
```
### How to box Number?

```java
    @Test
    public void canBeCreatedFromNumber() {
        StepVerifier.create(Flux.just(1)).expectNext(1).verifyComplete();
        StepVerifier.create(Flux.just(1.0)).expectNext(1.0).verifyComplete();
    }
```

```java
    @Test
    public void canBeCreatedFromNumber() { //TODO add test in mono example
        StepVerifier.create(Mono.just(1)).expectNext(1).verifyComplete();
        StepVerifier.create(Mono.just(1.0)).expectNext(1.0).verifyComplete();
    }
```
### How to box Optional or Nullable value?

```java
    @Test
    public void canBeCreatedFromNullableValue() {
        String value = null;
        StepVerifier.create(Mono.justOrEmpty(value)).verifyComplete();

        value = "hello world";
        StepVerifier.create(Mono.justOrEmpty(value)).expectNext("hello world").verifyComplete();
    }

    @Test
    public void canBeCreatedFromOptionalValue() {

        Optional<String> value = Optional.empty();
        StepVerifier.create(Mono.justOrEmpty(value)).verifyComplete();

        value = Optional.of("hello world");
        StepVerifier.create(Mono.justOrEmpty(value)).expectNext("hello world").verifyComplete();
    }
```

### How to box data generator?
```java
    @Test
    public void canBeCreatedFromCallable() {
        StepVerifier.create(Mono.fromCallable(() -> "hello world!")).expectNext("hello world!")
                .verifyComplete();
    }
```
### How to box Array?
```java
    @Test
    public void canBeCreatedFromArray() {
        StepVerifier.create(Flux.fromArray(new Integer[] {1, 2, 3, 4, 5})).expectNext(1)
                .expectNext(2).expectNext(3).expectNext(4).expectNext(5).verifyComplete();
    }
```
### How to box Iterable?
```java
    @Test
    public void canBeCreatedFromList() {
        List<Integer> list = new LinkedList<Integer>();
        list.add(1);
        list.add(2);
        list.add(3);
        list.add(4);

        StepVerifier.create(Flux.fromIterable(list)).expectNext(1).expectNext(2).expectNext(3)
                .expectNext(4).verifyComplete();
    }

    @Test
    public void canBeCreatedFromSet() {
        Set<Integer> set = new HashSet<Integer>();
        set.add(1);
        set.add(2);
        set.add(2);
        set.add(3);
        set.add(4);

        StepVerifier.create(Flux.fromIterable(set)).expectNext(1).expectNext(2).expectNext(3)
                .expectNext(4).verifyComplete();
    }
```
//TODO check if mono can work
### How to box Stream?

```java
    @Test
    public void canBeCreatedFromStream() {
        Stream<Integer> stream = Stream.of(1, 2, 3, 4, 5);
        StepVerifier.create(Flux.fromStream(stream)).expectNext(1).expectNext(2).expectNext(3)
                .expectNext(4).expectNext(5).verifyComplete();
    }
```

//TODO check if mono can work

### How to box Throwable?

```java
    @Test
    public void canBeCreatedFromThrowable() {
        StepVerifier.create(Flux.error(new Exception("some error")))
                .verifyErrorMessage("some error");
    }
```
//TODO mono

## Transforming

### How to filter element by some condition?
```java
    @Test
    public void canDoFilter() {
        Flux<Integer> flux = Flux.just(1, 2, 3, 4, 5, 6);
        StepVerifier.create(flux.filter(x -> x > 5)).expectNext(6).verifyComplete();
    }
```
//TODO mono

### How to apply a function A -> B?
//TODO more explanation
```java
    @Test
    public void canMapTheTypeOfValueInSequenceToAnotherType() {
        StepVerifier.create(flux.map(x -> x.toString())).expectNext("1").expectNext("2")
                .expectNext("3").expectNext("4").expectNext("5").expectNext("6").verifyComplete();
    }

    @Test
    public void canMapTheValueInSequenceToAnotherValue() {
        StepVerifier.create(flux.map(x -> x + 1)).expectNext(2).expectNext(3).expectNext(4)
                .expectNext(5).expectNext(6).expectNext(7).verifyComplete();
    }
```
//TODO mono
### How to apply a function A -> (Flux|Mono)<B>?
//TODO more explanation
```java
    @Test
    public void canMapTheValueInSequenceToAnotherSequence() {

        StepVerifier.create(flux.flatMap(x -> Flux.just(x, x))).expectNext(1).expectNext(1)
                .expectNext(2).expectNext(2).expectNext(3).expectNext(3).expectNext(4).expectNext(4)
                .expectNext(5).expectNext(5).expectNext(6).expectNext(6).verifyComplete();

        StepVerifier.create(flux.flatMap(x -> Mono.just(x))).expectNext(1).expectNext(2)
                .expectNext(3).expectNext(4).expectNext(5).expectNext(6).verifyComplete();
    }
```
//TODO mono
### How to give a default value if there is no data?
```java
    @Test
    public void canRecoverWithSingleDefaultValueFromEmptySequence() {
        StepVerifier.<Integer>create(Flux.<Integer>empty().defaultIfEmpty(1)).expectNext(1)
                .verifyComplete();
    }
```
//TODO mono
### How to replace with another Flux|Mono if there is no data?

```java
    @Test
    public void canRecoverWithAnotherSequenceFromEmptySequence() {
        StepVerifier.<Integer>create(Flux.<Integer>empty().switchIfEmpty(Flux.just(1)))
                .expectNext(1).verifyComplete();
    }
```
//TODO mono

## Peeking

### How to print log for each element?

```java
    @Test
    public void canDoSomethingForEveryElement() {
        Flux<Integer> flux = Flux.just(1, 2, 3, 4, 5, 6);
        flux.doOnNext(x -> System.out.println(x)).collectList().block(); // TODO check the println command
        assertThat(list.size()).isEqualTo(6);
    }
```
//TODO Mono

## Unboxing

### How to convert Flux to a List?

```java
    @Test
    public void canBeConvertedToList() {
        List<Integer> list = Flux.just(1, 2, 3).collectList().block();
        assertThat(list.size()).isEqualTo(3);
        assertThat(list.get(0)).isEqualTo(1);
        assertThat(list.get(1)).isEqualTo(2);
        assertThat(list.get(2)).isEqualTo(3);
    }
```

### How to get data out of Mono?

```java
//TODO
```

# Summary

I created a repo [reactor-examples](fix-url) to play the above examples, but there are more.

Clone the repo and make you hand dirty!
```sh
git clone <fix-url>
```

Welcome to raise PR to add more examples.