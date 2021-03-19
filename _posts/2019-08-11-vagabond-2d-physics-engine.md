---
layout: post
title: "Vagabond &#8211; 2D Physics Engine"
date: 2019-08-11
author: pierre
tab: blog
tags: vagabond game-engine
---

This week, I worked on the physics engine. It was the first time, I wrote a physics engine thus it took me a lot of time to design an algorithm that works well. In this article, I will try to share with you the issues I faced and then my current design of the physics engine.

<video controls width="256">
    <source src="/media/video/vagabond-2d-physics-engine/physics_engine_demo.mp4" type="video/mp4">
    Sorry, your browser doesn't support embedded videos.
</video>

<!--more-->

# Issues

Last week, I implemented a [quadtree]({{ site.baseurl }}{% post_url 2019-08-04-quadtree-collision-detection %}) which allows me to find all the collisions inside a set of entities fairly quickly. But one question remains: what to do then?

One idea is to move all the entities at the place they would be just before their first collision. The problem in doing so is that by moving them, we may create new collisions.

{: .image-table }
Before resolution | After resolution
:---:|:---:
![](/media/img/vagabond-2d-physics-engine/before_collision.svg){: width="250" } | ![](/media/img/vagabond-2d-physics-engine/after_collision.svg){: width="250" }

In the literature (see the Useful resources section), I found two main solutions to this problem:

* Resolving again until there is no collision left.
* Simulating the movement until the first collision occurs, resolving this collision, simulating until the next collision, resolving it, etc. Until we reach the current time.

While these methods give the most precise simulation (especially the second one, the first one is an approximation), I feel they are too complex and computationally expensive for what I am trying to achieve: a very simple physics engine for a 2D RPG.

Another issue that may occur with discrete simulation is tunneling: we may have no collision at the timestamps but go through obstacles between the two timestamps (see image below).

![](/media/img/vagabond-2d-physics-engine/tunneling.svg){: width="250" .center-image }

Finally, in most resources I found, they were simulating realistic physic laws. Thus, they resolved collision using [elastic](https://en.wikipedia.org/wiki/Elastic_collision) or [inelastic collisions](https://en.wikipedia.org/wiki/Inelastic_collision). These behaviors are not well-suited for a top-down 2D RPG.

# Design and Algorithm

## Design

In this section, I will share with you the current design of my physics engine. It is very simple but I think it works well for simple games and it is quite expandable. I will only show pseudo-code because there is no critical part but I leverage the [quadtree implementation I detailed last week]({{ site.baseurl }}{% post_url 2019-08-04-quadtree-collision-detection %}).

The first strong design choice is that there will be two types of nodes: static nodes and dynamic nodes. As you have surely guessed, static nodes will not be able to move while dynamic nodes can.

The other strong assumption is that we will only resolve collisions between static and dynamic nodes. In other words, a dynamic node cannot penetrate a static node but dynamic nodes may overlap. However, we will detect collisions between dynamic nodes and send a message when one occurs.

Thus, static nodes are perfect to model terrain or objects such as trees, furniture, etc. Dynamic nodes will model players, characters, monsters, etc.

I do no think that not resolving the collisions between dynamic nodes in the physics engine is a big limitation because it does not necessarily make sense to do that. Indeed, it may be convenient to let the player or characters be able to go through other characters. This way, it is not possible that one character blocks the passage for others. It can make the artificial intelligence controlling the characters way simpler. Moreover, if necessary most of these collisions between characters may be avoidable using steering behaviors.

Another argument is that during a fight, it may be better to let the entities determining how the enemy should react to a hit than letting the physics engine which has no idea of the characteristics of the weapon doing that.

The class that will model the static nodes is an abstract class in my implementation, it has two virtual methods: one for getting the bounding box of the entity to be able to insert the node in a quadtree, the other to detect a collision with a dynamic node:

```cpp
struct StaticNode
{
    virtual const Box& getBoundingBox() const = 0;
    virtual CollisionResult& detectCollision(const DynamicNode& node) const = 0;
};
```

`CollisionResult` is a simple structure which contains information about the collision such as the time of impact `t` and the normal of the axis of collision `n`.

Currently, I have two derived classes of `StaticNode`: `TileNode` for tiled terrains and `BoxNode` for other entities that are just a box.

Here is the data structure for dynamic nodes:

```cpp
struct DynamicNode
{
    float mass;
    Box box;
    Vector2 velocity;
    Vector2 netForce;
};
```

`mass` is the mass of the object, `box` contains the position of the entity and its size, `velocity` is its current velocity and `netForce` is the sum of all the forces that are currently applying on the object.

## Algorithm

Here are the three steps done in the physics engine:

```
1. Compute the new positions for all dynamic nodes.
2. For each dynamic node, find its first collision with a static node and resolve it.
3. Detect collisions between dynamic nodes.
```

### Simulation

In the first step, we compute the new position of a dynamic node if there is no obstacle.

To update the position, I just integrate the [second law of motion](https://en.wikipedia.org/wiki/Newton%27s_laws_of_motion) using [Euler method](https://en.wikipedia.org/wiki/Euler_method):

```
acceleration = node.netForce / node.mass
node.velocity += acceleration * dt
newPosition = node.box.position + node.velocity * dt
newBox = Box(newPosition, node.box.size)
```

Nothing fancy here.

### Dynamic-Static Collisions

The next step is to find the collisions with static nodes, to do that I use a quadtree `staticNodeQuadtree` that contains all the static nodes. Then, I look for the first collision, move the dynamic node at the place where it collides. Finally, I reset its velocity in the direction of the normal so that it remains only the tangential component and it stops trying to enter in the static node:

```python
staticNodesHit = staticNodeQuadtree.query(newBox)
minT = infinity
normal = nil
for staticNode in staticNodesHit
    result = staticNode.detectCollision(node)
    if result.t < minT
        minT = result.t
        normal = result.n
node.box.position += minT * node.velocity
node.box.velocity -= node.box.velocity.dot(normal) * normal
```

### Dynamic-Dynamic Collisions

The last step is finding all the collisions between dynamic nodes. Again, we will use a quadtree to do all the hard work:

```python
dynamicNodeQuadtree = Quadtree()
for node in dynamicNodes
    dynamicNodeQuadtree.add(node)
collisions = dynamicNodeQuadtree.findAllIntersections()
for node1, node2 in collisions
    sendMessage(CollisionMessage(node1, node2))
```

Contrary to the quadtree containing the static nodes that is kept between iterations, the quadtree for the dynamic nodes is created from scratch at each iteration as all the dynamic nodes may have moved and need an update.

# Possible Improvements

Now that we have a basic physics engine that works we can try to add some improvements.

## Tunneling

In the previous algorithm, we do not tackle the problem of tunneling. Tunneling may not be an issue depending on the geometry present in the world and if the framerate is constantly high. However, there is a simple way to check that no tunneling takes place. To do that, we must modify the quadtree data structure a bit so that we can check the collision between nodes in the quadtree and a given parallelogram. It is easy to do so.

Then, we have to remark that the swept area of the bounding box of a dynamic node between two iterations is a parallelogram. Thus, we can find all the static nodes it may collide with by querying the quadtree containing the static nodes with a parallelogram that represents the swept area.

![](/media/img/vagabond-2d-physics-engine/tunneling_parallelogram.svg){: width="250" .center-image }

## Wall Sliding

In a 2D top-down RPG, we may want the player to be able to slide along the walls even if a part of the velocity is toward the wall.

After searching a bit on the Internet, I have found that one method often proposed is to first move along one axis an then along the other. I am not fond of this method for several reasons. Firstly, because moving along X then along Y is not the same as moving along X and Y simultaneously so it may return weird results.

{: .image-table }
Diagonal | Along axes
:---:|:---:
![](/media/img/vagabond-2d-physics-engine/diagonal.svg){: width="250" } | ![](/media/img/vagabond-2d-physics-engine/move_along_axis.svg){: width="250" }

However, with a low velocity, I don't think it is noticeable. The second reason is that it works only if all the walls are vertical or horizontal, it is not expandable.

An alternative, that is in my opinion better, is after the first collision that occurs at `t` if there is time left, we try to move again with the new velocity that just has been updated (we kept only the tangential component). We can even do that several times until we reach `dt` or a maximum number of iterations:

```python
t = 0
nbCollisions = 0
while t < dt and nbCollisions < maxNbCollisions
    collision = findFirstCollision()
    update position and velocity
    t += collision.t
    nbCollisions += 1
```

Here is a figure to illustrate:

![](/media/img/vagabond-2d-physics-engine/wall_sliding.svg){: width="250" .center-image }

## Hitboxes

Finally, the last improvement I am thinking of is supporting several hitboxes for a dynamic node. For instance, we may want to use one hitbox for the body to be able to detect received hits and one hitbox for the weapon to be able to detect given hits.

![](/media/img/vagabond-2d-physics-engine/hitboxes.png){: .center-image }

I would simply add a list of hitboxes in the definition of `DynamicNode` and when there is a collision between the bounding boxes of two dynamic nodes, I would test the collision between their respective hitboxes.

# Useful Resources

* [Real-Time Collision Detection](http://realtimecollisiondetection.net/) by Christer Ericson
* Game Physics Engine Design by Ian Millington
* Mathematics & Physics for Programmers by Danny Kodicek

# Conclusion

I hope this article give you some ideas on how to start writing your own physics engine.

Next week, I may polish this a bit and start working on the audio engine.

See you next week for more!