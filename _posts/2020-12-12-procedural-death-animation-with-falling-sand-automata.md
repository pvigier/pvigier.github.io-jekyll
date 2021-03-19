---
layout: post
title: "Procedural Death Animation With Falling Sand Automata"
date: 2020-12-12
author: pierre
tab: blog
tags: vagabond pcg
---
Hi everyone!

In this post, I will show you how I used falling sand automata to generate death animations for monsters of my game [Vagabond](https://www.vagabondgame.com/).

![](/media/img/procedural-death-animation-with-falling-sand-automata/death_animations.gif){: .center-image .modal-image }

<!--more-->

# Falling Sand Automata

A falling sand automaton is a [cellular automaton](https://en.wikipedia.org/wiki/Cellular_automaton) that simulates how grains of sand move due to gravity and create piles.

The rules are simple:
* If the cell below a sand grain is empty, the sand grain moves to the empty cell (see (a)).
* If the cell below a sand grain is full but the cell at bottom left or the cell at bottom right is free, the sand grain moves there (see (b)). If both are free, choose one randomly.
* In the other cases, the sand grain does not move.

![](/media/img/procedural-death-animation-with-falling-sand-automata/automata_rules.png){: .center-image .modal-image }

If you want to know more about falling sand automata, you can look at this [article](https://w-shadow.com/blog/2009/09/29/falling-sand-style-water-simulation/).

With these simple rules you can obtain animations like this:

![](/media/img/procedural-death-animation-with-falling-sand-automata/sand.gif){: .center-image .modal-image }

# Generating Death Animations

Now let us see how to use the falling sand automata to create death animations for the monsters.

The idea is to consider the non transparent pixels of the image as the sand grains and to make them fall to create a pile from the corpse of the monster. The only difference with the rules previously mentionned is that each sand grain has now a color. Here is what we obtain:

![](/media/img/procedural-death-animation-with-falling-sand-automata/death_animation.gif){: .center-image .modal-image }

In my opinion it is not very appealing for two reasons: the pile is very high and everything is falling at the same speed.

To fix the height issue, I use a 3D cellular automaton. I use several layers of the simple 2D cellular automaton. At the initial state, the image is in the middle layer, then the grain sands can not only move to bottom left and bottom right cells but also to the bottom cells of the previous and next layers:

![](/media/img/procedural-death-animation-with-falling-sand-automata/3d_automata_rules.png){: .center-image .modal-image }

To obtain an image from the 3D state of the cellular automaton, I project the state to 2D by taking, for each (i, j) coordinates, the first non transparent cell where k iterates the layers. Here is the result with three layers:

![](/media/img/procedural-death-animation-with-falling-sand-automata/death_animation_3d.gif){: .center-image .modal-image }

To improve the speed issue, I randomize the number of rows a grain fall in one step between 1 and $$n$$. In practice, I use $$n = 2$$ or $$n = 3$$. Here is the result:

![](/media/img/procedural-death-animation-with-falling-sand-automata/death_animation_3d_speed.gif){: .center-image .modal-image }

It produces some holes during the fall as if the monster is disintegrating.

Finally, I skip some frames to have animations with exactly 6 frames. Here is the final result:

![](/media/img/procedural-death-animation-with-falling-sand-automata/death_animation_3d_speed_skip.gif){: .center-image .modal-image }

The grains are falling, on average, at a constant pace, while the gravity should make them accelerate. If you want you can skip more frames at the end of the animation to simulate that.

You can find the complete script on [GitHub](https://github.com/pvigier/lpc-scripts/blob/main/death_animation.py).

What's nice is that if we are not happy with the result, we can rerun the script with a different seed to obtain a new animation. Here are several animations for the bat:

![](/media/img/procedural-death-animation-with-falling-sand-automata/death_animation_seeds.gif){: .center-image .modal-image }

See you next time for more!