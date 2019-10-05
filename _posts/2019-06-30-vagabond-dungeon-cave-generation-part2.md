---
layout: post
title: "Vagabond &#8211; Dungeon and cave generation &#8211; Part 2"
date: 2019-06-30
author: pierre
tab: blog
tags: vagabond pcg
---

![](/media/img/vagabond-dungeon-cave-generation-part2/banner.png){: width="400" .center-image .modal-image }

This week, I continued to work on the [dungeon and cave generator]({{ site.baseurl }}{% post_url 2019-06-23-vagabond-dungeon-cave-generation %}). I refined some things I had started last week and then I started generating tiles from the output of the generator.

<!--more-->

# Removing isolated cells

The first thing I did is implementing a flood fill to remove unreachable cells:

![](/media/img/vagabond-dungeon-cave-generation-part2/flood_fill.gif){: width="400" .center-image .modal-image }

# Multiple corridors between rooms

Something really nice that I discovered when playing with the parameters of the generator is that if we make the noise around the rooms touching, interesting things happen.

Here is the difference of results just before the noise of rooms touch each other and just after, the parameter varies only by one unit:

![](/media/img/vagabond-dungeon-cave-generation-part2/0_9_30.png){: width="400" .center-image .modal-image }

![](/media/img/vagabond-dungeon-cave-generation-part2/0_10_30.png){: width="400" .center-image .modal-image }

If we make the rooms a bit larger, it is even more interesting:

![](/media/img/vagabond-dungeon-cave-generation-part2/0_10_00.png){: width="400" .center-image .modal-image }

What is nice is that we have more randomness and nice structures that appear but we still have the graph structure and the notion of rooms that are useful:

![](/media/img/vagabond-dungeon-cave-generation-part2/0_10_00_rooms.png){: width="400" .center-image .modal-image }

# Generating tiles for caves

I spent almost all my time working on the generation of tiles. It is not really difficult but a bit tricky to have everything right.

Here are some results:

![](/media/img/vagabond-dungeon-cave-generation-part2/tiles.gif){: width="400" .center-image .modal-image }

What is really cool is that it is super easy to switch from a rock cavern to a sand cavern or an ice cavern:

![](/media/img/vagabond-dungeon-cave-generation-part2/tilesets.gif){: width="400" .center-image .modal-image }

# Conclusion

I did not do a lot of interesting things this week. Moreover, there was a heatwave, here, in France and it was quite hard to work.

The next steps for the dungeon generation is adding decorations and monsters. I will not do that next week as I will start to work more on the game engine now that I have some interesting things to feed it with.

See you next week for more!

# Credits

* [[LPC] Mountains](https://opengameart.org/content/lpc-mountains)
