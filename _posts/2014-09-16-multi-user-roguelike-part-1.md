---
layout: post
title: "Multi User Roguelike: Part 1"
description: ""
tags:
 - multiplayer
 - games
 - roguelike
 - coffeescript
 - nodejs
 - telnet
 - tty
---
{% include JB/setup %}

Demo
====
`telnet ec2-54-68-162-49.us-west-2.compute.amazonaws.com`

Conception
=========
As I said in my last post, this past weekend I spent approximately way too long playing Achaea. Upon some research, I found a ton of good multi-user text adventures but no good multi-user roguelikes (Interhack? I can't find a server anywhere).

So, I decided to make one! I have some game mechanics in mind but I won't flesh them out here, because they are too numerous and too subject to change. For today I will implement basic board/player movement support and field-of-vision computation.
<!--more-->

Tech
====
As per modus operandi meus, I will be using CoffeeScript. I will run a little telnet server with node.js, which seems not to be a new frontier.

Hack
====
O-kay, lots of setup time. OOP first. I need structs for the game board, floor tiles, and space for denizens and objects eventually. I lay down classes `FloorCell` and `GameBoard` (2d array of `FloorCell` objects) and construct `FloorCell` to have room for some item contents, and also a terrain type.

Okay, great. Can't test yet, because there's nothing there, but I can start doing shadowcasting. I'm going to implement [this guy's algorithm][shadowcasting]. I considered briefly just taking his code, but his thing is so regimented (it's almost Java-esque) I feel like it would be easier (and more educational) to just write it myself.

Okay, so it looks like a fundamental part of the algorithm is the "shadow queue", which seems to be a struct for intervals that are enshadowed. I try implementing my own `ShadowQueue` first, but the code ends up pretty messy -- I don't even really want to test it. Then I realize this person has a pretty detail pseudocode on their emplacement operation, so I copy that. The read operation is a pretty easy extension from it, so I do this.

At this point I want some reward. So, it's unit test time! (Unit tests in a hackperday!? Blasphemy!) I write a little test file using node.js's `assert` module -- emplacement works properly, but reading fails. It ends up being a simple typo, so I fix it, and unit tests pass! Excellent.

Okay, time to actually implement the shadowcaster. We progress outward in rings and mark visibility -- looking at this other person's codebase, it seems that hardcoding the rings as offset sets is clean enough, so I implement this too.

To test, I need to mark some nodes as blocked. In retrospect, it was probably better to modify the preexistent @terrain attribute of the nodes, but that's okay. It's a hack-per-day. I add an additional flag for @blocked.

To really test, I need a renderer. So I'll add a render method on `GameBoard` that stringifies things. I put char representation inherent to the `FloorCell` (subject to change -- tilesets?) and do some simple concatenation. I'll just have the render method take a set of visible cells as an argument for now -- also subject to change later. To make this a little less clunky, I'll migrate the return value of the shadowcaster to a JS object (implemented as binary search trees natively).

Okay, great! Time to test. I render things, and typo is immediately clear -- the visibility map is reversed (visible/invisible were swapped). So I go fix this, and things seem to work. To see if the effect is visually good enough, I set the tty to rawmode and add simple motion with nethackesque `hjlkbyun`. To display the player, I will add yet another flag to the floor tile (I know, I know it ought to be in the `@items` I had allocated earlier), and render the `@` symbol. Great! Everything looks good.

Time to port to telnet and make it multiuser! I find an npm telnet library and pull off hello world with it. Then I try to cut-and-paste my tty interface into it, which works for the most part after some typo resolution. In order to make it multiuser, I do a simple event handler system and bind render handlers to it for each client. I call the all handlers whenever a change occurs from any user. Great! Things work well. I can connect with multiple users and the users can see each other move about. Launch time!

Launch
======
I don't know any good telnet hosting service, so I'll put this one on aws ec2. Immediately, a couple bugs -- if the lag is such that multiple characters come through at once they are not recognized (because "hh" !== "h" and "h"). This is easy to resolve; when I recieve a multichar string I split and iterate. Second, we throw errors when we try to render for people who have disconnected. This is also easy to resolve; I unbind handlers on disconnect.

Great! Everything seems to work. It's a bit laggy, but I think that's just ec2 free tier being cheap. Donate, anyone?

[shadowcasting]: http://www.roguebasin.com/index.php?title=Precise_Shadowcasting_in_JavaScript
