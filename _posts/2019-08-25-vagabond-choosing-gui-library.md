---
layout: post
title: "Vagabond &#8211; Choosing a GUI library"
date: 2019-08-25
author: pierre
tab: blog
tags: vagabond game-engine
---

The goal of this week was to find a GUI library for the in-game interface. While making some custom widgets may seem easy, writing a general, production-ready, full-fledged GUI library is notoriously difficult. Thus there are few open-source and well-maintained GUI libraries that work with OpenGL or SFML.

In this article, I will enumerate the alternatives I considered and give some details about them. Finally, I will select one and explain my choice.

![](/media/img/vagabond-choosing-gui-library/Vagabond_Menu.png){: width="400" .center-image .modal-image }

<!--more-->

# Writing my own library

The first alternative is not using an external library but writing my own. I did that for my last (unfinished) game Simulopolis.

![](/media/img/vagabond-choosing-gui-library/Simulopolis_GUI.png){: width="400" .center-image .modal-image }

I developed a certain number of widgets. But it was exhausting and took me a lot of time. Thus, I would prefer to use an external library to save some time and have something of better quality.

Especially because I think that a bad interface can make a game shitty but a great interface rarely makes a game great. In other words, a good interface is something expected by players and it is not where a game can make differences. So I would prefer to spend my time on other parts.

# SFML libraries

Two GUI libraries have been made specially to work with SFML: [SFGUI](https://github.com/TankOs/SFGUI) and [TGUI](https://github.com/texus/TGUI). Thus, it is very easy to integrate them in an SFML application as they natively support SFML events.

## SFGUI

SFGUI supports a large range of widgets as you can see in the picture below. It is very simple to build and to link in an existing SFML project.

![](/media/img/vagabond-choosing-gui-library/SFGUI.png){: width="400" .center-image .modal-image }

However, the major drawback is that the interface does not seem to be skinnable. Thus, it is, in my opinion, not suitable for in-game interfaces but it may be for debug/development interfaces.

## TGUI

The other one, TGUI, is as easy to build and to link. It also provides a lot of widgets as you can see below:

![](/media/img/vagabond-choosing-gui-library/TGUI.png){: width="400" .center-image .modal-image }

Contrary to SFGUI, TGUI is skinnable so it is more suitable for in-game interfaces. I also prefer TGUI API, I find it more modern. However, the default theme of SFGUI is cleaner.

Overall, both libraries are easy to build, install and use. But I think they are not as developed as the next libraries I am going to present, yet. Moreover, they are actively maintained thus they may become suitable for production very soon.

# OpenGL libraries

In this section, I will present libraries that are available for several renderers and in particular with OpenGL. It is an ecosystem that has not changed very much for years. Most libraries are not maintained anymore.

The most notable change is the emergence of [immediate mode GUI](https://en.wikipedia.org/wiki/Immediate_Mode_GUI) libraries such as [Dear ImGui](https://github.com/ocornut/imgui). But most of them are not skinnable, the exception is [nuklear](https://github.com/vurtun/nuklear) which is skinnable but it is a C library. Moreover, I feel that this paradigm is more appropriate for development GUI than in-game interfaces.

It remains the two usual suspects: [CEGUI](https://bitbucket.org/cegui/cegui/) and [MyGUI](https://github.com/MyGUI/mygui).

## CEGUI

I will be honest, I have already tried to use CEGUI before but my experiences were not very successful. I tried to give it a new chance as it is widely acknowledged as the most powerful GUI library. Moreover, it has been used by commercial games such as [Torchlight](https://www.torchlight2.com).

I succeeded in building the library and the samples but I failed to run the samples because GLFW failed to open a window. I investigated a bit but I was not able to make it work.

CEGUI may be a powerful GUI library but unfortunately, it is not very user-friendly: it is hard to make it work and the documentation is a bit obscure. It is maybe easier to use it with Ogre as renderer.

## MyGUI

The last library is MyGUI, like CEGUI, I have already used it but with more success. It is very easy to build and to run the samples. It is a bit trickier to link as they did not export CMake configuration files.

The library is really powerful, there are a lot of widgets and they are totally skinnable as you can see below.

![](/media/img/vagabond-choosing-gui-library/MyGUI.png){: width="400" .center-image .modal-image }

Moreover, it has good documentation: all the classes are documented and there are many samples where features are demonstrated, even for advanced features such as drag and drop.

Also, MyGUI provides really useful tools to create the interfaces and the skins.

![](/media/img/vagabond-choosing-gui-library/MyGUI_LayoutEditor.png){: width="400" .center-image .modal-image }

As you can see with the image at the beginning of the article, which is made with MyGUI, making a custom theme is quite easy. If it is a bit ugly, it is only because the artist, me, has no artistic talent. It is not in any way the fault of MyGUI.

Finally, it is easy enough to integrate it with SFML. If you are interested, I have coded a small library to integrate MyGUI with SFML, it is available on [GitHub](https://github.com/pvigier/MyGUI-SFML).

# Conclusion

I am very glad to have found a library that suits my needs and to have made a basic theme for the game.

The next and last step for the game engine is networking. It will be a big part so it may take me more than one week to work on that. But after that, I will work more directly on the game and I am really excited to be there.

See you next week for more!

*If you are interested in my adventures during the development of Vagabond, you can follow me on [Twitter](https://twitter.com/PierreVigier).*
