---
title: Project Mind Map Part 1 - Scene Graphs and Abandonment
taxonomy:
    tag:
        - phaser
        - tech
        - project-mind-map
    category:
        - blog
---

The last few weeks, I've been trying to make a game in [Phaser](https://phaser.io) - a FOSS framework for developing in-browser games.

Overall, it's been a pretty good experience. It does things a bit differently to LibGDX (the last framework I used heavily), but so far I've only run into one issue, which I'll talk about in a bit. Before I get to that though, I want to introduce my current project.

# Project Mind Map

About a month ago, my brother said to me "Hey you know how programmers hate when someone comes to them with a terrible half-baked idea? Yeah, I've got one of those."

The idea is based around an existing game, [Funny Farm](http://shygypsy.com/farm/p.cgi). In Funny Farm, you start by looking at this screen

![Funny Farm 1](ff-1.png)

A clue ("On The Farm"), and a textbox to put guesses into. If you guess one of the words directly connected to On The Farm, it reveals and opens up new hidden words.

![Funny Farm 2](ff-2.png)

The puzzle gets bigger and bigger, and different themes/genres are made available.

![Funny Farm 3](ff-3.png)

It's a great little puzzle, and I'd highly recommend giving it a go -- until it stops being fun, that is.

Yes, unfortunately Funny Farm runs into a brick wall right around the time the clue is "Poker Stars" and you realise that Funny Farm came out in 2006, so even if you had that kind of niche information it wouldn't be up to date anyway.

But luckily we don't live in the far gone land of 2006. The code running Funny Farm (at least the parts publicly available) is nice, if a bit archaic. All of the words are in a table that's had CSS scramble it around, each pixel of line is a 1 x 1 px div colored blue. The minimap is just a regular image that they're covering up parts of. It all works, but in a way that feels like it could be done better now.

And so, I would like to. My current project is creating a HTML5 version of this game. Specifically the checklist is

 1. Make a more modern, HTML5 version of Funny Farm
 2. Make it mobile-friendly and release it on app stores.
 3. Make a full-featured level editor, for users and siblings alike to make their own FF-style mind maps.

Is this a fully 100% original project? No, but everything is derivative and we're all going to die some day.

So now that you're up to speed on what the project is, I have good news! I've finished it to a technically functional degree! Great, right?

Not quite.

The version I've finished is... messy, to say the least. It was my first real Phaser project, and things didn't quite go how I expected. I reached a point where I could continue on in the doomed world I have created, or I could start from scratch and do it *right*.

So here I am - clean slate, and ready to write about the process of creating this thing that already kinda exists, only *better*. Let's do this.

# Scene Graphs

Before we start with the new and improved Project Mind Map, I would like to provide a little bit more context about why the old one fell over.

Two words - Scene Graphs.

One common way that video games will arrange things internally is called a Scene Graph. Basically they just split everything in the world up into little objects, called Nodes.

A node, at its core, has five* pieces of data attached to it
 * x position 
 * y position
 * width
 * height
 * texture

*I'm simplifying here, but just roll with it.

Using that information, the game engine will display any node that falls within the **camera**. The camera is a special node, which instead of having a texture uses **the magic of computer maths** to display other nodes.

To write some horrific pseudocode, this is how that works

~~~~
for (node in scene) {
    if ((node.x > camera.x OR
         node.x < camera.x + camera.width) AND
        (node.y > camera.y OR
        node.y < camera.y + height.height)) {
            camera.draw(node);
    }
}
~~~~

Or basically, "Hey gang, let's go through each and every node in our scene, and if it's in the view of the camera - let's draw it".

That's more or less the heart of the Scene Graph. A collection of nodes with a few properties.

# Zooming

Given that a camera has four properties (x, y, width, height) it would be easy to assume that in order to zoom in, you'd just make the height/width smaller, and that to zoom out you'd make them bigger.

On paper that's exactly what you do - but computers are decidedly *not* paper, so it gets a bit trickier.

See it turns out that displaying an image on screen is more difficult than you'd think. I'm not a graphics man, so I can't give you the full details - but because of the way that Phaser was coded originally, zooming is not on the menu. The creators, Photonstorm, are working hard to rectify this in V3, slated for release later this year, but until then the way that you zoom in Phaser is... well it's insane.

Let's expand our Scene Graph theory a little bit.

We've got two types of node right now - regular, and camera. Let's add a third - the group.

The group is a node with these three properties.

 * x position
 * y position
 * children

Children, in this context, just means a collection of nodes that belong to the group. So let's say that you're making a game, and you've got three planes. In order to keep them together (and write less code generally) instead of telling each one to move in formation, you can just put them all in a group and then tell the group to move. Groups are really good at, well - grouping. Heck, you can even put groups INSIDE groups.

One thing that this changes is suddenly instead of x and y position, all nodes have their x and y positions *relative to the x and y positions of their parent group*. If they aren't in any group, the scene itself becomes their parent group.

Now, this is all well and good - but what if I want to change the size of everything in a group, all at once, relative to each other?

The node model I gave earlier (x, y, width, height, texture) isn't quite full. **All** nodes (so camera and groups included), also have a scale. Let's say Mario has gotten a mushroom and so he wants to be bigger? Suddenly we can leave the width as it was, and just modify the scale.

And scale is carried through a group. The true width of any object is going to be `width * scale * parent scale`.

Luckily though, all of this scaling and relative positioning is handled by Phaser - completely abstracted away from game developers.

So, now that we understand groups - let's get back to zooming.

If I have a real-life actual camera, and I want to make something, a hat let's say, look smaller - what would I do? Zoom, right? What if I can't zoom for some reason? The answer is simple - I use my magical wizard powers to make the hat smaller. Duh.

Yeah yeah, "Magical wizard powers don't exist" and all that. Sure. But when you're working in a digital world, they... they kinda do.

It turns out the best way to zoom in Phaser is to literally just have a group containing *everything* and shrink it down. Or make it bigger, if you're zooming in.

This might sound really dumb, but honestly it's not an awful solution. It gives you more control over the zoom -- if you have two groups, "world" and "interface", then suddenly you have a camera that doesn't effect the user interface. It's not clean, but it works.

# Maths and Abandonment

While technically functional, this new world-warping camera has some drawbacks. Suddenly operations that used to be as simple as could be - such as mapping a location from the screen to the corresponding location in the world - have maths added to them. You have to account for the scale, for the screen width, for how much or how little to shift the camera during zooming. It's a headache. 

It's also why I'm abandoning my current build of Project Mind Map.

With a fresh start, I can build the game around the camera - and even save that code for future projects. Plus, I can write about my progress here. I've always wanted to have a series like [Shamus Young's Project Frontier](https://www.shamusyoung.com/twentysidedtale/?p=11874) (Or really any of his [programming series](https://www.shamusyoung.com/twentysidedtale/?cat=66)), and so that's what this is. Writing about this project is going to give me some good ol' practice at writing in this style.

Until next time!