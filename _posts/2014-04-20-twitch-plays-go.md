---
layout: post
title: "Twitch Plays Go"
date: 2014-04-20 22:40:00
description: ""
category: 
tags:
- browser
- websockets
- coffeescript
- html5_canvas
- games
- multiplayer
- heroku
---
{% include JB/setup %}

## Demo
[http://twitch-plays-go.herokuapp.com](http://twitch-plays-go.herokuapp.com)

## Conception
This hack idea came from Calvin Luo, who wanted to make a spin-off of [Twitch Plays Pokemon][tpokemon] (of which I had never heard until then; oh well). I felt a little guilty posting something so undemoable today, and this was a very easy concept, so I implemented it.
<!--more-->

## Tech
It's going to be a CoffeeScript webapp on the html5 canvas, leveraging websockets with [socket.io][socketio]. I'll probably host on Heroku, since it's the only free service I know offering websocktes.

## Hack
Okay, game logic first. I'll lay down some boilerplate OOP; fundamental data type is a Board (this is probably just dogmatic, but oh well, I'll keep it anway). Meatiest part of this hack will be the `place` operation, which will have to do a bfs to determine whether pieces need to be removed. The way I implemented it, search is actually done in dfs order, but that's fine. I can tell when a piece needs to be removed.GA

I first started with a nodejs `readline` interface for a simple person-vs-self command-line version of Go. This went up pretty quickly.

Time to render. I transposed the nodejs readline code to my standard 500x500 html5 canvas and bound `place` to `click`. Since the board is partitioned into non-overlapping squares I can do hit-testing in constant time. First iteration renders laid pieces as squares, to assure that things work in the browser; circles as pieces goes up quickly afterward.

Okay, time to run a socket.io server. I transpose the code back to server-side and run the socket.io boilerplate to get things running. This goes up fast. I'll have two socket.io events, `place` and `game_update`, sent by the client and server respectively. I write a couple lines of code regsitering these on the client, and we're up. This was super speedy.

## Launch
I'm launching to heroku. It turns out Heroku actually has a 5-app limit for devs who didn't register with a credit card -- I might need to watch out for that in the future.

[socketio]: http://socket.io/#home
[tpokemon]: http://www.twitch.tv/twitchplayspokemon
