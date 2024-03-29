---
layout: post
title: "Non Empty Destructors in C++"
date: 2018-02-15
author: pierre
tab: blog
tags: cpp
---
Have you already faced problems with nontrivial destructors?

I face one recently which was really annoying. In this article, I want to share with you my knowledge of this problem and the solutions I use to address it.

# The Problem

The problem is not really that the destructor is non empty but that the destructor is nontrivial: there is a release of memory or some states are changed in another part of the app.

Let us take a very simple example with a class that does dynamic allocation to explain the problem:

```cpp
class A
{
public:
    A() : mPointer(new int(0))
    {

    }

    ~A()
    {
        delete mPointer;
    }

private:
    int* mPointer;
}
```

As we allocate an integer in the constructor, the natural solution for memory management is to free it in the destructor. However, this will have terrible consequences.

For instance, if we do this:

```cpp
int main()
{
    A a;
    A anotherA(a);

    return 0;
}
```

A [segmentation fault](https://en.wikipedia.org/wiki/Segmentation_fault) will occur.

Why?

Because when the `main` function ends, the destructor of `A` is called to delete `a` and `anotherA`. When `a` is destroyed the memory cell to which the `mPointer` of `a` points to is freed. Then, when `anotherA` is destroyed, we try to free the memory to which the `mPointer` of `anotherA` points to. But as `anotherA` is a copy of `a`, its `mPointer` points to the same memory cell as that of `a`. Thus, we try to free twice the same memory cell which causes the `Segmentation fault`.

So, the problem is that because of the copy the destructor is called twice on the same attributes.

Note that the copy or move constructors are often called when we use containers. For instance, there is a copy or a move when the `std::vector` `push_back` is called.

<!--more-->

# First Solution: Rule of Three

Firstly, this rule has nothing to do with Star Wars' [Rule of two](http://starwars.wikia.com/wiki/Rule_of_Two).

The [rule of three](https://en.wikipedia.org/wiki/Rule_of_three_(C%2B%2B_programming)) is a design rule that say that if one of the following is defined then the two others should also be defined:

* destructor
* copy constructor
* copy assignment operator

In particular, if the destructor is non empty then we should define the copy constructor and the copy assignment operator.

So for our previous class `A`, we would do the following:

```cpp
class A
{
public:
    A() : mPointer(new int(0))
    {

    }

    // Copy constructor
    A (const A& other) : mPointer(new int(other.mPointer))
    {

    }

    ~A()
    {
        delete mPointer;
    }

    // Copy assignment operator
    A& operator=(const A& other)
    {
        delete mPointer;
        mPointer = new int(other.mPointer);
        return *this;
    }

private:
    int* mPointer;
}
```

With this definition of A, there is no problem anymore with our previous `main`.

With C++11 the rule of three becomes the rule of five as we should also define the move constructor and the move assignment operator but I omit that for the sake of simplicity and brevity.

The main disadvantage of this method is that we have to write a lot of code to obtain the correct behavior. But this is the idiomatic way to encapsulate a low-level resource and follow [RAII](https://en.wikipedia.org/wiki/Resource_acquisition_is_initialization).

# Second Solution: RAII

Another solution is to ensure the destructor is empty by, for instance, using only attributes that follows RAII (standard containers, `std::string`s, smart pointers, a file stream, etc.).

In our case, for `A`, we could use a smart pointer instead of a plain one:

```cpp
#include <memory>

class A
{
public:
    A() : mPointer(new int(0))
    {

    }

private:
    std::unique_ptr<int> mPointer;
}
```

The code is simpler, we have not to worry about the memory and the previous `main` executes successfully.

If adequate, this is the solution that should be chosen.

# Third Solution: Forbid Copy and Move

The third solution is a bit radical: it is to forbid copy and move.

To do that I use the two following class:

```cpp
class NonCopyable
{
public:
    NonCopyable() = default;
    NonCopyable(const NonCopyable&) = delete;
    NonCopyable& operator=(const NonCopyable&) = delete;
};
```

```cpp
class NonMovable
{
public:
    NonMovable() = default;
    NonMovable(NonMovable&&) = delete;
    NonMovable& operator=(NonMovable&&) = delete;
};
```

As copy and move are not safe for A, I would make A inherit from both:

```cpp
class A : public NonCopyable, public NonMovable
{
public:
    A() : mPointer(new int(0))
    {

    }

    ~A()
    {
        delete mPointer;
    }

private:
    int* mPointer;
}
```

This time, if we try to compile the previous `main`, we would obtain a compile-time error. But if the program compile, we are ensured that no wild segmentation fault will occur during execution because of a copy or a move.

This solution has the benefit of being very fast to implement.

# Fourth Solution: Set Up and Tear Down

The last solution is to manage the initialization and the finalization outside of the constructor and the destructor.

In our example, we could use a method `setUp` to allocate the memory and `tearDown` to release it:

```cpp
class A
{
public:
    A() : mPointer(nullptr)
    {

    }

    ~A()
    {

    }

    void setUp()
    {
        mPointer = new int(0)
    }

    void tearDown()
    {
        delete mPointer;
        mPointer = nullptr;
    }

private:
    int* mPointer;
}
```

Then we can transform the previous `main` to obtain the correct behavior:

```cpp
int main()
{
    A a;
    a.setUp();
    A anotherA(a);
    a.tearDown();

    return 0;
}
```

This solution works well but requires from the user of the class to be more careful and to manage himself the memory. Moreover, this is non-idiomatic and bad C++. It is better to follow the rule of three/five to encapsulate the resource.

That's all for this post, I hope you find it useful.
