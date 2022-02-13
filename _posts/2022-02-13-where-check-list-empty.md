---
title: Where should we check if the list is empty?
tags:
- Scala
- Java
categories:
- Discussion
header:
  overlay_image: https://images.shangjiaming.com/jeremy-bezanger-EisZddhzxUw-unsplash.jpeg
  overlay_filter: 0.5
  caption: 'Photo credit: [**Unsplash**](https://unsplash.com)'
date: 2022-02-13 15:04 +0800
---
In 2022, I'm back to the Java tech stack(I should be crazy!).  There is a poetry in China: "Of Mountain Lu we cannot make out the true face, For we are lost in the heart of the very place". I believe I still love functional programming, but I need to jump out of it to understand it better.

In this blog, I want to share a scenario in our project to see if we can get a better solution.

# A problem

We have a function that returns a List of Panel

```java
public Optional<List<Panel>> generatePanels() {
    ...
    return panels;
}
```

In the controller, we will return 404 if the panels are empty.

```java
Optional<List<Panel>> panels = generatePanels();
if(!panels.filter(panelList -> !panelList.isEmpty()).isPresent()){
    throw new NotFoundError("There is no panel")
}
```

We have multiple consumers of `generatePanels` with similar business rules and the framework already returns 404 for Optional.

That means we need to write duplicated code.

# A solution

We can move the logic into the `generatePanels` function

```java

public Optional<List<Panel>> generatePanels() {
    ...
    return panels.filter(panelList -> !panelList.isEmpty());
}
```
Then all consumers don't need to care about this logic, we can return the Optional to framework directly.

But we have an implicit context here, `generatePanels` will return a non-empty list of panels if it is present. We can't prevent people from challenging the following code

```java
Optional<List<Panel>> panels = generatePanels();
Panel firstPanel;
if(panels.isPresent()){
    firstPanel = panels.get().get(0); // the list may be empty, this line will raise a bug
}
```

We definitely can add a test to ensure `generatePanels` to return a non-empty list or add a document to explain it, but people will forget or ignore them. Like speeding, we can warn people or formulate the law, but there are always people who died of it. 

# Better solution

A better solution for speeding is to add the restriction in the physical layer, making cars whose speed can not be higher than 60 km/h for example.

A better solution for the problem is to add the restriction in the compiler layer, returning a NonEmptyList. then we don't need to remember any implicit context or explain anything to the consumer, the signature of this function already tells you what they can do.

I will show it in Scala code

```scala
def Option[NonEmptyList[Panel]] generatePanels() {
    ...
    val panels: Option[List[Panel]] = ...
    panels.flatMap(x=> NonEmptyList.fromList(x))
}
```

Then we can get the first element safely

```scala
val panels: Option[NonEmptyList[Panel]] = generatePanels();
var firstPanel Panel;
if(panels.isSome()){
    firstPanel = panels.get().head;
}
```

# Summary

We should not only rely on people to remember the implicit context, we should utilize the compiler to help us. A correct function signature can save us from bugs and debug.

We always talk about side-effect in FP. In the above example, if we return a List, but our purpose is actually to return a non-empty List, then
for the consumer of this function, the non-empty logic is a side-effect, because they can't see it in the function signature. From this point of view, maybe we should allow the duplicated code, do the checking in every consumer.

What do you think about it?