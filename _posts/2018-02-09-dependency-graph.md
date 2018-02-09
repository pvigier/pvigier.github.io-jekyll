---
layout: post
title: "Circular dependencies in C++"
date: 2018-02-09
author: pierre
tab: blog
comments: true
---
Hi guys, it has been a while since the last post.

I write this short post to tell you about a small script I coded recently. You can find it [here](https://github.com/pvigier/dependency-graph) on my github account.

Its goal is to draw the "include" dependencies between classes in a C++ project. In particular, it allows to detect circular dependencies very easily or to check the architecture of a project.

You can see the output of the script on a project of mine:

![Dependency graph](https://github.com/pvigier/dependency-graph/raw/master/examples/example1.png){: .center-image .modal-image }

I really like this visual representation which allows to see how classes interact.

However, the true reason why I created this tool is not because I like to see beautiful graphs but because I hate circular dependencies (note that there is none in the graph above). I consider circular dependencies as design flaws. But sometimes in a large project, it could happen that accidentally I create circular dependencies ...

<!--more-->

## Circular dependencies

Fistly, let us be clear about what is a circular dependencies.

Suppose that you have two classes A and B. If A uses B and conversely then there is a circular dependency. However, the circular dependency maybe more subtle. For instance, it may be A that uses B that uses C that uses A.

In C++, if a file "A.h" includes "B.h" then "B.h" can not include "A.h". The only way for B to use A is to forward declare A, use pointers or references on A in the header and finally include "A.h" in "B.cpp".

For example, these three files should compile successfully.

A.h:
```cpp
#pragma once

#include "B.h"

class A
{
private:
    B mB;
};
```

B.h:
```cpp
#pragma once

class A; // forward declaration

class B
{
private:
    A* mA;
};
```

B.cpp:
```cpp
#include "B.h"
#include "A.h" // We include "A.h" in the cpp file
```

This works well and is often the simpler solution for small projects.

Why so much hate toward circular dependencies after all?

I will list several reasons against circular dependencies:
* Compilation time can blow up: when a file in the cycle is changed, all the other files have to be recompiled.
* Prone to errors: as several pieces of code are tightly coupled, a change in one will probably break another.
* Harder to reuse the code: as many files are dependent on each other, if you want to reuse a file in another project, you must also take the other ones.
* Harder to debug: as many pieces of codes are coupled, you will have to look at a lot of files if you want to trace back a bug.

There are many ways to avoid circular dependencies. The most obvious one is to design well the project with independent modules or even to break a big project in smaller libraries.

Design patterns like [Observer](http://gameprogrammingpatterns.com/observer.html) or [Event Queue](http://gameprogrammingpatterns.com/event-queue.html) can also be of great help. By the way, I recommend you to have a look at [Game Programming Patterns](http://gameprogrammingpatterns.com/) by Robert Nystrom if you do not know this book already.

Finally, you could use my little script to monitor the architecture of your project.

## Some details on the code

The script is pretty simple. I use the `re` module of Python to find the `#include`'s in source files. 

Then the scripts creates a graph and uses a port to Python of [graphviz](https://www.graphviz.org/) to draw the graph. Graphviz creates very beautiful, this amazing tool is also used, for instance, by Doxygen to generate its diagram.

That's all for this short post. See you!
