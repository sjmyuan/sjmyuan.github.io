---
title: Reactor Quick Start
tags:
  - Java
categories:
  - Reactor
header:
  overlay_image: https://images.shangjiaming.com/vitaliy-paykov-yDDnNsKep2g-unsplash.jpeg
  overlay_filter: 0.5
  caption: "Photo credit: [**Unsplash**](https://unsplash.com)"
---

> [Reactor](https://projectreactor.io/) is a fully non-blocking reactive programming foundation for the JVM, with efficient demand management (in the form of managing “backpressure”).

Seems it's very hard, but we can only use 20% of its feature to implement almost 80% work.

In my current project, we don't touch `backpressure` any more, most of the operations we used are

- (Flux|Mono).just
- (Flux|Mono).map
- (Flux|Mono).flatMap
- (Flux|Mono).onErrorResume
- (Flux|Mono).filter

So don't be scared about it, after reading this blog, you can also use it easily in you project.

# How to install?

Add the following configuration to your `pom.xml`. we use [BOM](https://www.baeldung.com/spring-maven-bom) here, so we don't need to care about the version and dependency consistency.

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

- Verify elements and their order

  ```java
    StepVerifier.create(Flux.fromArray(new Integer[] {1, 2, 3, 4, 5})).expectNext(1)
        .expectNext(2).expectNext(3).expectNext(4).expectNext(5).verifyComplete();
  ```

- Verify error

  ```java
   StepVerifier.create(Flux.error(new Exception("some error")))
           .verifyErrorMessage("some error");
  ```

- Verify end of sequence

  ```java
    StepVerifier.create(Flux.empty()).verifyComplete();
  ```

In the following sections, we will use the `StepVerifier` to verify examples.

# What's the basic concept we need to know?

There are two basic concepts in reactor

- Flux

  A sequence of data, there are 0 or more elements

- Mono

  A sequence of data, there is 0 or 1 element

# What we need to know to do most of the work?

In this section, I will borrow some concepts to group the examples

- Boxing

  Put the data into Flux|Mono

- Transforming

  Transform the current data to another data

- Peeking

  Do something for each element but not modify the data

- Unboxing

  Get the data out of Flux|Mono

## Boxing

### How to box String?

- Flux

    ```java
        @Test
        public void canBeCreatedFromString() {
            StepVerifier.create(Flux.just("hello world")).expectNext("hello world").verifyComplete();
        }
    ```

- Mono

    ```java
        @Test
        public void canBeCreatedFromString() { //TODO add test in mono example
            StepVerifier.create(Mono.just("hello world")).expectNext("hello world").verifyComplete();
        }
    ```

### How to box Number?

- Flux

    ```java
        @Test
        public void canBeCreatedFromNumber() {
            StepVerifier.create(Flux.just(1)).expectNext(1).verifyComplete();
            StepVerifier.create(Flux.just(1.0)).expectNext(1.0).verifyComplete();
        }
    ```

- Mono

    ```java
        @Test
        public void canBeCreatedFromNumber() { //TODO add test in mono example
            StepVerifier.create(Mono.just(1)).expectNext(1).verifyComplete();
            StepVerifier.create(Mono.just(1.0)).expectNext(1.0).verifyComplete();
        }
    ```

### How to box Optional or Nullable value?

This can only be done by Mono

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

Data generator here is a function without parameter, its signature looks like () -> A.

This can only be done by Mono.

```java
    @Test
    public void canBeCreatedFromCallable() {
        StepVerifier.create(Mono.fromCallable(() -> "hello world!")).expectNext("hello world!")
                .verifyComplete();
    }
```

### How to box Array?

This can only be done by Flux.

```java
    @Test
    public void canBeCreatedFromArray() {
        StepVerifier.create(Flux.fromArray(new Integer[] {1, 2, 3, 4, 5})).expectNext(1)
                .expectNext(2).expectNext(3).expectNext(4).expectNext(5).verifyComplete();
    }
```

### How to box Iterable?

This can only be done by Flux.

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

### How to box Stream?

This can only be done by Flux.

```java
    @Test
    public void canBeCreatedFromStream() {
        Stream<Integer> stream = Stream.of(1, 2, 3, 4, 5);
        StepVerifier.create(Flux.fromStream(stream)).expectNext(1).expectNext(2).expectNext(3)
                .expectNext(4).expectNext(5).verifyComplete();
    }
```

### How to box Throwable?

- Flux

    ```java
        @Test
        public void canBeCreatedFromThrowable() {
            StepVerifier.create(Flux.error(new Exception("some error")))
                    .verifyErrorMessage("some error");
        }
    ```

- Mono

    ```java
        @Test
        public void canBeCreatedFromThrowable() {
            StepVerifier.create(Mono.error(new Exception("some error")))
                    .verifyErrorMessage("some error");
        }
    ```

## Transforming

### How to filter element by some condition?

- Flux

    ```java
        @Test
        public void canDoFilter() {
            Flux<Integer> flux = Flux.just(1, 2, 3, 4, 5, 6);
            StepVerifier.create(flux.filter(x -> x > 5)).expectNext(6).verifyComplete();
        }
    ```

- Mono

    ```java
        @Test
        public void canDoFilter() {
            StepVerifier.create(Mono.just(6).filter(x -> x > 5)).expectNext(6).verifyComplete();
            StepVerifier.create(Mono.just(2).filter(x -> x > 5)).verifyComplete();
        }
    ```

### How to apply a function A -> B?

If there is a function A -> B, A is the element type of Flux|Mono and B is not Flux|Mono, we can use `(Flux|Mono).map` to apply the function to each element.

- Flux

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

- Mono

    ```java
        @Test
        public void canMapTheTypeOfValueInSequenceToAnotherType() {
            StepVerifier.create(Mono.just(1).map(x -> x.toString())).expectNext("1").verifyComplete();
        }

        @Test
        public void canMapTheValueInSequenceToAnotherValue() {
            StepVerifier.create(Mono.just(1).map(x -> x + 1)).expectNext(2).verifyComplete();
        }
    ```

### How to apply a function A -> (Flux|Mono)<B>?

If there is a function A -> (Flux|Mono)<B>, A is the element type of Flux|Mono, we can use `(Flux|Mono).flatMap` or `Mono.flatMapMany` to apply the function to each element.

- Flux

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

- Mono

    ```java
        @Test
        public void canMapTheValueInSequenceToAnotherSequence() {

            StepVerifier.create(Mono.just(1).flatMapMany(x -> Flux.just(x, x))).expectNext(1)
                    .expectNext(1).verifyComplete();

            StepVerifier.create(Mono.just(1).flatMap(x -> Mono.just(x + 1))).expectNext(2)
                    .verifyComplete();
        }
    ```

### How to give a default value if there is no data?

- Flux

    ```java
        @Test
        public void canRecoverWithSingleDefaultValueFromEmptySequence() {
            StepVerifier.<Integer>create(Flux.<Integer>empty().defaultIfEmpty(1)).expectNext(1)
                    .verifyComplete();
        }
    ```

- Mono

    ```java
        @Test
        public void canRecoverWithSingleDefaultValueFromEmptySequence() {
            StepVerifier.<Integer>create(Mono.<Integer>empty().defaultIfEmpty(1)).expectNext(1)
                    .verifyComplete();
        }
    ```

### How to replace with another Flux|Mono if there is no data?

- Flux

    ```java
        @Test
        public void canRecoverWithAnotherSequenceFromEmptySequence() {
            StepVerifier.<Integer>create(Flux.<Integer>empty().switchIfEmpty(Flux.just(1)))
                    .expectNext(1).verifyComplete();
        }
    ```

- Mono

    ```java
        @Test
        public void canRecoverWithAnotherSequenceFromEmptySequence() {
            StepVerifier.<Integer>create(Mono.<Integer>empty().switchIfEmpty(Mono.just(1)))
                    .expectNext(1).verifyComplete();
        }
    ```

## Peeking

### How to print log for each element?

We can use `(Flux|Mono).doOnNext` to peek the value of each element but not modify its value.

- Flux

    ```java
        @Test
        public void canDoSomethingForEveryElement() {
            Flux<Integer> flux = Flux.just(1, 2, 3, 4, 5, 6);
            flux.doOnNext(x -> System.out.println(x)).collectList().block();
        }
    ```

- Mono

    ```java
        @Test
        public void canDoSomethingForEveryElement() {
            Mono.just(1).doOnNext(x -> System.out.println(x)).block();
        }
    ```

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
    @Test
    public void canBeConvertedToValue() {
        assertThat(Mono.just(1).block()).isEqualTo(1);
        assertThat(Mono.empty().block()).isNull();
    }
```

# Summary

I created a repo [reactor-examples](https://github.com/sjmyuan/reactor-examples) to play the above examples.

Clone the repo and get your hand dirty!

```sh
git clone git@github.com:sjmyuan/reactor-examples.git
```

Welcome to raise PR to add more examples.
