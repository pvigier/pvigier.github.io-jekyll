---
layout: post
title: "Quadtree and Collision Detection"
date: 2019-08-04
author: pierre
tab: blog
tags: vagabond game-engine cpp
redirect_from:
  - /2019/07/28/quadtree-collision-detection.html
---
This week was short as I was still working on the [2D light system]({{ site.baseurl }}{% post_url 2019-07-28-vagabond-2d-light-system %}) on Monday and Tuesday. I spent the rest of my time working on a [quadtree](https://en.wikipedia.org/wiki/Quadtree) implementation.

In this article, I will share with you my implementation and my thoughts while designing it.

Firstly, I want to argue why I decided to implement a quadtree.

A Quadtree is a [space partitioning data structure](https://en.wikipedia.org/wiki/Space_partitioning). Its main advantage against other data structures is its versatility. It offers good performance for insertion, removal and lookup. Thus, we can use it in a dynamic context where the data often change. Moreover, it is quite easy to understand and implement.

If you are totally new to space partitioning, I advise you to read this [article](http://gameprogrammingpatterns.com/spatial-partition.html) by Robert Nystrom. If you want a gentler introduction to quadtrees, you can read this [article](https://gamedevelopment.tutsplus.com/tutorials/quick-tip-use-quadtrees-to-detect-likely-collisions-in-2d-space--gamedev-374) or this [one](https://jimkang.com/quadtreevis/).

In my game, there are several places where using a quadtree is an instant win:

* During collision detection, using a quadtree is way more efficient than the brute-force approach (testing all pairs). It is not the most efficient approach though, see this [article](https://0fps.net/2015/01/23/collision-detection-part-3-benchmarks/) if you want an overview of possible approaches and benchmarks. But, I will start using this in the first version of my physics engine. I may change later and use a more specialized algorithm if I need to.
* In the [scene graph]({{ site.baseurl }}{% post_url 2019-07-14-vagabond-game-engine-foundations %}), to perform culling, I can use a quadtree to find all the nodes that are visible.
* In the [light system]({{ site.baseurl }}{% post_url 2019-07-28-vagabond-2d-light-system %}), I can use a quadtree to find the walls that are intersecting the light visibility polygon.
* In an AI system, I can use a quadtree to find all the objects or enemies that are close to an entity.
* etc.

As you can notice, quadtrees are very versatile. They are a nice tool to have in your toolbox.

You can find all the code that I will show next on [GitHub](https://github.com/pvigier/Quadtree).

<!--more-->

# Preliminaries

Before to detail the code for the quadtree, we need small classes for geometric primitives: a `Vector2` class to represent points and a `Box` class to represent boxes. Both will be templated.

## Vector2

The [`Vector2`](https://github.com/pvigier/Quadtree/blob/master/include/Vector2.h) class is a minimalist class. It only contains constructors and, `+` and `/` operators. That is all we will need:

```cpp
template<typename T>
class Vector2
{
public:
    T x;
    T y;

    constexpr Vector2<T>(T X = 0, T Y = 0) noexcept : x(X), y(Y)
    {

    }

    constexpr Vector2<T>& operator+=(const Vector2<T>& other) noexcept
    {
        x += other.x;
        y += other.y;
        return *this;
    }

    constexpr Vector2<T>& operator/=(T t) noexcept
    {
        x /= t;
        y /= t;
        return *this;
    }
};

template<typename T>
constexpr Vector2<T> operator+(Vector2<T> lhs, const Vector2<T>& rhs) noexcept
{
    lhs += rhs;
    return lhs;
}

template<typename T>
constexpr Vector2<T> operator/(Vector2<T> vec, T t) noexcept
{
    vec /= t;
    return vec;
}
```

## Box

The [`Box`](https://github.com/pvigier/Quadtree/blob/master/include/Box.h) class is not really more complicated:

```cpp
template<typename T>
class Box
{
public:
    T left;
    T top;
    T width; // Must be positive
    T height; // Must be positive

    constexpr Box(T Left = 0, T Top = 0, T Width = 0, T Height = 0) noexcept :
        left(Left), top(Top), width(Width), height(Height)
    {

    }

    constexpr Box(const Vector2<T>& position, const Vector2<T>& size) noexcept :
        left(position.x), top(position.y), width(size.x), height(size.y)
    {

    }

    constexpr T getRight() const noexcept
    {
        return left + width;
    }

    constexpr T getBottom() const noexcept
    {
        return top + height;
    }

    constexpr Vector2<T> getTopLeft() const noexcept
    {
        return Vector2<T>(left, top);
    }

    constexpr Vector2<T> getCenter() const noexcept
    {
        return Vector2<T>(left + width / 2, top + height / 2);
    }

    constexpr Vector2<T> getSize() const noexcept
    {
        return Vector2<T>(width, height);
    }

    constexpr bool contains(const Box<T>& box) const noexcept
    {
        return left <= box.left && box.getRight() <= getRight() &&
            top <= box.top && box.getBottom() <= getBottom();
    }

    constexpr bool intersects(const Box<T>& box) const noexcept
    {
        return !(left >= box.getRight() || getRight() <= box.left ||
            top >= box.getBottom() || getBottom() <= box.top);
    }
};
```

It contains some useful getters.

More interestingly, it contains the method `contains` to check if a box is contained inside another and the method `intersects` to check if a box intersects another one.

We will use `contains` during insertion and removal, and `intersects` for intersection detection.

# Quadtree

Here is a skeleton of the [`Quadtree`](https://github.com/pvigier/Quadtree/blob/master/include/Quadtree.h) class:

```cpp
template<typename T, typename GetBox, typename Equal = std::equal_to<T>, typename Float = float>
class Quadtree
{
    static_assert(std::is_convertible_v<std::invoke_result_t<GetBox, const T&>, Box<Float>>,
        "GetBox must be a callable of signature Box<Float>(const T&)");
    static_assert(std::is_convertible_v<std::invoke_result_t<Equal, const T&, const T&>, bool>,
        "Equal must be a callable of signature bool(const T&, const T&)");
    static_assert(std::is_arithmetic_v<Float>);

public:
    Quadtree(const Box<Float>& box, const GetBox& getBox = GetBox(),
        const Equal& equal = Equal()) :
        mBox(box), mRoot(std::make_unique<Node>()), mGetBox(getBox), mEqual(equal)
    {

    }

private:
    static constexpr auto Threshold = std::size_t(16);
    static constexpr auto MaxDepth = std::size_t(8);

    struct Node
    {
        std::array<std::unique_ptr<Node>, 4> children;
        std::vector<T> values;
    };

    Box<Float> mBox;
    std::unique_ptr<Node> mRoot;
    GetBox mGetBox;
    Equal mEqual;

    bool isLeaf(const Node* node) const
    {
        return !static_cast<bool>(node->children[0]);
    }
};
```

As you can notice, `Quadtree` is a template class. This will allow us to use the class for different purposes as I explained in the introduction.

The template parameters are:

* `T`: the type of the values that will be contained in the quadtree. `T` should be a lightweight type at it will be stored inside the quadtree. A pointer or a small POD type are ideal.
* `GetBox`: the type of a callable that will take a value as input and return a box.
* `Equal`: is the type of a callable to test equality between two values. By default, we use the standard equal operator.
* `Float`: is the arithmetic type to use in the computations. By default, we use `float`.

At the beginning of the class definition, there are three static assertions to check that the given template parameters are valid.

Let us have a look at the node definition. A node just contains pointers to its four children and a list of values it contains. We do not store its bounding box or its depth. We will compute that on the fly.

I have benchmarked both approaches (storing the box and depth or not) and there is no performance penalty in computing them on the fly. Moreover, we are saving some memory.

To be able to distinguish an interior node from a leaf, there is the method `isLeaf`. It just checks that the first child is not null. As all children are null or none are, it is sufficient to check the first one.

Now, we can look at the member variables of `Quadtree`:

* `mBox` is the global bounding box. All values inserted in the quadtree should be contained in it.
* `mRoot` is the root of the quadtree.
* `mGetBox` is the callable that we will use to get a box from a value.
* `mEqual` is the callable that we will use to test the equality between two values.

The constructor simply sets `mBox`, `mGetBox` and `mEqual`, and creates the root node.

The last two parameters, we have not discussed yet, are `Threshold` and `MaxDepth`. `Threshold` is the maximum number of values a node can contain before we try to split it. `MaxDepth` is the maximum depth of a node, we stop trying to split nodes which are at `MaxDepth` because it can hurt performance if we subdivide too much. I have set up these constants to some sane values that should work for most of the cases. You can try to optimize them for some specific configuration.

Now, we are ready to dive in more interesting operations.

# Insertion and Removal

Before, I show the code for insertion, we need to discuss which nodes will contain the values. There are two strategies:

* Only the leaves store values. As the bounding box of a value can intersect several leaves, all these leaves will store the value.
* All the nodes can store values. We store the value in the smallest node that entirely contains its bounding box.

If all the bounding boxes are small and roughly the same size, the first strategy is more efficient when looking for intersections. However, if there are big bounding boxes, there may be degenerate cases where the performance will be very bad. For instance, if we insert a value whose bounding box is the global bounding box, it will be added to all leaves. And if we insert `Threshold` such values, all the nodes will split until we reach `MaxDepth` and all the leaves will contain the values. Thus, the quadtree will contain $$\texttt{Threshold} \times 4^{\texttt{MaxDepth}}$$ values which is ... a lot.

Moreover, with the first strategy, insertion and removal are a bit slower as we have to insert to (or remove from) all nodes that intersect the value.

Thus, I will use the second strategy where there is no degenerate case. As I plan to use the quadtree in many contexts, it will be more convenient. Moreover, it is more suitable for dynamic contexts where we will do a lot of insertions and removals to update the values such as in a physics engine where entities are moving.

To find in which node, we will insert or remove a value, we will rely on two utility functions.

The first one, `computeBox`, computes the box of a child from the box of its parent and the index of its quadrant.

```cpp
Box<Float> computeBox(const Box<Float>& box, int i) const
{
    auto origin = box.getTopLeft();
    auto childSize = box.getSize() / static_cast<Float>(2);
    switch (i)
    {
        // North West
        case 0:
            return Box<Float>(origin, childSize);
        // Norst East
        case 1:
            return Box<Float>(Vector2<Float>(origin.x + childSize.x, origin.y), childSize);
        // South West
        case 2:
            return Box<Float>(Vector2<Float>(origin.x, origin.y + childSize.y), childSize);
        // South East
        case 3:
            return Box<Float>(origin + childSize, childSize);
        default:
            assert(false && "Invalid child index");
            return Box<Float>();
    }
}
```

The second one, `getQuadrant`, returns the quadrant in which a value is:

```cpp
int getQuadrant(const Box<Float>& nodeBox, const Box<Float>& valueBox) const
{
    auto center = nodeBox.getCenter();
    // West
    if (valueBox.getRight() < center.x)
    {
        // North West
        if (valueBox.getBottom() < center.y)
            return 0;
        // South West
        else if (valueBox.top >= center.y)
            return 2;
        // Not contained in any quadrant
        else
            return -1;
    }
    // East
    else if (valueBox.left >= center.x)
    {
        // North East
        if (valueBox.getBottom() < center.y)
            return 1;
        // South East
        else if (valueBox.top >= center.y)
            return 3;
        // Not contained in any quadrant
        else
            return -1;
    }
    // Not contained in any quadrant
    else
        return -1;
}
```

It returns `-1` if it is not entirely contained in any of the quadrants.

We are now ready to see the insertion and removal methods.

## Insertion

The `add` method just calls a private auxiliary method:

```cpp
void add(const T& value)
{
    add(mRoot.get(), 0, mBox, value);
}
```

Here is the code of the auxiliary method:

```cpp
void add(Node* node, std::size_t depth, const Box<Float>& box, const T& value)
{
    assert(node != nullptr);
    assert(box.contains(mGetBox(value)));
    if (isLeaf(node))
    {
        // Insert the value in this node if possible
        if (depth >= MaxDepth || node->values.size() < Threshold)
            node->values.push_back(value);
        // Otherwise, we split and we try again
        else
        {
            split(node, box);
            add(node, depth, box, value);
        }
    }
    else
    {
        auto i = getQuadrant(box, mGetBox(value));
        // Add the value in a child if the value is entirely contained in it
        if (i != -1)
            add(node->children[static_cast<std::size_t>(i)].get(), depth + 1, computeBox(box, i), value);
        // Otherwise, we add the value in the current node
        else
            node->values.push_back(value);
    }
}
```

Firstly, there are some assertions to check that we are not doing something that is not making sense such as inserting a value in a node that is not containing its bounding box.

Then, if the node is a leaf and we can insert a new value in it i.e. we are at `MaxDepth` or `Threshold` is not reached yet, we insert there. Otherwise, we split this node and we try again.

If it is an interior, we compute the quadrant in which the value's bounding box is contained. If it is entirely contained in a child, we do a recursive call. Otherwise, we insert in this node.

Finally, here is the split routine:

```cpp
void split(Node* node, const Box<Float>& box)
{
    assert(node != nullptr);
    assert(isLeaf(node) && "Only leaves can be split");
    // Create children
    for (auto& child : node->children)
        child = std::make_unique<Node>();
    // Assign values to children
    auto newValues = std::vector<T>(); // New values for this node
    for (const auto& value : node->values)
    {
        auto i = getQuadrant(box, mGetBox(value));
        if (i != -1)
            node->children[static_cast<std::size_t>(i)]->values.push_back(value);
        else
            newValues.push_back(value);
    }
    node->values = std::move(newValues);
}
```

We create the four children, and then for each value of the parent, we decide in which node (a child or the parent) the value should go.

## Removal

Again, the `remove` method just calls an auxiliary method:

```cpp
void remove(const T& value)
{
    remove(mRoot.get(), mBox, value);
}
```

Here is the code of the auxiliary method, it is very analogous to insertion:

```cpp
bool remove(Node* node, const Box<Float>& box, const T& value)
{
    assert(node != nullptr);
    assert(box.contains(mGetBox(value)));
    if (isLeaf(node))
    {
        // Remove the value from node
        removeValue(node, value);
        return true;
    }
    else
    {
        // Remove the value in a child if the value is entirely contained in it
        auto i = getQuadrant(box, mGetBox(value));
        if (i != -1)
        {
            if (remove(node->children[static_cast<std::size_t>(i)].get(), computeBox(box, i), value))
                return tryMerge(node);
        }
        // Otherwise, we remove the value from the current node
        else
            removeValue(node, value);
        return false;
    }
}
```

If the current node is a leaf, we remove the value from the list of values of the current node and we return `true`. The return value will tell its parent that it should try to merge with its children. Otherwise, we determine in which quadrant the value's bounding box lies in. If it is entirely contained in a child, we do a recursive call and we try to merge the node if we are told to. Otherwise, we remove from the values of the current node.

As we do not care about the order of the values stored in a node, I use a small optimization to erase a value, I just swap the value to erase with the last one and pop back:

```cpp
void removeValue(Node* node, const T& value)
{
    // Find the value in node->values
    auto it = std::find_if(std::begin(node->values), std::end(node->values),
        [this, &value](const auto& rhs){ return mEqual(value, rhs); });
    assert(it != std::end(node->values) && "Trying to remove a value that is not present in the node");
    // Swap with the last element and pop back
    *it = std::move(node->values.back());
    node->values.pop_back();
}
```

Finally, we must have a look to `tryMerge`:

```cpp
void tryMerge(Node* node)
{
    assert(node != nullptr);
    assert(!isLeaf(node) && "Only interior nodes can be merged");
    auto nbValues = node->values.size();
    for (const auto& child : node->children)
    {
        if (!isLeaf(child.get()))
            return false;
        nbValues += child->values.size();
    }
    if (nbValues <= Threshold)
    {
        node->values.reserve(nbValues);
        // Merge the values of all the children
        for (const auto& child : node->children)
        {
            for (const auto& value : child->values)
                node->values.push_back(value);
        }
        // Remove the children
        for (auto& child : node->children)
            child.reset();
        return true;
    }
    else
        return false;
}
```

`tryMerge` checks that all its children are leaves and that the total number of its values and its children's values is lower than the threshold. If it is the case, we copy all the values in the children in the current node and we remove the children. If the node is merged we return `true` so that its parent also tries to merge with its children.

# Finding Intersections

## Intersection with a Box

Finally, we are coming to interesting things: finding intersections. The first use case is retrieving all the values that are intersecting a given box. This is what we need to do culling for instance.

This is the goal of `query`:

```cpp
std::vector<T> query(const Box<Float>& box) const
{
    auto values = std::vector<T>();
    query(mRoot.get(), mBox, box, values);
    return values;
}
```

In this method, we just allocate a `std::vector` that will contain the values that intersect the bounding box, and we call an auxiliary method:

```cpp
void query(Node* node, const Box<Float>& box, const Box<Float>& queryBox, std::vector<T>& values) const
{
    assert(node != nullptr);
    assert(queryBox.intersects(box));
    for (const auto& value : node->values)
    {
        if (queryBox.intersects(mGetBox(value)))
            values.push_back(value);
    }
    if (!isLeaf(node))
    {
        for (auto i = std::size_t(0); i < node->children.size(); ++i)
        {
            auto childBox = computeBox(box, static_cast<int>(i));
            if (queryBox.intersects(childBox))
                query(node->children[i].get(), childBox, queryBox, values);
        }
    }
}
```

Firstly, we add all the values stored in the current node that intersect with the query box. Then, if the current node is an interior node, we do a recursive call for each child whose bounding box intersects the query box.

## All Pairwise Intersections

The second use case that is supported is finding all the pairs of values stored in the quadtree that intersect. It is particularly useful for doing a physics engine. It is possible to achieve that with the `query` method. Indeed, we can call `query` for the bounding box of all values. However, it is possible to do that a bit more efficiently by adding the intersection only once for a pair (while we would find it twice by using `query`).

To be able to do that, we need to notice that an intersection can only occur:

* between two values stored in the same node

or

* between a value stored in a node and another one stored in a descendant of this node.

Thus, we only need to check the intersection between:

* a value and the next values stored in the same node

and

* a value and the values stored in the descendant.

This way, we are sure to not report twice the same intersection.

Here is the code of `findAllIntersections`:

```cpp
std::vector<std::pair<T, T>> findAllIntersections() const
{
    auto intersections = std::vector<std::pair<T, T>>();
    findAllIntersections(mRoot.get(), intersections);
    return intersections;
}
```

Again, we just allocate a `std::vector` to store the intersections and we call an auxiliary function:

```cpp
void findAllIntersections(Node* node, std::vector<std::pair<T, T>>& intersections) const
{
    // Find intersections between values stored in this node
    // Make sure to not report the same intersection twice
    for (auto i = std::size_t(0); i < node->values.size(); ++i)
    {
        for (auto j = std::size_t(0); j < i; ++j)
        {
            if (mGetBox(node->values[i]).intersects(mGetBox(node->values[j])))
                intersections.emplace_back(node->values[i], node->values[j]);
        }
    }
    if (!isLeaf(node))
    {
        // Values in this node can intersect values in descendants
        for (const auto& child : node->children)
        {
            for (const auto& value : node->values)
                findIntersectionsInDescendants(child.get(), value, intersections);
        }
        // Find intersections in children
        for (const auto& child : node->children)
            findAllIntersections(child.get(), intersections);
    }
}
```

In the first step, we check the intersections between values stored in the current node. Then, if the current node is an interior node, we check the intersections between values stored in this node and values stored in its descendants by calling `findIntersectionInDescendants`. Finally, we do recursive calls.

`findIntersectionsInDescendants` recursively finds intersections between the given value and all the values stored in the subtree:

```cpp
void findIntersectionsInDescendants(Node* node, const T& value, std::vector<std::pair<T, T>>& intersections) const
{
    // Test against the values stored in this node
    for (const auto& other : node->values)
    {
        if (mGetBox(value).intersects(mGetBox(other)))
            intersections.emplace_back(value, other);
    }
    // Test against values stored into descendants of this node
    if (!isLeaf(node))
    {
        for (const auto& child : node->children)
            findIntersectionsInDescendants(child.get(), value, intersections);
    }
}
```

That's all! Again, you can retrieve all the code from [GitHub](https://github.com/pvigier/Quadtree).

# Useful Resources

If you want to know more about collision detection and space partitioning data structure, I advise you to read [Real-Time Collision Detection](http://realtimecollisiondetection.net/) by Christer Ericson. It covers a lot of topics in-depth but it is very understandable. Moreover, it is possible to read chapters independently. It is really a good reference.

# Conclusion

That is all for collision detection. However, collision detection is only half of a physics engine. The other half is [collision resolution]({{ site.baseurl }}{% post_url 2019-08-11-vagabond-2d-physics-engine %}). Next week, I will work on that and I hope I will have more nice pictures to share with you.

See you next week for more!