---
layout: post
title: "Vagabond &#8211; 2D Light System"
date: 2019-07-28
author: pierre
tab: blog
tags: vagabond game-engine
---
This week, I have worked on a light system. It will allow me to simulate a day-night cycle and to create a nice atmosphere during night or inside dungeons and caves.

![](/media/img/vagabond-2d-light-system/banner.png){: width="400" .center-image .modal-image }

<!--more-->

# The Big Picture

My light system uses two textures.

On the first one, I draw all the visible objects without any particular processing.

The other one is cleared with the ambient light color. Then, I draw the contribution of each light one by one with [additive blending](https://learnopengl.com/Advanced-OpenGL/Blending).

Finally, I draw the lighting texture on the first one with multiplicative blending.

Here is an animation to sum up the process (sorry for the compression artifacts):

![](/media/img/vagabond-2d-light-system/big_picture.gif){: width="400" .center-image .modal-image }

# Light Shading

For now, my light system only supports point lights.

A point light is defined by the position of its center, a radius, a color and an intensity.

For each point light that is visible on screen, I draw a square centered on the light of size $$2r$$, where $$r$$ is the radius, and use a shader to set the color of each pixel in the square.

The next step is to choose a formula to describe the color received by a point from the light.

As a point light emits the same light in all the directions, the color received only depends on the distance. Hence, we can use a formula which looks like that:

$$
C(d) = attenuation(d) \times I \times C_{light}
$$

where $$I$$ is the intensity, $$C_{light}$$ the color of the light and $$attenuation$$ a mapping such that $$attenuation(0) = 1$$ and $$attenuation(d) = 0$$ for $$d > r$$.

My first try was to use a [well-known formula](http://wiki.ogre3d.org/tiki-index.php?page=-Point+Light+Attenuation) for the attenuation of 3D point lights:

$$
attenuation(d) = \frac{1}{1 + K_l \times d + K_q \times d^2}
$$

Unfortunately, it does not work well as the center is bright but the luminosity decreases quickly. Moreover, it is never zero thus the radius of the light must be really large compared to the useful lit area in order to not see the seam at the border of the square. Here is an example where the seam is visible:

![](/media/img/vagabond-2d-light-system/realist_glitch.png){: width="400" .center-image .modal-image }

Here is another example with a point light of same radius, intensity and color but I tweaked the parameters $$K_l$$ and $$K_q$$ so that the seam is invisible. However, the lit area is then even smaller:

![](/media/img/vagabond-2d-light-system/realist.png){: width="400" .center-image .modal-image }

Consequently, I looked for other formulas, that are maybe less physically realist, but that gives better results.

I start with a simple linear attenuation:

$$
attenuation(d) = \max(0, 1 - \frac{d}{r})
$$

What is nice with this formula is that the luminosity is zero for points outside of the disk of radius $$r$$. However, in my opinion, the attenuation is a bit brutal near the border:

![](/media/img/vagabond-2d-light-system/linear.png){: width="400" .center-image .modal-image }

To obtain a smoother attenuation, I simply square the previous formula to obtain a quadratic attenuation:

$$
attenuation(d) = \max(0, 1 - \frac{d}{r})^2
$$

I find the results much more appealing:

![](/media/img/vagabond-2d-light-system/quadratic.png){: width="400" .center-image .modal-image }

Finally, I advise to use a texture that supports [HDR](https://learnopengl.com/Advanced-Lighting/HDR) with tone mapping. Otherwise, it is likely that the colors will saturate as additive blending is used.

# Shadows

For now, we do not take into account any obstacle for the light. We simply draw a square around the light.

To be able, to take into account walls and have shadows, I use a more complex shape. I compute the set of points that are visible for the light center, this set is called the [visibility polygon](https://en.wikipedia.org/wiki/Visibility_polygon).

There are some good resources on the web that describe algorithms to compute the visibility polygon from naive methods to more elaborate ones. Here are two articles that I found useful to gain some intuition: [here](https://www.redblobgames.com/articles/visibility/) and [there](https://ncase.me/sight-and-light/).

Here is a small demo of my implementation:

<video controls width="400">
    <source src="/media/video/vagabond-2d-light-system/visibility_polygon.mp4" type="video/mp4">
    Sorry, your browser doesn't support embedded videos.
</video>

I may write a separate article describing my implementation of an $$O(n\log{n})$$ algorithm to compute the visibility polygon. A bit like what I did for [Fortune's algorithm]({{ site.baseurl }}{% post_url 2018-11-18-fortune-algorithm-details %}), but with even more details as this algorithm is simpler.

Once we have computed the visibility polygon, we just have to render it with the shader to obtain shadows:

<video controls width="400">
    <source src="/media/video/vagabond-2d-light-system/lights_and_walls.mp4" type="video/mp4">
    Sorry, your browser doesn't support embedded videos.
</video>

Here is an example in a cave:

<video controls width="400">
    <source src="/media/video/vagabond-2d-light-system/light_cave.mp4" type="video/mp4">
    Sorry, your browser doesn't support embedded videos.
</video>

Here is another video in a cave but this time, I display all the walls in red, the walls that intersect with the light bounding box in yellow and the visibility polygon in green:

<video controls width="400">
    <source src="/media/video/vagabond-2d-light-system/light_cave_debug.mp4" type="video/mp4">
    Sorry, your browser doesn't support embedded videos.
</video>

# Conclusion

That is all for this article on my light system. I may blur the lighting texture to have smoother transitions to shadows. I may also add other types of lights such as spotlights and area lights later.

Next week, I will work on the [physics system]({{ site.baseurl }}{% post_url 2019-08-04-quadtree-collision-detection %}).

See you next week for more!