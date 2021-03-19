---
layout: post
title: "Vagabond &#8211; Let's Fight"
date: 2019-11-18
author: pierre
tab: blog
tags: vagabond
---
Hi everyone!

It's been a long time since the last devlog.

I have been mainly working on monsters and combats. I created a lot of components and systems but nothing which deserves its own blog post. Let's see the improvements.

<video controls width="600">
    <source src="/media/video/vagabond-let-s-fight/attack_dungeon_updated.mp4" type="video/mp4">
    Sorry, your browser doesn't support embedded videos.
</video>

<!--more-->

# Monsters

Firstly, I ingested several monsters, even if you will only see bats in this devlog. And implemented a very basic artificial intelligence: they wander in rooms of dungeons.

<video controls width="600">
    <source src="/media/video/vagabond-let-s-fight/monster_wander.mp4" type="video/mp4">
    Sorry, your browser doesn't support embedded videos.
</video>

# Combat System

It was then time to kick their ass. I had to modify the physics engine to support hurtboxes and hitboxes.

Then, I had to improve the way animations are synchronized between clients which requires me to do huge changes in the codebase.

Eventually, it was possible to attack the monsters:

<video controls width="600">
    <source src="/media/video/vagabond-let-s-fight/combat_system.mp4" type="video/mp4">
    Sorry, your browser doesn't support embedded videos.
</video>

Monsters receive a small knockout just after being hit.

It is not impressive yet, but with some particle effects and sound effects, it should be funnier.

# HUD

It had been weeks, I was postponing the work on the user interface. I rolled up my sleeves and made a basic HUD to display the target monster's name and health.

Moreover, I change the inputs. Now, everything is done with the mouse. Thus, I created a custom golden cursor you can see in the video below.

<video controls width="600">
    <source src="/media/video/vagabond-let-s-fight/attack_interface.mp4" type="video/mp4">
    Sorry, your browser doesn't support embedded videos.
</video>

I also started to do preparatory work on a drag and drop system for the future inventory interface.

# Cave Generation Update

Finally, I worked on the cave generator. I fixed pernicious bugs occurring during tile generation. Moreover, I added some cycles during maze generation. I am very happy with the results:

![](/media/img/vagabond-let-s-fight/dungeon_generation_v3.gif){: width="400" .center-image .modal-image }

I shared this animation on Reddit on [r/gamedev](https://www.reddit.com/r/gamedev/comments/dx95df/cave_generation_using_bsp_and_cellular_automaton/), I received many kind comments, it was very motivating!

# Conclusion

That's all for this devlog.

I am currently working on items, inventory, and characters. I think it will be a great step for the game and I hope I will have a lot of nice stuff to show.

See you next time for more!