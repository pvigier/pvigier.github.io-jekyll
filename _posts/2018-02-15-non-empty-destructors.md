---
layout: post
title: "Non empty destructors in C++"
date: 2018-02-09
author: pierre
tab: blog
comments: true
---
Have you already faced problems with non trivial destructors?

I face one recently which was really annoying. In this article, I want to share with you my knowledge of this problem and the solutions I use to address it.

# The problem

The problem is not really that the destructor is not empty but that the destructor is non trivial: there is a release of memory or some states are changed in another part of the app.

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

A `Segmentation fault` will occur.

Why?

Because when we the `main` function ends, the destructor of `A` is called to delete `a` and `anotherA`. When `a` is destroyed the memory cell to which `a`'s `mPointer` points to is freed. Then, when `anotherA` is destroyed, we try to free the memory to which `anotherA`'s `mPointer` points to. But as `anotherA` is a copy of `a`, its `mPointer` points to the same memory cell as `a`'s `mPointer`. Thus we try to free twice the same memory cell which causes the `Segmentation fault`.

So the problem is that because of copy the destructor is called twice on the same attributes.

Note that the copy or move constructor are often called when we use containers. For instance, there is a copy or a move when the `std::vector`'s `push_back` is called.

<!--more-->

# First solution: rule of three

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

The main disadvantage of this method is that we have to write a lot of code to obtain the correct behavior.

# Second solution: RAII

Another solution is to ensure the destructor is empty by, for instance, using only attributes that follows [RAII](https://en.wikipedia.org/wiki/Resource_acquisition_is_initialization).

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
	std::shared_ptr<int> mPointer;
}
```

The code is simpler, we have not to worry about the memory and the previous `main` executes successfully.

If adequate, I think that this solution should be chosen.

# Last solution: forbid copy and move

The last solution is a bit radical: it is to forbid copy and move.

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
    NonMovable(NonCopyable&&) = delete;
    NonCopyable& operator=(NonCopyable&&) = delete;
};
```

As copy and move is not sure for A, I would make A inheriting of both:

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

This time, If we try to compile the previous `main`, we would obtain a compile-time error. But if the program compile, we are ensured that no wild `Segmentation Fault` will appear because of a copy or a mode.

This solution has the benefit of being very fast to implement.

That's all for this post, I hope you find it useful.