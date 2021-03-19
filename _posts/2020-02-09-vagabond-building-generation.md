---
layout: post
title: "Vagabond &#8211; Building Generation"
date: 2020-02-09
author: pierre
tab: blog
tags: vagabond pcg
---
Hi everyone!

These last weeks, I had been working on building generation. I want to share with you the main ideas of my method, some insights, and my results!

Here is a quick glimpse of the overall process:

![](/media/img/vagabond-building-generation/building_generation.gif){: .center-image .modal-image }

<!--more-->

# Floor Plan Generation

The first step is to generate the floor plan that will be the input for the next steps.

There were a lot of trials and errors at this step. I wanted something flexible, that is, the generator takes as input a list of rooms that the building must contain and it generates a floor plan, whatever the number of rooms or their shape is. Thus, we cannot rely on some patterns to arrange the rooms. Moreover, I wanted the generator to be robust and always return good enough shapes.

I decided to use an incremental method: to place the rooms one by one.

My first attempt was to maintain the frontier of the building as a list of edges. Then, at each step, I would select an edge and grow a new room from it. The size of the new room was randomly generated. It was a terrible idea, it creates a lot of holes in the building shapes.

![](/media/img/vagabond-building-generation/floor_plans_edges.gif){: width="400" .center-image .modal-image }

My second attempt to prevent the strange holes from occuring was to use a grid to place the rooms. A room would occupy exactly one cell and the frontier is now the set of cells that are neighbors to the already selected cells. Again, at each step, we select a cell in the frontier and add a room in this cell. But to have more variety, I chose to have columns with variable widths and rows with variable heights. It was a terrible idea, again. The issue, this time, is that all the rooms in the same column have the same width, and all the rooms in the same row have the same height, it was a bit weird. Moreover, the topology of the building was too simple for my liking.

![](/media/img/vagabond-building-generation/floor_plans_variable.gif){: width="400" .center-image .modal-image }

In my third attempt, I decided to fix the size of columns and rows, but the rooms can now occupy several cells which will allow more complex topologies. I think this is a good trade-off: we lose a bit of variety on the room shapes but having predictable room sizes will ease the next steps. Moreover, I find this solution particularly clean and simple to implement. I am still wondering why this was not the first thing I tried.

![](/media/img/vagabond-building-generation/floor_plans.gif){: width="400" .center-image .modal-image }

I want to give a bit more insight into how a cell is chosen from the frontier. If you select a random cell, there may be a collision problem (if the room takes several cells) and after several steps, you may have something very weird. To prevent that, all the cells in the frontier are tried, we check for collisions and a score is attributed to the configuration. Then, I randomly pick a cell among the cells that have the highest scores.

One question remains: how to determine the score? Well, in the beginning, I tried to quantify the beauty of a floor plan, and it was hard to find good criteria. Then, I changed my way of thinking instead of trying to select the most beautiful floor plans, I penalize ugly and weird floor plans. And, it reveals to be way simpler to craft a score function that penalizes weird floor plans than one that finds the beautiful ones. I mainly penalize two things: extreme ratios, I don't want too flattened buildings, and holes. Just with these two criteria, most weird building shapes are ruled out and the ones that are kept are at least correct.

# Interior Generation

Now, that we have floor plans, we can generate floors and walls. As the generator is for a 2D game, I have to take care of the projection and let enough place between rooms to be able to put the walls.

I can assign different tiles for the floor and walls of the rooms. But for now, they are uniform for the whole building.

![](/media/img/vagabond-building-generation/interiors.gif){: width="400" .center-image .modal-image }

# Exterior Generation

Oddly, this was the first thing, I implemented. Even before I finished floor plan generation. Surely because I thought it was one of the easiest steps. In fact, there are some subtleties so that everything works well with 2D tiles.

One preprocessing step that reveals useful is to merge rooms to have larger rectangle parts. To achieve that, I used a [greedy meshing](https://0fps.net/2012/06/30/meshing-in-a-minecraft-game/) algorithm.

![](/media/img/vagabond-building-generation/building_parts.gif){: width="400" .center-image .modal-image }

## Wall Generation

Nothing really difficult there. I just draw the bottom walls of building parts.

![](/media/img/vagabond-building-generation/walls.gif){: width="400" .center-image .modal-image }

## Roof Generation

Roofs are more challenging. I studied the different types of roofs a bit. For now, two of the simplest types, flat roofs and mansard roofs, are supported.

![](/media/img/vagabond-building-generation/roofs.gif){: width="400" .center-image .modal-image }

I also have an experimental implementation of hip roof generation. Hip roofs are more complex because the tiling depends on the width and you must have a special case for the transition between two building parts. I will finish that later.

![](/media/img/vagabond-building-generation/hip_roof.png){: width="400" .center-image .modal-image }

# Room Generation

Placing objects in the rooms is by far the most interesting part of this generator.

The first step is to define object groups, it may be one object alone or objects that should always be placed together, for instance, a table with chairs. For each object group, I will also associate a collision box and a margin box. The collision box corresponds to the tiles used by the object group while the margin box corresponds to the tiles that must be free around the group to be able to interact with the objects in the game.

Here you can visualize the collision box and the margin box for some objects:

![](/media/img/vagabond-building-generation/object_boxes.png){: width="400" .center-image .modal-image }

Then, for each room definition, I make a difference between objects that are necessary and those that are optional. Necessary objects must be placed in the room, otherwise, the room is invalid. For example, there must be a bed in a bedroom, if we fail to place one there, then the generator fails. On the contrary, decorations, such as a pot with a plant, are optional and if we fail to place them, the room is still valid.

Besides, for each object group in the room, constraints can be assigned to it. Some constraints I implemented are: distance to a wall, in a corner of the room or horizontally centered in the room.

To place, necessary objects, I use a [CSP](https://en.wikipedia.org/wiki/Constraint_satisfaction_problem) solver I have designed especially for this problem. In particular, it can check very quickly for collisions between objects and that the room is connected, that is, we can access to all objects and doors in the room. My CSP solver uses a simple [backtracking](https://en.wikipedia.org/wiki/Backtracking) algorithm.

One important thing, for procedural generation, is to keep the solver "non-deterministic" i.e. it does not always return the same solution. Otherwise, it would be boring. To achieve that, I simply shuffle the domains, the sets of positions tried for each object group.

On the contrary, for optional objects, I do not use the CSP solver. This allows keeping the problems relatively small and easy for the solver and thus to solve them very quickly. For each optional object, I just pick randomly one of its valid position and if there are none, it is not placed and I try the next object.

Here are some living rooms generated with the same set of parameters:

![](/media/img/vagabond-building-generation/living_rooms.gif){: width="400" .center-image .modal-image }

And some buildings with objects:

![](/media/img/vagabond-building-generation/objects.gif){: width="400" .center-image .modal-image }

For now, there are only three different types of rooms: a bedroom, a kitchen and a living room. I will design more later.

# Conclusion

Let us look at some buildings generated by the generator to conclude this blog post:

![](/media/img/vagabond-building-generation/buildings.gif){: .center-image .modal-image }

The whole process is quite fast, under 1ms for a whole building, it will allow me to generate hundreds to thousands of buildings during world generation!

Most of the ideas described here are very simple, but as we say, the devil is in the detail, and there were a lot of details there!

Now, that we have buildings, the next step is obviously to generate cities. It is one of the last steps before the release of the alpha version. I am so excited to eventually share my work with people! :)

All the assets used in this post were made by [OpenGameArt](https://opengameart.org/) artists, you can find them [there](https://opengameart.org/content/vagabonds-assets).

See you next time for more!