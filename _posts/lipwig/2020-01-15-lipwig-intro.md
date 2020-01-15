---
layout: post
title:  "A somewhat roundabout introduction to Lipwig"
date:   2020-01-15 19:01:00 +1000
categories: lipwig
---

The story goes that in late October 1969 Bill Duvall and Charley Kline were attempting to send the first networked message between two devices. Trying to send the word "login", the first two letters transmitted and then the entire system crashed. 

This is a pretty accuration summary of what writing netcode is like.

My first foray into real-time netcode (that is to say, not things like PHP) was in high school when I made a very basic networked program in Game Maker 8. In it, the host had a sprite that would continually update to the mouse position, and then send that out to all the clients to move their sprite to the same position. It was unstable and it took me hours, but hot dang was I proud of it.

Aside from a few other projects of similar scale, my next major networking attempt was a project called "Crowd9" that I worked on in university. It was a nifty little social game where all the players were presented with 3 options (Star Wars, Star Trek, and Firefly for example) and had to select one. If you were in the majority, you would get points. It was good light fun, but unfortunately the project fell apart when I started running into some networking issues that were beyond my skills as a programmer at the time.

Not long after leaving that project, I started work on Lipwig.

Before I get too into what Lipwig actually is though, I'd like to talk broadly about networking. See, as I alluded to with that story at the start - networking is *hard*. There's a lot of things that can go wrong, and there's a lot of knowledge you have to have to utilise it well. Sockets, TCP/UDP, flags - it can be intimidating. Luckily most developers won't have to deal with that stuff directly. Rather, they'll use libraries - collections of code made and released by other developers. For Crowd9 I was using [Socket.io](https://socket.io/) - a really good JavaScript library for networking through WebSockets.

The simplest and most common type of server (Used in games like Minecraft or Counterstrike: Source) follow a simple structure: There's a host which handles the game logic, and clients connect to the host to send/receive relevant game data. In this type of situation, the host could be a dedicated server or another player who is hosting the game while also playing. There's usually a lot more to it - aspects like authoritative servers and client side prediction, which are interesting and worth learning about but not really relevant to Lipwig so I'm gonna blow past them. Here's a visualisation of this basic server model.

![](/assets/img/lipwig/host-client.png)

Some games however will require ALL players (regardless of if they're playing together or not) join a single\* centralised server. Games like League of Legends and Overwatch do this to minimise cheating and to ensure that nobody is hacking themselves to have items you have to purchase to obtain. This works very similarly to the previous server model except that the host handling the game logic is shared by everybody, they just group players in a game together like this:

![](/assets/img/lipwig/remote-host.png)

\* This is a simplification - it'll often be several servers acting together to distribute the load

Lipwig is basically an extension of that with one major difference: the central server doesn't handle game logic, but simply passes messages between connected users. A "room" is created by someone, and then other people join that room. The person who creates the room gets to handle the game logic, and the other users can pass messages to the room creator (who in turn can send or receive messages from anyone who joins the room). It looks a bit like this:

![](/assets/img/lipwig/lipwig-model.png)

This might seem like a really confusing and roundabout way to do it, so let me explain. What I'm showing here is an entirely behind the scenes look at Lipwig. The real benefit of it is that beyond initial setup, people using Lipwig don't need to worry about any of this. See, the person who creates a room (the "Host", if you will) handles the game logic, and has the ability to send and receive messages from anyone who joins (a "Client", as it were) through Lipwig. Because Lipwig just exists to pass along messages, you can more or less remove it from the equation. From the programmer's point of view, the Host launches a room, Clients join, and the networking is taken care of behind the scenes. To put that into an image, you'd be looking at something like this:

![](/assets/img/lipwig/host-client.png)

Bam, we're back to where we started - a simple host/client server structure. THAT is what Lipwig is - a way to simulate basic servers without having to think about the netcode behind it.

But now that we know what it is, let's talk about why I made it. There's a games company that I adore, Jackbox Games, who make fun party games that often follow a similar structure. The game runs on a computer, starts up a room on their server, and then players join on their phones without having to download a thing. The game logic happens on the computer, but all player input happens through your mobile device. 

I think this is **brilliant** and it blows me away that there's not more games like this. I think there's more now, but when I started Lipwig in 2017 I was really struggling to find games that used this system - and I knew from my Crowd9 experiences that a major reason for this is that networking is *hard* - so I figured hey, why not help take the headache out of it for other people. And thus, Lipwig was born.

The first version of Lipwig was built in regular JavaScript using Node.js to power the server and Socket.io to power the networking. It worked, but was... let's say messy. I didn't know how to modularise JS into classes (so all of my code was in one enourmous file), and while I knew about style guides I for some reason decided to forgo them. I got it working, I didn't touch it for six months, and then came back to a writhing monstrosity of spaghetti code. 

So I restarted from scratch, still with Node.js but now using Typescript and a websocket library directly. I also added compulsory linting (enforcement of a style guide) into my build process. As a result, this version is basically infitely better than the original. It's still somewhat lacking in documentation and comments, but without having even looked at the code in over a year I was able to build a rudimentary Java API for it within an hour (most of which was deciding what Java websocket library to go with).

This version of Lipwig does work, but there's a lot of stuff to do before I could comfortably call it finished. The afformentioned documentation is gonna be my starting point, but there's also some handy features like auto-reconnection and audiences that I'm keen to work on. I'll be doing this in tandem to developing a simple game with it - Rock Off, a rock paper scissors game - to make sure that it's as easy and intuitive to work with as I can make it.

The last thing I want to talk about here in relation to Lipwig is the name. I looooove the names programmers give things. PHP stands for "PHP: Hypertext Processor", Valgrind (a memory debugging tool) is pronounced like "grin" not "grind", and Python is named after Monty Python. Moist von Lipwig (pronounced Lip-Vig) is the main character of the Discworld novel "Going Postal", which is about the establishment of a post office in a fantasy society. As well as being a great book, naming a network messaging system after someone involved in a postal messaging system just felt right.

And there you go, that's a somewhat roundabout introduction to Lipwig. Next up: using it in an actual project.

WillHay out.

P.S. I wrote the majority of this blog post in Vim! 2020 goals are coming along.

![](/assets/img/lipwig/vim-post.png)

# Links

- [Socket.io](https://socket.io)
- [LipwigCore](https://github.com/WilliamHayward/LipwigCore) - This is the remote server part of Lipwig, where the messaging sorting happens
- [LipwigJS](https://github.com/WilliamHayward/LipwigJS) - This is the JavaScript API for Lipwig
- [Jackbox Games](https://jackboxgames.com/)
