---
layout: post
title: "Vagabond &#8211; Audio Engine"
date: 2019-08-18
author: pierre
tab: blog
tags: vagabond game-engine
---

After the [graphics engine]({{ site.baseurl }}{% post_url 2019-07-28-vagabond-2d-light-system %}) and the [physics engine]({{ site.baseurl }}{% post_url 2019-08-11-vagabond-2d-physics-engine %}), this week, I have worked on the audio engine.

To play sounds and music I use the [audio part of SFML](https://www.sfml-dev.org/tutorials/2.5/audio-sounds.php) which is really good. It is a thin wrapper around OpenAL. It supports WAV, OGG/Vorbis and FLAC. Moreover, it also supports [spatialization](https://www.sfml-dev.org/tutorials/2.5/audio-spatialization.php) and it is really easy to use.

There are only two things for which an audio engine is needed.

The first one is to make sure that the total number of sounds and music never exceeds 256 (which is an internal limit according to SFML documentation). To do that, the audio engine refuses to play a new sound or music if the limit has already been reached. And as soon as a sound or a music finishes, it removes its instance to be able to play a new one.

The second thing is being able to pause all sounds and music, play other sounds and music and then be able to resume all the sounds and music that have been paused. This use case happens when you are in-game and some sounds or music are playing. Then you go to the pause menu, all the game simulation is paused, so the sounds and music should be paused too. Some sounds are played by the user interface in the pause menu. Finally, when the player returns to the game, the simulation resumes and all the sounds and music should be resumed too.

To achieve that, I will use two abstractions, one for each of the two issues, respectively the audio engine and the audio scene.

Let us dive into the details of these abstractions.

<!--more-->

# Audio Engine

The audio engine will manage all the audio sources i.e. the sounds and the music. All creation of a new audio source should be done through the audio engine.

To do that, I use 4 data structures:

* a sparse set for sounds;
* a sparse set for music;
* a priority queue for sounds;
* a priority queue for music.

A sparse set is a data structure that I like and that is quite common in the video game industry. The idea is to associate an id to each object of the set. The container that makes the link between the ids and the objects may be sparse but the container that contains the objects is dense as shown on the image below.

![](/media/img/vagabond-audio-engine/sparse_set.svg){: width="400" .center-image }

If you want more details about sparse sets, you can read this [article](http://bitsquid.blogspot.com/2011/09/managing-decoupling-part-4-id-lookup.html) on bitsquid's blog. I think it is the article that democratizes this data structure. I may write an article on my implementation later that is a bit more modern than theirs.

I use sparse sets to store only the sounds and music that are currently playing and to have an id that can be used to pause or stop the audio source later on. An alternative is to use an `std::map` or `std::unordered_map` but it is a bit less efficient.

The priority queues are used to know when sounds or music should be removed. When an audio source is played, an event with its expected stop time is added to the corresponding priority queue. Then at each frame, we check the current time against the stop time of the first element of the priority queue, if it is more, we remove the audio source and check again, otherwise, we do nothing. For an audio source that loops or that is paused, I set the stop time to infinity.

I do not use an `std::priority_queue` to implement the priority queues because we cannot easily remove an element from the queue. Instead, I use an `std::set` which sorts elements and hence can be used as a priority queue. Moreover, it allows removing any element.

# Audio Scene

The audio scene will keep track of all the ids of sounds and music that are played in the scene. Then, when we want to pause the current audio scene, it saves the state of all currently playing audio sources and stops them. Finally, when we want to resume the audio scene, it restores the audio sources with the saved states.

It is quite simple to save the states of an audio source with SFML API as you have an easy access to current playing offset, volume, pitch, position in the space, etc.

The last thing that must be saved and restored correctly is the position and direction of the listener. Again, the audio scene just keeps track of their values.

# Conclusion

That is all for my audio engine, it is simple but robust and easy to use. I am happy with its design.

The next goals are choosing a GUI library and starting to work on the networking part.

See you next week for more!