---
layout: post
title: "Vagabond &#8211; Exploring the world"
date: 2019-09-22
author: pierre
tab: blog
tags: vagabond
---
Hi everyone!

This is the first devlog since I stopped working on the game engine and really started working on the game. I did not publish a post last week, I started writing an article about the new implementation of my [entity-component-system library](https://github.com/pvigier/ecs) but I did not find the time to finish it yet. The last two weeks were pretty busy.

<!--more-->

# Moving in the world

The first goal was to be able to move a character in a generated world. To do that, I had to plug the [world generator]({{ site.baseurl }}{% post_url 2019-05-26-vagabond-generating-tiles %}) with the rest of the game engine. Then, displaying the world and the character was just creating some nodes in the scene graph.

I tried to use the entity-component-system pattern as much as possible. It was a bit difficult at the beginning as it was the first time I used it in practice. Separating data in components and logic in systems is a manner of thinking I was not familiar with. However, it revealed extremely powerful, in particular in a roguelite and a multiplayer context. Being able to create new entities by blending components is useful to generate a huge amount of content. Moreover, the separation of logic in systems allows reusing common parts of logic between the server and the client easily.

Here is a small screencast of a (naked) character walking in the world:

<video controls width="600">
    <source src="/media/video/vagabond-exploring-the-world/walking.mp4" type="video/mp4">
    Sorry, your browser doesn't support embedded videos.
</video>

# Multiplayer mode is working

I decided to directly create the multiplayer mode. In fact, the single-player mode just runs a local server which does not accept any external connection. I think that in the long term, starting with this design is easier than firstly creating the single-player game and then trying to add a multiplayer mode.

The client does not generate nor store the world, it is streamed by the server. Thus, the connection is pretty fast, the player does not have to wait while the entire world is downloading.

Here is a small screencast of a second player joining the server opened by another player:

<video controls width="600">
    <source src="/media/video/vagabond-exploring-the-world/multiplayer.mp4" type="video/mp4">
    Sorry, your browser doesn't support embedded videos.
</video>

# Conclusion

The next goal is to be able to add pieces of equipment to the characters so that they are not naked anymore!

See you next week for more!