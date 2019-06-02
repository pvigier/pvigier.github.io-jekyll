---
layout: post
title: "Vagabond &#8211; Map generation"
date: 2019-05-12
author: pierre
tab: blog
tags: vagabond pcg
---

![](/media/img/vagabond-map-generation/0/Biome.png){: width="400" .center-image .modal-image }

In this first devlog about Vagabond, I am going to talk about the map generation algorithm I have started to design. Indeed, in Vagabond all the maps will be procedurally generated. 

Firstly, I will list some of the constraints that will influcence the design of the map generation pipeline:

* It must be fast as I do not want the player to wait for too long during map generation.
* It must not be totally random as Vagabond is a RPG, the world must be coherent.
* It must generate the entire map at once as different part of the world will interact.
* It must return a data structure easy to manipulate to be able to add more elements later such as rivers, cities, roads, dungeons, etc.

<!--more-->

# Generating cells

The first step is to divide the world into cells. I like the idea popularized by [Amit Patel](http://www-cs-students.stanford.edu/~amitp/game-programming/polygon-map-generation/) of using Voronoi cells. Thus I randomly generate a bunch of points and compute the [Voronoi diagram](https://en.wikipedia.org/wiki/Voronoi_diagram) using my own implementation of [Fortune's algorithm](https://en.wikipedia.org/wiki/Fortune%27s_algorithm). I talked about it in this [article]({{ site.baseurl }}{% post_url 2018-11-18-fortune-algorithm-details %}) but I have revamped the code and fixed many edge cases to turn it into a library called [MyGAL](https://github.com/pvigier/MyGAL).

Here are the results with 1024 points. For each step, I will show the result for four seeds:

{: .image-table }
![](/media/img/vagabond-map-generation/0/Voronoi.png){: width="250" .modal-image } | ![](/media/img/vagabond-map-generation/3/Voronoi.png){: width="250" .modal-image }
:---:|:---:
![](/media/img/vagabond-map-generation/5/Voronoi.png){: width="250" .modal-image } | ![](/media/img/vagabond-map-generation/7/Voronoi.png){: width="250" .modal-image }

Then to have something a bit less random, I apply [Lloyd's relaxation](https://en.wikipedia.org/wiki/Lloyd%27s_algorithm) twice:

{: .image-table }
![](/media/img/vagabond-map-generation/0/Voronoi_relaxed.png){: width="250" .modal-image } | ![](/media/img/vagabond-map-generation/3/Voronoi_relaxed.png){: width="250" .modal-image }
:---:|:---:
![](/media/img/vagabond-map-generation/5/Voronoi_relaxed.png){: width="250" .modal-image } | ![](/media/img/vagabond-map-generation/7/Voronoi_relaxed.png){: width="250" .modal-image }

# Height

The next step is to assign a height to each cell to determine if a cell will be land or sea.

I start by generating some fractal noise using a minimalist library I wrote called [noisette](https://github.com/pvigier/noisette):

{: .image-table }
![](/media/img/vagabond-map-generation/0/Height_noise.png){: width="250" .modal-image } | ![](/media/img/vagabond-map-generation/3/Height_noise.png){: width="250" .modal-image }
:---:|:---:
![](/media/img/vagabond-map-generation/5/Height_noise.png){: width="250" .modal-image } | ![](/media/img/vagabond-map-generation/7/Height_noise.png){: width="250" .modal-image }

Then I scale the noise between 0 and 1 and multiply by a gaussian kernel centered on the center of the map to then obtain an island roughly circular and centered in the middle of the world.

Finally, I set the height of cells on the frontier of the world to zero to ensure that these cells will be seas later. And I redistribute the values so that half of the cells have a height smaller than 0.5 and the other half higher as 0.5 will be the threshold to be land:

{: .image-table }
![](/media/img/vagabond-map-generation/0/Height.png){: width="250" .modal-image } | ![](/media/img/vagabond-map-generation/3/Height.png){: width="250" .modal-image }
:---:|:---:
![](/media/img/vagabond-map-generation/5/Height.png){: width="250" .modal-image } | ![](/media/img/vagabond-map-generation/7/Height.png){: width="250" .modal-image }

# Temperature

The biome of a cell will be determined by its temperature and its moisture. Thus we need to generate both.

For temperature, I did not want something too much random as it does not feel right to me if there is a glacier next to a desert. Instead, I decided to generate temperatures so that it is cooler in the north of the island and warmer in the south as it is the case for countries in the north hemisphere of Earth. It is also the case for some fantasy worlds such as Westeros, the main continent of Game of Thrones where the North is cold and the southern regions are warmer.

To do that, I just [warp](https://iquilezles.org/www/articles/warp/warp.htm) a vertical linear gradient with some Perlin noise:

{: .image-table }
![](/media/img/vagabond-map-generation/0/Temperature_noise.png){: width="250" .modal-image } | ![](/media/img/vagabond-map-generation/3/Temperature_noise.png){: width="250" .modal-image }
:---:|:---:
![](/media/img/vagabond-map-generation/5/Temperature_noise.png){: width="250" .modal-image } | ![](/media/img/vagabond-map-generation/7/Temperature_noise.png){: width="250" .modal-image }

And here is the temperature of each cell (cold is blue and hot is red):

{: .image-table }
![](/media/img/vagabond-map-generation/0/Temperature.png){: width="250" .modal-image } | ![](/media/img/vagabond-map-generation/3/Temperature.png){: width="250" .modal-image }
:---:|:---:
![](/media/img/vagabond-map-generation/5/Temperature.png){: width="250" .modal-image } | ![](/media/img/vagabond-map-generation/7/Temperature.png){: width="250" .modal-image }

<!--image-->

# Precipitation

Then, we need to generate precipitation. I could have used a model which takes into account the height of each cell and wind as presented [here](https://azgaar.wordpress.com/2017/05/08/river-systems/) and [here](https://heredragonsabound.blogspot.com/2016/10/is-it-windy-in-here.html).

But I wanted to start with something really simple so I just use raw Perlin noise again. I do not use fractal noise this time because I did not want the precipitation to vary too quickly:

{: .image-table }
![](/media/img/vagabond-map-generation/0/Precipitation_noise.png){: width="250" .modal-image } | ![](/media/img/vagabond-map-generation/3/Precipitation_noise.png){: width="250" .modal-image }
:---:|:---:
![](/media/img/vagabond-map-generation/5/Precipitation_noise.png){: width="250" .modal-image } | ![](/media/img/vagabond-map-generation/7/Precipitation_noise.png){: width="250" .modal-image }

And here is the precipitation of each cell (white is no precipitation and blue a lot):

{: .image-table }
![](/media/img/vagabond-map-generation/0/Precipitation.png){: width="250" .modal-image } | ![](/media/img/vagabond-map-generation/3/Precipitation.png){: width="250" .modal-image }
:---:|:---:
![](/media/img/vagabond-map-generation/5/Precipitation.png){: width="250" .modal-image } | ![](/media/img/vagabond-map-generation/7/Precipitation.png){: width="250" .modal-image }

Maybe I will use a more physically accurate model later but it seems sufficient for now.

# Biome

At the moment, I have 8 different biomes:

* Desert
* Grassland
* Forest
* Dark forest
* Jungle
* Snow
* Toundra
* Scorched

I assign each of them using a [Whittaker diagram](http://w3.marietta.edu/~biol/biomes/biome_main.htm). Here are the final results:

{: .image-table }
![](/media/img/vagabond-map-generation/0/Biome.png){: width="250" .modal-image } | ![](/media/img/vagabond-map-generation/3/Biome.png){: width="250" .modal-image }
:---:|:---:
![](/media/img/vagabond-map-generation/5/Biome.png){: width="250" .modal-image } | ![](/media/img/vagabond-map-generation/7/Biome.png){: width="250" .modal-image }

# Useful links

Before finishing this article, I must cite some other map generators that inspired me:

* [Fantasy Map Generator](https://azgaar.wordpress.com) by Azgaar
* [Generating fantasy maps](http://mewo2.com/notes/terrain/) by Martin O'Leary
* [Here Dragons Abound](https://heredragonsabound.blogspot.com) by Scott Turner
* [Polygonal Map Generation for Games](http://www-cs-students.stanford.edu/~amitp/game-programming/polygon-map-generation/) by Amit Patel

The next steps are to generate the rivers, the regions and placing some cities. See you [next week]({{ site.baseurl }}{% post_url 2019-05-19-vagabond-borders-rivers-cities-roads %}) for more!

*If you are interested by my adventures during the development of Vagabond, you can follow me on [Twitter](https://twitter.com/PierreVigier).*