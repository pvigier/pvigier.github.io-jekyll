---
layout: post
title: "Vagabond &#8211; Dungeon and cave generation"
date: 2019-06-23
author: pierre
tab: blog
tags: vagabond pcg
---

![](/media/img/vagabond-dungeon-cave-generation/cave_generation.gif){: width="400" .center-image .modal-image }

I am sorry, there was no article last week as I was on vacation.

But, this week I started working on a new topic: dungeon and cave generation. I used space partitioning to generate rooms, maze generation algorithms to generate the corridors and cellular automata to have a more organic look and feel for caves. Let's dive in the details!

<!--more-->

# Space partitioning

There are many ways ([random placement](https://gamedevelopment.tutsplus.com/tutorials/create-a-procedurally-generated-dungeon-cave-system--gamedev-10099), [agent based](http://pcgbook.com/wp-content/uploads/chapter03.pdf), using [separation steering behavior](https://www.reddit.com/r/roguelikes/comments/1dodsv/my_procedural_dungeon_generation_algorithm/) or [a physics engine](https://www.gamasutra.com/blogs/AAdonaac/20150903/252889/Procedural_Dungeon_Generation_Algorithm.php), etc.) to create rooms for a dungeon. But my favorite method is space partitioning because it is controllable and easily expandable.

There are also many ways to split the space: grid partitioning, binary space partitioning, quad tree space partitioning, Voronoi diagrams, etc. I chose to use binary space partitioning this time as it is well suited to generate rectangular rooms. This method was popularized by an [article](http://roguebasin.roguelikedevelopment.org/index.php?title=Basic_BSP_Dungeon_generation) on RogueBasin.

The only difficulty with this algorithm is to choose the split position. Indeed, if we do not set any constraint on the split position, we obtain weird space partitions:

![](/media/img/vagabond-dungeon-cave-generation/bsp_no_constraint.gif){: width="400" .center-image .modal-image }

There are several ways to avoid this behavior. One is to constrain the split to be between two ratios of the side length for instance between 30% and 70% or 40% and 60%. Another method is to use a normal or a binomial distribution instead of a uniform distribution, it is then more likely to split near the center of the side than its extremities. These methods fix the issue but it is hard to understand exactly how the parameters modify the final result.

So I use another method that has the advantage of having only one parameter which is easily understandable: the maximum allowed ratio between the length and the width of cells. When I sample a new split, I firstly compute the minimum value and the maximum value it can take so that the two new cells have their ratio below the limit then I sample uniformly between the two bounds. Here is the result when I vary the maximum allowed ratio:

![](/media/img/vagabond-dungeon-cave-generation/bsp_varying_ratio.gif){: width="400" .center-image .modal-image }

A maximum ratio between 2.0 and 3.0 gives good results:

![](/media/img/vagabond-dungeon-cave-generation/bsp.gif){: width="400" .center-image .modal-image }

# Room generation

The next step is to generate a room inside each cell. There is no special issue here, I just set some constraints so that the rooms are not too small nor too close to the walls of the cells.

Here are the results:

![](/media/img/vagabond-dungeon-cave-generation/rooms.gif){: width="400" .center-image .modal-image }

# Edge selection

Commonly in dungeon generators based on binary space partitioning, the binary tree used during space partitioning is used again for corridor generation. I did not do that because, in my opinion, it is quite limited.

Instead, during space partitioning, I maintain a [half-edge data structure](https://en.wikipedia.org/wiki/Doubly_connected_edge_list) which allows to know which cells are next to each other. Thus, I have graphs like that:

![](/media/img/vagabond-dungeon-cave-generation/graphs.gif){: width="400" .center-image .modal-image }

There are three advantages with this approach. The first one is that if later I want to change the method used for space partitioning, the rest of the generator is still valid as it only takes a half-edge data structure as input. The second one is that I can now use any [maze generation algorithm](https://en.wikipedia.org/wiki/Maze_generation_algorithm) to select the edges that will become corridors. The third one is that if I want to add cycles in the dungeon, I can easily do so.

For now, I simply use Kruskal algorithm with Manhattan distance to select the edges. Here are the results:

![](/media/img/vagabond-dungeon-cave-generation/selected_edges.gif){: width="400" .center-image .modal-image }

# Corridor generation

The next step is the generation of corridors from the selected edges. This is maybe the trickiest part of the generator as we need to be careful that no corridor crosses another one.

Here are the results:

![](/media/img/vagabond-dungeon-cave-generation/corridors.gif){: width="400" .center-image .modal-image }

# Cave generation

The previous results are fine for dungeons, crypts or other man-made structures but I would like a more organic aspect for caves or mines. The classic method to generate caves is to use a cellular automaton as described in [this article](http://www.roguebasin.com/index.php?title=Cellular_Automata_Method_for_Generating_Random_Cave-Like_Levels) on RogueBasin. The big issue with cellular automata is that the results are not really controllable.

My idea is to still use cellular automata to obtain an organic look but I set constraints to have a controllable result. Instead of using only two states: dead or alive, I use four: definitively dead, dead, alive, definitively alive. The "definitively" states cannot change from one step to another, they serve to constrain the result.

The rooms and the corridors we have generated in the previous steps are filled with definitively alive cells. Thus, we are still have a notion rooms and we are sure that they are connected to each other. The edges that were not selected are filled with definitively dead cells to make sure that no new path from one room to another appears. Finally around the rooms and the corridors we randomly set some cells to alive. Here is the initial configuration:

![](/media/img/vagabond-dungeon-cave-generation/cave_initial_configuration.png){: width="400" .center-image .modal-image }

Then we run the cellular automaton to obtain the desired organic look:

![](/media/img/vagabond-dungeon-cave-generation/cellular_automaton.gif){: width="400" .center-image .modal-image }

Here are some more results:

![](/media/img/vagabond-dungeon-cave-generation/caves.gif){: width="400" .center-image .modal-image }

One thing that I will surely implement later is a flood fill to remove unreachable parts.

# Conclusion

This is the first leg of the long journey of building an interesting dungeon generator. I am happy with the results so far. I am especially proud of the constrained cellular automaton method to have controllable and organic caves. I also like the fact that each step of the generation is separated from the others and can be modified independently.

The next step is to [generate tiles]({{ site.baseurl }}{% post_url 2019-06-30-vagabond-dungeon-cave-generation-part2 %}) from the output of this generator to have much more appealing dungeons.

See you next week for more!
