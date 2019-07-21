---
layout: post
title: "Vagabond &#8211; Game engine foundations"
date: 2019-07-14
author: pierre
tab: blog
tags: vagabond game-engine
---

This week, I have continued to develop my game engine. I have laid the foundation for a certain number of engines and systems.

As you may know, I am using SFML and C++ to create my game. However, building a game directly with SFML is a bit tough as SFML is a low-level library. Thus I have decided to build a small game engine to help me later on. Nothing too fancy or general, just the necessary to be able to power Vagabond.

But, I did not do everything from scratch. I used the experience I acquired during the development of [Simulopolis](https://github.com/pvigier/Simulopolis). Moreover, when I finished developing Simulopolis, I read two books [SFML Game Development](https://www.packtpub.com/game-development/sfml-game-development) and [SFML Game Development By Example](https://www.packtpub.com/game-development/sfml-game-development-example) to compare their approaches and mine. It was really instructive.

<!--more-->

# States

The first thing I implemented was the state system. States are used to separate the different screens of the game such as the menu screen, the option screen, the credits screen or the game screen. They allow to separate the logic of each screen nicely.

I decided to use a state stack and not a finite state machine as it is more powerful and flexible. For example, with a state stack, it is a possible to open a pause screen when playing and then get back the game. Doing so is not possible with a simple finite state machine.

# Render engine

Next, I worked on the render engine. SFML provides the `sf::RenderWindow` class which is used to open a window, poll events, and draw inside the window.

My render engine is just a wrapper around `sf::RenderWindow`. It manages the screen resolution and other settings such as switching between full-screen and windowed mode.

Finally, its last mission is to forward input events to the input manager.

# Scene graph

Then, I implemented a [scene graph](https://en.wikipedia.org/wiki/Scene_graph). The scene graph is used by the game state to draw all the game entities easily and efficiently. It provides several types of nodes to display sprites, animated sprites, tiles or particles. Moreover, it will also implement the light system with light nodes.

The scene graph will also take care of drawing all the nodes in the right order. Finally, as all the nodes and the camera are managed by the scene graph it is easy to implement culling with a space-partitioning data structure such as a [quadtree](https://en.wikipedia.org/wiki/Quadtree).

# Input management

I also work on an input manager. The input manager goal is to manage the bindings and to process the input events. It will allow the player to configure the bindings as he wants. When all the conditions of a binding are satisfied, the input manager simply calls the corresponding registered callback.

# Integration of Dear ImGui

Finally, I integrated [Dear ImGui](https://github.com/ocornut/imgui) in the engine. To do that with SFML, I used a [nice binding](https://github.com/eliasdaler/imgui-sfml) by [Elias Daler](https://eliasdaler.github.io/). It was super easy!

I made a small window with some metrics to get started with ImGui:

![](/media/img/vagabond-game-engine-foundations/debug_window.png){: width="400" .center-image .modal-image }

# Conclusion

That is all for this week. I did not code a lot, I essentially designed the engine. Next week, I will add more features to the graphics engine and maybe start to work on the networking part.

See you next week for more!

*If you are interested by my adventures during the development of Vagabond, you can follow me on [Twitter](https://twitter.com/PierreVigier).*