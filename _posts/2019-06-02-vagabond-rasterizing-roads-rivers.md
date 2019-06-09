---
layout: post
title: "Vagabond &#8211; Rasterizing roads and rivers"
date: 2019-06-02
author: pierre
tab: blog
tags: vagabond pcg
---

![](/media/img/vagabond-rasterizing-roads-rivers/roads_rivers.png){: width="400" .center-image .modal-image }

This week, I finished the rasterization of polygonal maps that I started to implement [last week]({{ site.baseurl }}{% post_url 2019-05-26-vagabond-generating-tiles %}). Then, I improved my renderer to be able to display worlds way larger than before. Finally, I started to fill the biomes with some decorations and vegetation.

<!--more-->

# Rasterizing rivers

The first thing I did this week was to rasterize the rivers. I reuse my line rasterization algorithm. But instead of setting only one tile per coordinates output by the algorithm, I rasterize a whole square around the coordinates. It is as if I used a brush in a graphics editor such as GIMP or Photoshop.

Moreover, I would like the rivers to be wider when many tributaries meet. So I just use a wider brush when the rivers should be larger. The width of the rivers grows as the square root of the number of tributaries.

Here are the results:

![](/media/img/vagabond-rasterizing-roads-rivers/Rivers.gif){: width="400" .center-image .modal-image }


# Rasterizing roads

Rasterizing roads was harder. Indeed, there are several issues I must take care of:

* Roads should avoid sea. If you [remember well]({{ site.baseurl }}{% post_url 2019-05-19-vagabond-borders-rivers-cities-roads %}), a road segment joins two sites of Voronoi cells. But if we draw a straight between the sites, it may go through sea as shown on the image below.
* When a road crosses a river, there should be a bridge. But a bridge should be straight (totally vertical or totally horizontal).

I use a smaller number of cells so that we can clearly observe that the roads go through the sea:

![](/media/img/vagabond-rasterizing-roads-rivers/roads_issues.png){: width="400" .center-image .modal-image }

I use A* algorithm to find paths between two sites that fulfill all the mentioned constraints. Here is the macroscopic result:

![](/media/img/vagabond-rasterizing-roads-rivers/Roads.png){: width="400" .center-image .modal-image }

 And here are some of the generated bridges:

![](/media/img/vagabond-rasterizing-roads-rivers/Bridges.gif){: width="400" .center-image .modal-image }

You can see that there are still some problems with the rasterization. I did not fix that yet because I am not happy with the method I used. While A* algorithm outputs a path that is valid, it is a list of coordinates which is not easy to work with. In particular, it is not appropriate to place inns or small villages along the roads. Or even for NPCs to follow the roads.

The fundamental reason is that the description of the road is given in the world space (made of tiles) by A* star algorithm. However for things like village placement, we would like to use the map space (made of shapes) which is more adapted.

Thus, I think I will use another method which will not necessarily give the shortest path but something more easy to manipulate. I am currently thinking about using distorted lines as I used for rivers and cell borders or Bezier curves which is a common technique to model roads.

# Bigger worlds

Usually, rendering is not a big issue for tile-based video games as only a small portion of the world is displayed. But in my editor/generator, I want to be able to explore the entire world. For small worlds, there is no problem with the naive approach of drawing a grid of tiles. Indeed, for 600x600x4 (I use 4 layers of tiles to display the terrains) worlds which means 1 440 400 tiles, my laptop has no problem. But for 3000x3000x4 worlds which means 36 000 000 tiles the application slows down seriously.

The solution I chose was to simplify the geometry. To do that, I use a popular technique in the voxel game development community: [greedy meshing](https://0fps.net/2012/06/30/meshing-in-a-minecraft-game/). The idea is quite simple: you just merge as many as possible similar adjacent tiles to reduce the number of faces. With greedy meshing, it takes less than one million faces to display 3000x3000x4 worlds, it is a big improvement!

If you want to get the impression of what a 3000x3000 world looks like, here is a small animation:

![](/media/img/vagabond-rasterizing-roads-rivers/Big.gif){: width="400" .center-image .modal-image }

# Filling the biomes

Finally, I started to work on filling the cells that are currently so empty. The goal is not to add points of interest such as ruins, dungeon entries or small villages but to fill the cells with some monotonous decorations and vegetation.

I use a popular method in the procedural generation community to do object placement called Poisson disk sampling. I like it because it is simple, fast and controllable. I have put some links at the end of this article, If you want to know more.

Here is an example of points generating by Poisson disk sampling:

![](/media/img/vagabond-rasterizing-roads-rivers/poisson_disk_sampling.png){: width="400" .center-image .modal-image }

By the way, if you are a C++ developer and are looking for a good pseudorandom number generator, I advise you to use the [PCG](http://www.pcg-random.org/) generators which are compatible with the standard random library, faster than standard PRNGs and produce high quality random numbers.

Then I map each point to an object, here is the result for the desert biome:

![](/media/img/vagabond-rasterizing-roads-rivers/Desert.gif){: width="400" .center-image .modal-image }

I have not done the other biomes yet.

# Conclusion

That's all for this week. I spent a lot of time refactoring my code to be able to add new features. A lot of things such as biomes and terrains were hardcoded in the generator. But when the moment of adding decorations and vegetation arrived it was not manageable anymore thus I moved all the data to XML files.

The next steps are refining the object placement algorithm and adding decorations and vegetation for other biomes.

See you [next week]({{ site.baseurl }}{% post_url 2019-06-09-vagabond-forest-generation %}) for more!

# Useful links

* [Meshing in a Minecraft Game](https://0fps.net/2012/06/30/meshing-in-a-minecraft-game/) by 0fps
* [Poisson Disk Sampling](https://web.archive.org/web/20150817064406/http://devmag.org.za/2009/05/03/poisson-disk-sampling/) by Herman Tulleken
* [Fast Poisson Disk Sampling in Arbitrary Dimensions](https://www.cs.ubc.ca/~rbridson/docs/bridson-siggraph07-poissondisk.pdf) by Robert Bridson

*If you are interested by my adventures during the development of Vagabond, you can follow me on [Twitter](https://twitter.com/PierreVigier).*