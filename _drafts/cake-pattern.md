---
title: Cake Pattern
tags:
  - Scala
categories:
  - Scala Tutorial
---
There were some day to day pains for us in using the Cake Pattern. 
The obvious first one was the need for all the boilerplate `*Component` and `*ComponentImpl` code for every service and repository, 
which is tedious to write and read.

The second and bigger pain was that when wiring (cooking?) up all the layers of the cake 
in the controllers (or in controller or service unit tests), 
it was necessary to explicitly include not just the controller’s direct dependencies (typically just a single service), 
but rather all of the controller’s transitive dependencies.

I have a large codebase that I made into a cake years ago.
I'm gradually moving away from it but it's not so easy.
Here are some issues with it:1.
It's viral: the more things need to be in the cake, the more other things need to be as well.    
2.  Since your dependencies are not a list of imports but a composite self-type,
when you refactor and make something less coupled it's very hard to tell which dependencies you still need. 
As a result you often depend on more things than you need.
With parameters you can see what classes are imported and when you refactor the IDE can remove unused imports or the compiler can tell you about them.

The cake pattern is in theory pretty neat, being able to do DI without any libraries, however:*   It can make compile times a lot longer    
*   It can be hard to refactor as it is not immediately clear what layers of the cake need to be added or deleted when changing your dependency setup.    
*   It is hard to escape the cake. You can only solve dependencies by mixing them in, which cannot be done partially, as that would result in compile-errors. 

So in the end you need to build an enormous cake in your main to get it to work.


# Self Type Annotation

# Cake Pattern

# Module Pattern
