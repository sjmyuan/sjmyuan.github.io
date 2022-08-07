---
title: Do you trust the function return type?
excerpt: ''
tags:
- Discussion
- fun
date: 2022-08-08 00:34 +0800
---
Say we have a function to return the absolute value of an integer

```java
public Integer abs(Integer value) {
    return Math.abs(value);
}
```

And we have another function to get the element of a list by the absolute value of an integer

```java
public Integer getElement(Integer index, List<Integer> list) {

   Integer absIndex = abs(index);

   if(absIndex >= list.size()){
       return -1;
   }

   return list.get(absIndex);
}
```

The question is do we need to check if `abs(index) >= 0 `in `getElement`?

We may trust `abs` to return a non-negative integer because this is the requirement. But`abs` is implementing a logic different from the requirement. The requirement is that `abs` should return the absolute value of an integer that is non-negative, but it returns `Integer` which may be negative.

Considering the logic of `abs` is simple, which invokes `Math.abs` directly, we may have enough confidence to say it will return a non-negative integer even if its return type is `Integer`.

But what if the logic is complicated? it's hard to always remember the complicated logic to correct the wrong meaning of return type.

We can write tests to ensure the returned value meets the requirement, but we can't cover all the possible scenarios, for example, we can't test all integer values. if some people change the logic in the future, the test may still pass and the compiler won't blame the change, but there will be a runtime error.

So the function return type should meet the requirement, then the caller can trust it. If the function return type does not meet the requirement, we need to do an extra check.

For `abs`, the ideal solution is to return an unsigned integer. But there is no unsigned integer in Java, we have to return Integer, so we'd better check its return value.

I know this is not what we did for `Math.abs`, because we trust the JVM function. It's all about confidence, if our team has enough confidence, we can ignore the extra check.
