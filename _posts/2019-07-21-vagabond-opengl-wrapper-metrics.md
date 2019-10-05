---
layout: post
title: "Vagabond &#8211; OpenGL wrapper and metrics"
date: 2019-07-21
author: pierre
tab: blog
tags: vagabond game-engine
---
This week, I have implemented a metric manager to easily report metrics. Then I have added features to my graphics engine. In particular, I have coded a wrapper around OpenGL and used it to finally render the procedural generated worlds in the game.

![](/media/img/vagabond-opengl-wrapper-metrics/world_metrics.png){: width="400" .center-image .modal-image }

<!--more-->

# Metrics

My metric manager stores data for registered metrics and allows to query statistics on these metrics. Currently, it stores values for the last 600 frames which corresponds roughly to ten seconds.

It supports few operations, I can:

* register a new metric
* push a new value for a metric
* get the values for a metric
* get the minimum, maximum and the mean over the window for a metric

To store the values of a metric, I use a circular buffer. To maintain, the mean of a metric over the sliding window, there is a very simple constant time algorithm: we maintain the sum of all the values by adding the new value and subtracting the one that gets popped and then we divide the sum by the number of values.

It is more interesting to maintain the minimum and the maximum in amortized constant time. The idea is to use a data structure that can pop at both extremities and append in constant time or amortized constant time. Then, when a new value is pushed, we pop the first value and remove at the end of the container all the values that are greater (resp. lower) than the new value because they will never be a minimum (resp. maximum). Finally, we can remark that the container is sorted in ascending (resp. descending) order, so the current minimum (resp. maximum) is the first element. It is in amortized constant time as all elements are added and removed exactly twice.

If you want more details, you can read [this article](https://people.cs.uct.ac.za/~ksmith/articles/sliding_window_minimum.html).

# OpenGL wrapper

SFML is a fantastic library but it uses the legacy OpenGL API to implement its graphics features. Thus, it does not give access to the more recent features such as [array textures](https://www.khronos.org/opengl/wiki/Array_Texture) or custom [vertex specification](https://www.khronos.org/opengl/wiki/Vertex_Specification) that I find really useful.

I tried to do something simple and clean. I drew inspiration from SFML for the API but SFML is written in old C++ so they can not use `move` constructors and assignments and consequently it was harder to write nice classes that follow [RAII](https://en.wikipedia.org/wiki/Resource_acquisition_is_initialization). I tried to use a more modern approach.

I started by writing object-oriented wrappers for [VAO](https://www.khronos.org/opengl/wiki/Vertex_Specification#Vertex_Array_Object) and [VBO](https://www.khronos.org/opengl/wiki/Vertex_Specification#Vertex_Buffer_Object). The classes respect RAII idiom and are only movable not copyable like a `std::unique_ptr`.

I did the same for the texture class. My API is similar to the [SFML's one](https://www.sfml-dev.org/documentation/2.5.1/classsf_1_1Texture.php). It has methods `loadFromFile` and `loadFromImage` which create a simple 2D texture and `loadFromFiles` and `loadFromImages` which create an array texture.

That is all for the low-level classes. Then, I created a template class `Mesh` where the template argument is the vertex type. This class inherits from [`sf::Drawable`](https://www.sfml-dev.org/documentation/2.5.1/classsf_1_1Drawable.php) thus it is fully compatible with the rest of SFML framework. The design of this class is inspired from [`sf::Sprite`](https://www.sfml-dev.org/documentation/2.5.1/classsf_1_1Sprite.php) and [`sf::VertexArray`](https://www.sfml-dev.org/documentation/2.5.1/classsf_1_1VertexArray.php). It has a pointer to a shader and a texture (mine, not SFML's one), a list of vertices, a list of indices, two VBOs (one for vertices and one for indices) and a VAO.

Finally, I made a specialization, called `TileMesh`, of my `Mesh` class for meshes made of tiles with a special vertex type. Its vertex type, in addition of `u` and `v` coordinates which are used to specify a point on the texture, has a third coordinate to choose the texture in the array texture.

It is very useful to be able to use several textures in the same mesh. Texture atlases are useful but a bit limited as a 1024x1024 texture can only store 1024 tiles (32x32) and a 2048x2048 texture, 4096 tiles. It is especially useful for an open world where a lot of different tiles can be expected anywhere.

Moreover, I noticed that my custom class using modern OpenGL features is twice faster than using a `sf::VertexArray` for the same geometry (I benchmarked with a very large mesh with several hundreds of thousands of vertices).

The only drawback is that I require OpenGL 3.3. But nowadays, it is really a low requirement.

# Conclusion

That is all for this week. I am happy that I can finally display the world and move inside it.

Next, I will continue to work on the graphics engine and in particular on the [lighting system]({{ site.baseurl }}{% post_url 2019-07-28-vagabond-2d-light-system %}).

See you next week for more!