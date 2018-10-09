---
layout: post
title: "Distributing a C++ program on Linux"
date: 2018-10-07
author: pierre
tab: blog
comments: true
tags: cpp linux simulopolis
---

Hi! 

Today I will tackle a problem I faced when I wanted to distribute the alpha version of Simulopolis two weeks ago: distributing a C++ binary on different Linux distributions.

So two weeks ago, I was happy, I had a version of Simulopolis which ran smoothly on my laptop. And I thought it was time to share it with people to have some feedback. Very naively, I took the binary and the assets, put them on a new folder and uploaded the whole thing on itch.io.

Then, I sent a message to my big brother telling him that he can try my game. He downloaded the game and when he tried to run the game, he obtained this error message:

```
./Simulopolis: error while loading shared libraries: libboost_serialization.so.1.65.1: cannot open shared object file: No such file or directory
```

Uhh, this was not expected!

![Disappointed gif](https://media.giphy.com/media/U4VXRfcY3zxTi/giphy.gif){: .center-image .modal-image }

We read the error message, it tells that a dependency I use to build the game is missing on my brother's system. So he tried to install Boost Serialization, SFML and TinyXML the three libraries I use in Simulopolis. But that did not solve the problem because the program is looking for a specific version of the dependency (here the version 1.65.1 of Boost serialization). And as he was on a different Linux distribution, Fedora, than me, Ubuntu, it was not the same version of Boost Serialization that was available in its package manager.

<!--more-->

# Packaging vs static linking vs dynamic linking

I search over the Internet to see how people usually tackle this problem. I mainly found three methods.

The first one is to use the package managers of the different distributions to manage the dependencies. This method requires to write a little manifest for each package manager and to compile the game for each system. Then instead of distributing a zip, I would distribute a .deb or a .rpm depending of the package manager. I think this is the recommended method because you use the system's libraries that are shared by all the applications. However, I thought this requires too much work for a first version. Maybe, I will investigate that later.

The second one is to use static linking i.e. to embed the dependencies in the binary. What bothers me with static linking is that if you want to link a library licensed under the GPL then the whole app becomes licensed under the GPL. In the case of Simulopolis, it was not a problem as the game is already open-source and distributed under the GPL. But I could not reuse the method for a future project not distributed under the GPL.

The third one is to use dynamic linking and to distribute the dependencies with the binary. Exactly as we do on Windows when we distribute DLLs alongside the binary. The difference between Linux and Windows is that by default on Linux a binary only looks for libraries on the system's folder like `/usr/local/lib` while on Windows the executable looks first in the current folder. But there is a way to tell the executable to look for the dependencies in the current folder, this is called run-time search path (rpath).

# Setting the rpath

I encourage you to read the [wikipedia page on rpath](https://en.wikipedia.org/wiki/Rpath) which is very instructive to understand how the [dynamic linker](https://en.wikipedia.org/wiki/Dynamic_linker) looks for shared libraries.

So to tell the dynamic linker to look for the dependencies in the same folder where the binary is we have to set the RPATH of the binary to `.`. To do that with gcc, you just have to use this parameter: `-Wl,-rpath,.`. Personally, I find cleaner to put all the dependencies in a `lib/` folder so I use this parameter `-Wl,-rpath,lib`.

We can check that the rpath is correctly set by using the command:

```
readelf -d <executable>
```

It reads the information written in the binary (see [ELF](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format)). For Simulopolis, it outputs:

```
0x000000000000001d (RUNPATH)            Library runpath: [lib]
```

Now you must add your dependencies in the `lib/` folder. But how to know which files it is neccessary to put there? Again, using the `readelf -d` command we can read the shared libraries that are needed by the binary. For Simulopolis, it is:

```
0x0000000000000001 (NEEDED)             Shared library: [libsfml-audio.so.2.5]
0x0000000000000001 (NEEDED)             Shared library: [libsfml-graphics.so.2.5]
0x0000000000000001 (NEEDED)             Shared library: [libsfml-window.so.2.5]
0x0000000000000001 (NEEDED)             Shared library: [libsfml-system.so.2.5]
0x0000000000000001 (NEEDED)             Shared library: [libtinyxml2.so.6]
0x0000000000000001 (NEEDED)             Shared library: [libboost_serialization.so.1.65.1]
0x0000000000000001 (NEEDED)             Shared library: [libstdc++.so.6]
0x0000000000000001 (NEEDED)             Shared library: [libm.so.6]
0x0000000000000001 (NEEDED)             Shared library: [libgcc_s.so.1]
0x0000000000000001 (NEEDED)             Shared library: [libc.so.6]
``` 

Then if you installed the libraries using your package manager you can use `locate <library>` to quickly find where the libraries are. Be careful to copy the shared library and not a link to the shared library! You may need to rename the library so that it matches exactly the name written in the binary.

I decided not to put the last four because there are standard libraries of C and C++ and they should be present in all modern Linux distribution.

If needed, you can set the rpath of a binary after its creation using the command `chrpath`.

Finally, my brother can play to Simulopolis!

![Success gif](https://media.giphy.com/media/uTuLngvL9p0Xe/giphy.gif){: .center-image .modal-image }

I hope these insights were useful to you!

See you!
