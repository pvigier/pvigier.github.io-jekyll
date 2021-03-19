---
layout: post
title: "Beginner's Guide to Game Networking"
date: 2019-09-08
author: pierre
tab: blog
tags: vagabond game-engine
---

During the last two weeks, I was working on the network engine of my game. And before that, I knew nothing about game networking. Thus, I read a lot of articles and did a lot of experiments to understand all the concepts and be able to code my own network engine.

In this guide, I want to share with you the different concepts you must learn before to write your own game engine and the best resources and articles to learn them.

There are mainly two possible network architectures: peer-to-peer and client-server. In the peer-to-peer architecture, data is exchanged between any pair of connected players while in the client-server architecture, data is only exchanged between players and the server.

While the peer-to-peer architecture is still used in some games, client-server is the standard as it is easier to implement, it requires less bandwidth and it is easier to prevent cheating. Thus, in this guide, we will focus on client-server architecture.

More precisely, we will be interested in authoritative servers, it means that the server is always right. For instance, if a player thinks it is at coordinates (10, 5) but the server tells him that it is at (5, 3), then the client should update its position to the server's one, and not the opposite. Using an authoritative server enables us to detect cheating more easily.

There are mainly three components in game networking:

* Transport protocol: how to transport the data between clients and the server?
* Application protocol: what to send from clients to the server and from the server to clients and in which format?
* Application logic: how to use the exchanged data to update clients and the server?

It is really important to understand the role of each part and its challenges.

<!--more-->

# Transport Protocol

The first part is choosing a protocol to transport the data between the server and the clients. There are two Internet protocols available for that: [TCP](https://en.wikipedia.org/wiki/Transmission_Control_Protocol) and [UDP](https://en.wikipedia.org/wiki/User_Datagram_Protocol). But you can also make your own custom transport protocol based on one of them or use a library that uses them.

## TCP vs UDP

Both TCP and UDP are based on [IP](https://en.wikipedia.org/wiki/Internet_Protocol). IP allows to transmit a packet from a source to a destination but it provides no guarantee that the sent packet will eventually arrive at destination nor that it will arrive only once, nor that the data is not corrupted during the transfer, nor that a sequence of packets will arrive in order. Moreover, a packet can only contain a limited size of data which is given by the [MTU](https://en.wikipedia.org/wiki/Maximum_transmission_unit).

UDP is only a thin layer above IP. Consequently, it has the same limitations. TCP, however, has a lot of features. It provides a reliable, ordered, and error-checked connection between two hosts. Thus TCP is really handy and is used in numerous other protocols such as [HTTP](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol), [FTP](https://en.wikipedia.org/wiki/File_Transfer_Protocol) or [SMTP](https://en.wikipedia.org/wiki/Simple_Mail_Transfer_Protocol). But all these features come with a cost: [*latency*](https://en.wikipedia.org/wiki/Latency_(engineering)).

In order to understand why these features may cause latency. We must understand how TCP works. When a source host sends a packet to a destination host, it expects to receive an acknowledgement (ACK). If it does not receive any after some time (because its packet has been lost or the acknowledgement has been lost or any other reason), it sends again the packet. Moreover, TCP ensures the packets are arriving in order, so while the lost packet has not been received, the other packets cannot be processed even if they have already been received by the destination host.

But as you may know, latency is important in multiplayer video games especially for action based games such as FPS. That is the reason why many games use UDP with a custom protocol.

They are several ways a custom protocol using UDP may be more efficient than TCP. For example, it can mark some packets as reliable and some others as unreliable. Thus, it will not care if an unreliable packet arrives or not at its destination. Or it can manage several streams of data so that a lost packet in one stream will not slow down the other streams. For instance, there can be a stream for user inputs and another one for chat messages, thus if a chat message, which is nonurgent data, is lost, it will not slow down a trigger input, which is urgent. Or, the custom protocol can implement reliability in another way than TCP which is more efficient under video games assumption.

So, if TCP is so shitty, let's implement our custom transport protocol based on UDP?

It is more complex than that. Even if TCP is almost always sub-optimal for video game networking. It may nevertheless work fine for your game and spare you precious time. For instance, latency may not be an issue for a turn-by-turn game or for a game that is only playable on LAN networks, where the latency and the packet loss rate is way smaller than on the Internet.

Many successful games, such as World of Warcraft, Minecraft or Terraria, use TCP. However, most of FPS use a custom protocol based on UDP, we are going to talk more about that in the next sections.

If you decide to use TCP, ensure that [Nagle's algorithm](https://en.wikipedia.org/wiki/Nagle%27s_algorithm) is disabled as it buffers packets before sending them and consequently increases the latency.

To know more about the differences between UDP and TCP in a multiplayer game context, you can read the article [*UDP vs. TCP*](https://web.archive.org/web/20180823015049/https://gafferongames.com/post/udp_vs_tcp/) by Glenn Fiedler.

## Custom Protocol

So you want to create your own transport protocol but you are not sure where to start? You are lucky because Glenn Fielder wrote two great series of articles explaining a way to do that. You will find plenty of smart ideas there.

The first one [*Networking for Game Programmers*](https://web.archive.org/web/20180823012240/https://gafferongames.com/categories/game-networking) from 2008 is simpler than the second one [*Building A Game Network Protocol*](https://web.archive.org/web/20180823014904/https://gafferongames.com/categories/building-a-game-network-protocol) from 2016. I advise you to start with the oldest one.

Be aware that Glenn Fiedler is a strong advocate of using a custom UDP based protocol. And after having read his articles, you will surely share its point of view that TCP has major flaws for video games and you will want to implement your own protocol.

But if you are new to networking, please do yourself a favor and use TCP or a library. You have surely many things to learn before being able to implement successfully a custom transport protocol.

## Network Libraries

If you need something more efficient than TCP but do not want to bother implementing a custom protocol and dive into many issues, you can use a networking library. There are plenty available:

* [yojimbo](https://github.com/networkprotocol/yojimbo) by Glenn Fiedler
* [RakNet](https://github.com/facebookarchive/RakNet) which is not maintained anymore but a fork [SLikeNet](https://github.com/SLikeSoft/SLikeNet) seems to be active.
* [ENet](http://enet.bespin.org/) which is a library that has been made for the multiplayer FPS [Cube](http://cubeengine.com/)
* [GameNetworkingSockets](https://github.com/ValveSoftware/GameNetworkingSockets) by Valve

I have not tried them all but my preference goes to ENet as it is simple to use and robust. Moreover, it has clear documentation and a tutorial to get started.

## Transport Protocol -- Conclusion

To sum up, there exists two base transport protocols: TCP and UDP. TCP has a lot of useful features: reliability, preserving packet order, error detection, while UDP does not but due to its design TCP has higher latency which may be inadequate for certain games. Thus, to have lower latency, it is possible to create a custom transport protocol using UDP or to use a library that provides a transport protocol based on UDP adapted for multiplayer video games.

Choosing between TCP, UDP or using a library depends on several factors. Firstly, the needs of your game: does it require a very low latency? Secondly, the needs of the application protocol: does it require a reliable protocol? As we will see in the next part, it is possible to design an application protocol that is fine with an unreliable protocol. Finally, the experience of the networking developer.

I have two pieces of advice:

* Abstract your transport protocol as much as possible from the rest of the application. So that you can easily change it without rewriting everything.
* Do not optimize prematurely. If you are not a networking expert and you are not sure if you really need a custom transport protocol build on UDP, you may start using TCP or a library that provides reliability and, test and measure. If there are issues and you are sure it comes from the transport protocol, then it may be the time to create your own transport protocol.

To finish, this part, I advise you to read [*Introduction to Multiplayer Game Programming*](https://web.archive.org/web/20190519135537/http://trac.bookofhook.com/bookofhook/trac.cgi/wiki/IntroductionToMultiplayerGameProgramming) by Brian Hook which covers a lot of topics we have discussed here.


# Application Protocol

Now that you have a way to exchange data between clients and the server you must decide what data to exchange and with which format.

The classic scheme is that clients send inputs or actions to the server and the server sends the current game state to the clients.

The server does not send the whole state but a filtered state with the entities that are around a player. It does that for three reasons. Firstly, the whole state may be way too large to be transmitted at high frequency. Secondly, the clients are mainly interested in the visual and audio data as most of the game logic is only simulated on the game server. Finally, in some games, the player must not know some data such as the position of an opponent at the other end of the map, otherwise, he can sniff the packets and know exactly where to go to kill him.

<!--image-->

## Serialization

The first step is to convert the data we want to send (the inputs or the game state) in a format suitable for transmission. This process is called [*serialization*](https://en.wikipedia.org/wiki/Serialization).

A first idea may be to use a human readable-format such as JSON or XML. But it would not be efficient at all and takes a lot of bandwidth needlessly.

Instead, it is advisable to use a binary format which is much more compact. Thus, the packets will just contain a bunch of bytes. One issue you should be careful about is [*endianness*](https://en.wikipedia.org/wiki/Endianness), the order of bytes may vary from one computer to another.

You can use a library to help you serialize your data such as:

* [FlatBuffers](https://google.github.io/flatbuffers/) by Google
* [Cap'n Proto](https://capnproto.org/) by Sandstorm
* [cereal](http://uscilab.github.io/cereal/serialization_archives.html) by Shane Grant and Randolph Voorhies

Just be careful that the library makes portable archives and takes care of endianness.

The alternative is to handle everything yourself, it is not really difficult, especially if you have a data-oriented approach in your code. It may also allow you to do certain optimization that is not always possible to achieve with a library.

Glenn Fiedler wrote two articles about serialization: [*Reading and Writing Packets*](https://web.archive.org/web/20180823004533/https://gafferongames.com/post/reading_and_writing_packets/) and [*Serialization Strategies*](https://web.archive.org/web/20180823015044/https://gafferongames.com/post/serialization_strategies/).

## Compression

The quantity of data that can be exchanged by clients and the server is limited by the bandwidth. Compressing your data may allow you to exchange more data in each snapshot, to have a faster refresh rate, or simply to have lower requirements on the bandwidth.

### Bit packing

The first technique is bit packing. It consists in using exactly the number of bits you need to represent a given quantity. For instance, if you have an enumeration that can take 16 different values, you will use only 4 bits instead of a whole byte (8 bits).

Glenn Fiedler explains how to achieve that in the second part of [*Reading and Writing Packets*](https://web.archive.org/web/20180823004533/https://gafferongames.com/post/reading_and_writing_packets/).

Bit packing works particularly well with quantization which is the next topic.

### Quantization

[*Quantization*](https://en.wikipedia.org/wiki/Quantization_(signal_processing)) is a lossy compression technique which consists in only using a subset of possible values to encode a quantity. The simplest way to achieve quantization is by truncating floating-point numbers.

Glenn Fiedler (again!) shows how to use quantization in practice in his article [Snapshot Compression](https://web.archive.org/web/20180823021121/https://gafferongames.com/post/snapshot_compression/).

Shawn Hargreaves also has some interesting articles on compression including quantization, you can find them all [here](http://www.shawnhargreaves.com/blogindex.html#networking).

### Compression Algorithms

The next technique is using lossless compression algorithms.

In my opinion, the three more interesting algorithms to know are:
* [Huffman coding](https://en.wikipedia.org/wiki/Huffman_coding) with a precomputed code which is extremely fast and can give good results. It was used to compress the packets in the Quake3 network engine.
* Using [zlib](http://www.zlib.net/) which is a general-purpose compression algorithm and it never expands the data. It is used in numerous applications as you can see [here](https://en.wikipedia.org/wiki/Zlib). It may be overkill for state updates. But it may be interesting if you have to send assets, long texts or terrains from the server to clients.
* [Run-length encoding](https://en.wikipedia.org/wiki/Run-length_encoding) is maybe the simplest compression algorithm but it is very efficient for certain types of data. It is specifically suitable for compressing terrains made of tiles or voxels where many adjacent elements are similar.

There is also a paid library by Rad Game Tools called [Oodle Network Compression](http://www.radgametools.com/oodlenetwork.htm). On the page, they show an interesting graph where they compared the compression ratio of Huffman coding, zlib and their solution, very instructive.

### Delta Compression

The last compression technique is delta compression. It consists in sending only the differences between the current game state and the last state received by a client.

It was first used in Quake3 network engine, here are two articles explaining how it was used:
* [*The Quake3 Networking Model*](https://web.archive.org/web/20190628180906/http://trac.bookofhook.com/bookofhook/trac.cgi/wiki/Quake3Networking) by Brian Hook
* [*Quake 3 Source Code Review: Network Model*](http://fabiensanglard.net/quake3/network.php) by Fabien Sanglard

Glenn Fiedler also used it in the second part of his article [*Snapshot Compression*](https://web.archive.org/web/20180823021121/https://gafferongames.com/post/snapshot_compression/).

## Encryption

Finally, you may want to encrypt the communication between clients and the server for several reasons:

* privacy/confidentiality: the messages can only be read by its receiver, any other person who sniffs the network will not be able to read them.
* authenticity: any person who wants to impersonate a player will have to know its key.
* cheating prevention: it will be way harder for malicious players to craft custom packets to cheat, they will have to reproduce the encryption scheme and find the key (which changes at each connection).

I strongly advise you to use a library to help you. I suggest [libsodium](https://download.libsodium.org/doc/) as it is particularly simple to use and it provides great tutorials. You will be in particular interested by the [key exchange](https://download.libsodium.org/doc/key_exchange) tutorial to generate new keys for each new connection.

## Application Protocol -- Conclusion

That is all for this part. I think that compression is totally optional and it depends on your game and how much bandwidth is needed. Encryption is, in my opinion, not optional but it may be skipped in a first prototype.

# Application Logic

You are now able to update the state in the client but you may experience latency issues. Indeed, you have to wait for a game state update from the server after having triggered an input to see its effect in the world.

Moreover, between two state updates, the world is completely static. Thus, movements will be completely chopped if the state update rate is low.

There are several techniques to mitigate these issues that I will present in the next section.

## Latency Mitigation Techniques

All the techniques presented in this section are presented in-depth in [*Fast-Paced Multiplayer*](https://www.gabrielgambetta.com/client-server-game-architecture.html) by Gabriel Gambetta. I strongly advise you to read this series of articles which is great. There is also a live demo to see how these techniques work in practice.

The first technique is to apply the result of an input directly without waiting for the response from the server. It is called *client-side prediction*. However, when the client receives an update from the server, it has to check that its prediction was correct, otherwise, it must modify its state according to what it has received from the server as the server is the authority. This technique was first used in Quake, you can read more in the [*Quake Engine code review*](http://fabiensanglard.net/quakeSource/index.php) by Fabien Sanglard.

The second set of techniques is for smoothing the movement of other entities between two state updates. There are two ways to achieve this: by doing interpolation or extrapolation. Interpolation is using the two last states and showing the transition from one to another. Its drawback is that it induces a bit of latency because the client always shows what happens in the past. Extrapolation consists in predicting where the entities would be now according to the last state the client received. Its drawback is that if an entity completely changes its direction, there would be a large error between the prediction and the real position.

The last technique which is most advanced and only useful in FPS is *lag compensation*. With lag compensation, the server takes into account the latency of the client when they are shooting at a target. For instance, if the player did a headshot on its screen but in reality, its target is elsewhere due to latency, it would be unfair to the player to refuse him his kill due to latency. So the server will rewind in time, at the time the player shot to simulate what the player saw on its screen and check the collision between his shot and the target.

Glenn Fiedler (always!) wrote [*Network Physics (2004)*](https://web.archive.org/web/20180823005028/https://gafferongames.com/post/networked_physics_2004/) in 2004 where he laid the foundations of synchronizing a physics simulation between a server and a client. In 2014, he wrote a new series of articles, [*Networking Physics*](https://web.archive.org/web/20180823004853/https://gafferongames.com/categories/networked-physics), where he showed more techniques to synchronize a physics simulation.

There are also two articles on Valve's wiki, [*Source Multiplayer Networking*](https://developer.valvesoftware.com/wiki/Source_Multiplayer_Networking) and [*Latency Compensating Methods in Client/Server In-game Protocol Design and Optimization*](https://developer.valvesoftware.com/wiki/Latency_Compensating_Methods_in_Client/Server_In-game_Protocol_Design_and_Optimization), which deal with latency compensation.

## Cheating Prevention

There are mainly two ways to cheat in a mutliplayer game: by sending malicious packets to the server or by reading data coming from the server that give an unfair advantage to the cheater.

A first technique is to make hard for cheaters to craft malicious packets and to read incoming packets. As we explained before, encryption is a good way to achieve that as it will [obfuscate](https://en.wikipedia.org/wiki/Obfuscation) the incoming packets and the cheaters will have to get the keys and reproduce the encryption scheme to craft malicious packets.

The second technique is to have an authoritative server that only receives commands/inputs/actions. The client should never be able to modify the server state by another way than sending inputs. Then, each time the server receives an input, it should check that this input is valid before to apply it.

The best technique to prevent cheaters from accessing data they should not know about is simply by making sure the server does not send it in the first place. For instance, the server should not send to the players the position of opponents or monsters that are far from them. Otherwise, even if they are not visible in the game, the players can read the incoming packets and know exactly where to go to kill their targets. This kind of cheating is called *map hack* or *world hack*.

If you want to know more about cheating you can read the article [Cheating in online games](https://en.wikipedia.org/wiki/Cheating_in_online_games) on Wikipedia which contains a list of possible ways of cheating and solutions to detect and prevent them.

## Application Logic -- Conclusion

I advise you to implement a way to simulate high latency and low refresh rates in your game to be able to test your game in bad conditions even if both the client and the server are running on your computer. It will simplify greatly the implementation of latency mitigation techniques.

# Other Useful Resources

If you are looking for more networking resources, you can find them there:
* [Glenn Fielder's blog](https://web.archive.org/web/20190328001900/https://gafferongames.com/), you should read its entire blog, it contains lots of great articles. [Here](https://web.archive.org/web/20180823014743/https://gafferongames.com/tags/networking) are all its articles concerning networking.
* [Awesome Game Networking](https://github.com/MFatihMAR/Awesome-Game-Networking) by M. Fatih MAR is an extensive list of articles and videos on game networking.
* [r/gamedev's wiki](https://www.reddit.com/r/gamedev/wiki/index#wiki_networking) also contains many useful links.

# Conclusion

That is all for this guide. I hope you learned a few things and find interesting articles. Good luck with the creation of your network engine!

See you next week for more!

Edit: Thanks to the redditors for the [feedback](https://www.reddit.com/r/gamedev/comments/d1oz18/beginners_guide_to_game_networking/). I tried to take it into account.