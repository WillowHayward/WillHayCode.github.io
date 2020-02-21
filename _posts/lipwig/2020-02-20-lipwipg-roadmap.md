---
layout: post
title:  "Lipwig: What needs doing"
date:   2020-02-21 11:51:00 +1000
categories: lipwig
---

Heya, it's been a while! Things have been BUSY. Mostly non-dev life stuff, but I've also been keeping busy with Lipwig. The progress has been less tangible, and more along the lines of investigating and planning. Right now I've got a pretty good idea of what exactly needs to be done for Lipwig to become a viable tool for production. So without further ado, here is the Lipwig Roadmap.

# Documentation

[LipwigCore](https://github.com/WilliamHayward/LipwigCore) (the server portion of Lipwig) and [LipwigJS](https://github.com/WilliamHayward/LipwigJS) (the JavaScript library to access it) are currently functional. I've used them in a few minor projects before. It's easy enough to use if, say, you're the person who wrote it and thus have intimate knowledge of its API. If you're not that person (or even if you're that person using it for the first time in 2 years), it's a bit trickier.

I did write some docs for it way back when, and had them stored on a GitHub wiki associated with the project. Unfortunately those were somehow lost to the sands of time, and the in-code documentation is somewhere between "laughable" and "nonexistent". So that's the first step - documenting the existing code. That comes in 2 parts - in code documentation, and a public API document.

Documenting code is a long and tedious project (which is presumably why 2017 WillHay decided to make it a 2020 WillHay problem). The main reason for this is that while it adds infinite value to a project, it doesn't add any new features, and thus involves minimal coding. I like writing code, and if I need to work on a non-coding portion of a project then suddenly I'll suddenly have a **burning desire** to improve my .vimrc, or to learn tmux

![](/assets/img/lipwig/tmux.png)

(My current work terminal setup, if anyone is curious)

Now, as a sort of compromise with myself I've decided that while I'm documenting (and likely beyond) I'm going to work through some coding puzzles to scratch that itch. A few years back I participated in [Advent of Code](adventofcode.com), where every December 25 programming challenges are released as an advent calendar. I'll be starting with the 2016 challenges and posting my progress [on GitHub](https://github.com/WilliamHayward/Advent-Of-Code), so if anyone wants to follow along that's where to do it.

The fun part of documenting code is writing the public API. A decent chunk of it can likely be generated from the comments, and the rest is just going to be discussing exactly how to use it - which means writing code.

So that's the first thing I need to get to get out the way before I can get to some of the more fun stuff.

# Fixes

The overall lack of maintenance on Lipwig has caused it to get left in the dust somewhat. It doesn't compile under the latest version of the TypeScript compiler, and GitHub has not been subtle in letting me know that a lot of the project dependencies are out of date and are vulnerable as a result.

![](/assets/img/lipwig/vulnerable.png)

So I'll have to make some updates and do some refactoring before I can start adding new features. I'm also hoping that I'll be able to combine it with what I've learned from documenting and make a couple of changes to improve how it is to work with.

# More Tests

Luckily 2017 WillHay had the foresight to write unit tests for Lipwig, but they're not quite as comprehensive as 2020 WillHay would like them to be. This must be remedied. I'm also going to look into static analysis for JavaScript and TypeScript, just to make sure that everything is as good as it can be.

# Reconnection Protocol

Right now, if you're connected to Lipwig on a mobile device and you lock the device, the WebSocket connection is permanently severed. This is fine if you assume that users will at no point lock their phones, but that is a very silly assumption indeed. Thus, a reconnection protocol is required. It shouldn't be too complex to implement - all of the aspects (error codes, user ids) are already in there, it's just the actual protocol itself that's missing.

# Audiences

Lipwig was very much designed with [Jackbox Party Pack](https://jackboxgames.com/party-pack/) style games in mind, and one feature that these sorts of games have are Audience Members. People who joined late (or after the game was full), but still get to interact in minor ways. With the Host/Client architecture of Lipwig, you could absolutely implement that by having each audience member as an individual client, but it would involve the host having to field a tonne of messages. Rather, I've been designing a new type of user - an Audience Member. The host can still interact with audience members, same as with a client, but the audience data is aggregated on the Lipwig data before being sent to the host. It removes a lot of the individuality of the audience member, in exchange for halving the total number of network messages that need to be sent.

# Performance and Security Upgrades

> The real problem is that programmers have spent far too much time worrying about efficiency in the wrong places and at the wrong times; premature optimization is the root of all evil (or at least most of it) in programming. *~Donald Knuth, The Art of Computer Programming*

Right now, I'm the only person using Lipwig - but I would love for that to change some day. One of the things that would be required for that to happen is for Lipwig to perform better than other solutions. I've got a few ideas on how to do that, but I'm not going to put the cart before the horse and start working on that any time soon.

Security on the other hand I wanna get *locked down*. Right now if I were to host a Lipwig server online, anyone would be able to connect to it for whatever purposes. That's all fine in theory, but it could mean I would end up with a massive hosting bill for someone else's project. To avoid that, I want to implement an API key system (or something similar) for connecting to Lipwig servers, just to ensure that only the right people are doing so.

# Dashboard

Sort of an extension to the previous one - managing API keys and such. I also think it would be neat to be able to view metrics like "current number of rooms" or "number of people connected". That sort of stuff. 

# WebRTC Support

WebRTC (short for Web Real Time Communication) is a relatively recent development in web tech. Basically, they are a way for two client devices (read: web browsers) to directly connect to each other. You might be thinking to yourself "WillHay! I know you can't hear me right now, but I think WebRTC renders the entirety of Lipwig obsolete"

To that I only have two things to say

1. I **can** hear you, 
2. Not quite, and here's why.

Lipwig (as it exists currently) is built on top of WebSockets, which in turn are built on top of a network protocol called "TCP", short for "Tiny Computer Primates"[^1]. I won't get too deep into it, but in essence a TCP network connection is a kind of connection that sacrifices a little bit of speed for stability - they might not be getting there as fast as the wind can take them, but messages sent on a TCP connection have a pretty decent chance of

a) Arriving

and

b) Arriving in order.

For contrast, WebRTC mostly uses UDP when it can. UDP (short for Undying Dread Primates[^2]) is usually much faster, but messages are processed in the order they happen to arrive, if they manage to arrive at all.

They both have different use cases, and for the core functionality of Lipwig I think it makes more sense to sacrifice a small amount of speed in order to be confident that messages are being received. That said, I think having that direct connection would be a neat auxilary function of Lipwig. You could use a WebRTC connection in conjunction with the Lipwig WebSockets, or if a WebRTC connection is the primary functionality you need the most then I think Lipwig could work quite comfortably as a signalling and TURN server - two tools used to help with WebRTC connections.

I'm pretty excited for this, partly because I have an idea for another new feature that would basically require it to function.

# Controller

Back in 2018 I used Lipwig to build [this proof of concept](https://www.youtube.com/watch?v=8z0C9y_z3I4) for a mobile-based controller. There's a little bit of lag between the computer and the device, but I think that with the WebRTC support you could really trim that down. This would probably be more of a plugin for LipwigJS than core functionality.


And that's... about it. There's a few other minor features I wanna implement, though most of them are currently kinda formless and would require a lot of the above features anyway.

Right now I'm in a pretty good spot with Lipwig. I understand how it works, and I know what to do next. Now for the tricky part: Actually doing it.

WillHay out.


[^1]: Transmission Control Protocol
[^2]: User Datagram Protocol
