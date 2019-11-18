---
layout: post
title: "Vagabond &#8211; Teleporting into Dungeons"
date: 2019-10-20
author: pierre
tab: blog
tags: vagabond
---
Hi everyone!

Last two weeks, I was working on teleporting the players from one map to another. I will use this system to enter into dungeons, buildings, etc. and also to leave them. But it was not that easy to implement as it required some deep architectural changes in the game engine.

<video controls width="600">
    <source src="/media/video/vagabond-teleporting-into-dungeons/dungeon_entrance.mp4" type="video/mp4">
    Sorry, your browser doesn't support embedded videos.
</video>

<!--more-->

# A multilayer world

In a 2D top-down game, it is hard to make an open-world with only one map like Minecraft. It would mean it is not possible to go underground or upstairs in buildings. Thus, I need to be able to display several maps and to go from one map to another using *doors*.

But it is easier said than done. In particular for a multiplayer game, as there may be several players on different maps, and so you must manage several maps simultaneously. While in a single-player game, you just keep the map where the player is currently on.

Moreover, when I store the position of an entity, not only I must store its `x` and `y` coordinates but also the map on which it is. More precisely, as the world can be quite large, it is cut in chunks so I store the indices of the chunk on which the entity is currently on.

Previously, I was using a data structure like this one:

```cpp
struct Position
{
    float x;
    float y;
};
```

And now, I am using something like that:

```cpp
struct Position
{
    // Chunk id
    int i;
    int j;
    int k;
    // Offset
    float x;
    float y;
};
```

This little change in the `Position` data structure forced me to rework my physics engine and other systems.

# Teleportation system

Once these architectural changes were done, I was ready to implement my teleportation system. As I use an entity-component-system, implementing this was just creating a new component and a new system.

The door component just contains a destination:

```cpp
struct DoorComponent
{
    Position destination;
}
```

The door system receives collision messages from the physics engine. If a collision occurs between an entity that has a door component and another entity, it transports the other entity at the door's destination. Nothing difficult, once all the other systems are in place.

And it works well, even in multilayer mode where there are two players on different maps:

<video controls width="600">
    <source src="/media/video/vagabond-teleporting-into-dungeons/dungeon_entrance_multiplayer.mp4" type="video/mp4">
    Sorry, your browser doesn't support embedded videos.
</video>

# Dungeon entrances

Finally, I worked on a dungeon entrance generator. Nothing fancy but it works well and you can change the materials of the walls to match the one used in the dungeon:

![](/media/img/vagabond-teleporting-into-dungeons/dungeon_entrances.gif){: .center-image .modal-image }

# Conclusion

That's all for this devlog.

Next week, I will work on monsters and add them inside the dungeons. That will be a big step!

See you next week for more!