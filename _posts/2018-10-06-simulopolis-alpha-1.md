---
layout: post
title: "Simulopolis Alpha Is Released"
date: 2018-10-06
author: pierre
tab: blog
tags: simulopolis
---

![Cover](/media/img/simulopolis-alpha-1/cover.png){: .center-image .modal-image }

Today, I'm happy to announce you that the first release of my city building game, Simulopolis, is available online.

It's been a long and chaotic journey which lasted a bit more than six months.

<!--more-->

# Philosophy

At first, there was no idea or concept behind Simulopolis, it was only an experiment about game engine architecture. But then I decided that it would be cool to make a full game from it.

My main goal with Simulopolis is not to ship the best game ever but to learn as much as possible about game development. That's the reason why I decided to use as few dependencies as possible. Currently, Simulopolis has only three: [SFML](https://www.sfml-dev.org/), [TinyXML-2](http://www.grinninglizard.com/tinyxml2/) and [Boost Serialization](https://www.boost.org/doc/libs/1_68_0/libs/serialization/doc/index.html). This means that all the rest (game engine, GUI, AI, PCG, etc.) is handmade.

And doing everything is as interesting as it is time consuming. The game I have after months of development (not full-time, only couple of hours a day and not every day) is barely as good as some games developed during a game jam.

# Concept

When I started, I had no idea of what the game would be. I had a prototype of a game engine and some assets done by [Kenney](https://opengameart.org/users/kenney). I also wanted to explore game AI and PCG so there must be some in the game.

Finally, I choose to make a city building game where each citizen is an autonomous agent. Citizens have different personalities and skills. They are free to evolve in the city you build for them.

![Screenshot](/media/img/simulopolis-alpha-1/screenshot.png){: .center-image .modal-image }

They are no precise goal in Simulopolis, it is a sandbox game. You can do whatever you want. Not so much for now.

But my goal is that you can try several economic systems such that capitalism, socialism, a mixed of both or even a system without money.

# Try It!

The game is available on [itch.io](https://pvigier.itch.io/simulopolis) on Windows and Linux and it is totally free!

<iframe src="https://itch.io/embed/295935" width="552" height="167" frameborder="0"></iframe>

However, I must warn you that the game is not finished. Many features are missing, there is no tutorial, the interface is ugly, ... This makes the game hardly playable. But you can try this first version to see where I start and see the future evolution.

Finally, the game is open-source. It is licensed under the GPLv3. You can find the code on my [github account](https://github.com/pvigier/Simulopolis).

# What's Next?

My goal is to make the game enjoyable before moving on. I hope to reach that point before the end of the year and I will try to release a new version of the game, with new features, every two weeks.

I will also try to publish some articles on the blog about what I learnt in this project. I have already planned to write one about the procedural generation of terrains and the difficulties I faced to distribute the game on Linux. Maybe I will write on the citizen AI too. If you want me to write on a specific point, do not hesitate to ask!

See you!
