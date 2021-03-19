---
layout: post
title: "Simulopolis Alpha 3 Is Released"
date: 2018-11-03
author: pierre
tab: blog
tags: simulopolis
---

Today, I release the third alpha version of Simulopolis.

In this post, I will detail a bit what's new since the [initial release]({{ site.baseurl }}{% post_url 2018-10-06-simulopolis-alpha-1 %}).

![Cover](/media/img/simulopolis-alpha-3/newspaper.png){: .center-image .modal-image }

On this screenshot, you can saw some minor changes on the interface and the main new feature: the newspaper.

<!--more-->

# Interface

I worked a lot on the user interface. In alpha 1, the window's resolution was 800x600 and it was not possible to resize the window which makes the experience very painful on some screen.

Thus, in alpha 2, I made the window resizable and I improved the GUI code to make widgets resizable too. It was a lot of work, but I am very satisfied with the result. The GUI code starts to look like a small library and I am thinking about separating this code from the rest of the game to publish it as a library.

In alpha 3, I continued to improve the interface by fixing some small issues. In addition, in the city creation screen, I prefilled the city name input field with a randomly chosen city name. The list of city names is based on this Wikipedia's [list of mythological places](https://en.wikipedia.org/wiki/List_of_mythological_places).

Finally, I implemented a new GUI widget called `GuiText` which allows to display text on several lines and to choose its alignment (left, right, centered, justified). This widget was particularly useful to implement the newspaper window.

# Newspaper

The big new feature of alpha 3 is the newspaper. Many people complained about the lack of tutorial. They found difficult to figure what to do. And they are totally right!

The newspaper is my answer to this. It will be the place when players will find advices on what to do and feedbacks on what they do.

For now, the newspaper is still a rough draft. It only shows a message when the player creates a new city. For this version, I focus on the design of the newspaper. To give it a look like *The New York Times* or *The Washington Post*, I use a [gothic font](https://www.dafont.com/julia-black.font) for the title. For the articles, I use a nice [serif font](https://www.dafont.com/linux-libertine.font).

However, in the future, there should be a new edition of the newspaper each month with articles about the city: what's new, problems, etc.  Moreover, there will be some procedurally generated articles, but I keep that for another blog post

If you want to test the new version, it is available on [itch.io](https://pvigier.itch.io/simulopolis) for free.

See you!
