---
layout: post
title: "Vagabond &#8211; Forest generation"
date: 2019-06-09
author: pierre
tab: blog
tags: vagabond pcg
---

![](/media/img/vagabond-forest-generation/cell_generation.gif){: width="400" .center-image .modal-image }

This week, I have continued to work on the object placement system that I started to implement [last week]({{ site.baseurl }}{% post_url 2019-06-02-vagabond-rasterizing-roads-rivers %}). I entitled this article "Forest generation" as we will place trees, but we will also place other decorations that will fill biomes such as plants, bushes, rocks, carcasses, etc.

<!--more-->

# Multi-class Poisson disk sampling

Last week, I show an image of points sampled using Poisson disk sampling. Then I mapped these points to one of the decorations randomly.

![](/media/img/vagabond-forest-generation/poisson_disk_sampling.png){: width="400" .center-image .modal-image }

The problem is that I do not want the same distribution for all the decorations. For instance, small objects such as flowers or bushes should appear more frequently but also be more tightly packed than big trees.

To solve this problem, I will generate several classes of points that have different properties (minimal distance to a point of the same class, minimal distance to a point of other classes, ...). This problem is called multi-class Poisson disk sampling.

The common way to perform multi-class Poisson disk sampling is to reuse the algorithm for the one-class case to generate several layers of points (one layer for each class) and then to discard samples that overlapped. Usually, the layers are ordered so that higher layers have priority over lower layers. And we put the classes with larger minimal distances in the higher layers.

I decided to use another method and to generate simultaneously the samples for all the classes. I have not had the time to compare both methods in term of performance and quality of the sampling yet. But if some of you are interested, I will write a separate article describing the algorithms in detail and analyzing the differences.

Here is a picture with four classes that have different properties:

![](/media/img/vagabond-forest-generation/multiclass_poisson_disk_sampling.png){: width="400" .center-image .modal-image }

# Filling the cells

Now that we have an algorithm to place objects, it is time to effectively add objects. For now, I annotated each object with two distances (one for the minimal distance with objects of the same class and the minimal distance with objects of other classes). But I may change the system where a same class can represent several objects because I have many objects that have the same properties. For instance, all the variations of the same flower have the same properties.

Here is an example of a cell filled with multi-class Poisson disk sampling where I displayed the disk of each object:

![](/media/img/vagabond-forest-generation/desert_disks.png){: width="400" .center-image .modal-image }

We can check that no center of a disk is inside another disk.

# Conclusion

That's all for this week. It took me some time to design the algorithm, ingest all the objects in the editor and tweak the parameters for the generation. But I also started to code some parts of the game engine and to design other generators.

Next, I think I will work on the [dungeon and cave generator]({{ site.baseurl }}{% post_url 2019-06-23-vagabond-dungeon-cave-generation %}).

See you next week for more!

# Credits

Here are the links to all the tilesets I used in this article:

* [[LPC] Terrains](https://opengameart.org/content/lpc-terrains)
* [[LPC] Beach / Desert](https://opengameart.org/content/lpc-beach-desert)
* [[LPC] Jungle](https://opengameart.org/content/lpc-jungle)
* [[LPC] Conifers](https://opengameart.org/content/lpc-conifers)
* [[LPC] Trees](https://opengameart.org/content/lpc-trees)
* [[LPC] Flowers / Plants / Fungi / Wood](https://opengameart.org/content/lpc-flowers-plants-fungi-wood)
* [Liberated Pixel Cup (LPC) Base Assets (sprites & map tiles)](https://opengameart.org/content/liberated-pixel-cup-lpc-base-assets-sprites-map-tiles)

*If you are interested by my adventures during the development of Vagabond, you can follow me on [Twitter](https://twitter.com/PierreVigier).*