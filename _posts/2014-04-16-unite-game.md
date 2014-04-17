---
layout: post
title: "Unite (game)"
date: 2014-04-16 22:00:00
description: ""
category: 
tags:
- games
- browser
- GAE
- Python
- coffeescript
- browser
- bootstrap
---
{% include JB/setup %}

## Demo
[http://unite-game.appspot.com](http://unite-game.appspot.com)

## Conception
I want to do a big MMO that feels vaguely prisoner's-dillema-y. Here's the mechanic: you can "join" someone, and they become your leader. At the end of the game, the person who is leading the most people (transitively, so if A leads B leads C then A leads C) gets the game jackpot. Then points trickle down her leadership tree, so the further you are from the root, the less points you get. Of course, if you're not in the tree at all, you get no points.

<!--more-->

## Tech
I was impressed by GAE with Crowdsourced Banter, so I'll use it again for backend. This game will require accounts, and GAE is good at that. Frontend'll use Bootstrap and CoffeeScript. I'll be using d3.js today for a force-directed graph, which is new for me.

## Hack
Okay, backend first. With GAE that means defining an OOP Model. Fundamental data unit is a Player. A Player knows their current tree size (how many people they are leading), their leader, their direct children, and the Google account associated with them (it's GAE).

Basic endpoints to go up first: `start_playing` to register a new user with the current game, `join` to join a player with a leader. Up without many issues. Then some simple read endoinpoints: `game_state` will suffice for now, to give all game state.

Frontend expriments: make a force-directed graph of the game state. Basically mimicked the d3 tutorials for this, but it's up. Now I want interaction; clicking a node should pop a tooltip which has a "JOIN" button. Still getting used to d3, stumbled a bit positioning the tooltip, but that was ephemeral and things are going smoothly. On a whim, I'll also plan to set up a messaging system, and I'll put a "MESSAGE" button up next to "JOIN". Hooked the JOIN button up to the `join` endpoint, works fine. I need to refresh game state, which, since I'm hacking, I'll just do by refreshing the whole page. Performance is pretty good, so this is a reasonable end-user experience.

UI fix: you shouldn't be able to JOIN yourself. So I'll just disable the tooltip altogether for the user's own player. In fact, I should probably highlight the user's player in a different color. Okay, all done.

I want to display some info about the logged-in user on a left-hand panel. Time to make a `my_data` endpoint. Very quickly I'm going to refactor the Player serialization code into `Player.serialize` (this used to be in the `game_state` endpoint), to avoid code repetition on a basic level.

In making the `my_data` frontend, I need to know some info about the player's leader and subordinates, about which I only have uids right now. So I'm going to make a `lookup` endpoint, to spout data about any user. Up without issues. Again, still getting used to d3, so I had some trouble on the frontend, but other that things went fine.

Bug fix: when you join in cycles the server hangs. Simple fix, just throw an error if there's a cycle. At the same time, let's throw an error if you're joining someone when you've already joined someone else.

At this bug fix, I'll also implemented an "unjoin" functionality. To avoid extra work, I'm just building this in to the "join" functionality; if you've already joined with someone, the `join` endpoint will *unjoin* you from them. Okay, great, that's up.

Time to implement the messaging system I'd planned earlier. This'll require some more Model OOP: a Message knows its sender and its text. Let's make `check_messages` and `message` endpoints for interacting with this. Great, works well. I'll do the UI for this with Bootstrap's `modal` library. Bootstrap doesn't like me using JS for interacting with it, so I'll have to hack it a bit -- its api binds the modal to a button, which I'll do. I'll then have the button set some global state about what the modal should be doing (to whom we should send stuff). Okay, great, works fine.

Time to implement message-viewing. The `check_messages` endpoint works fine, so this is solely a UI problem. UI was a little tricky, though; I ultimately decided to lay this out with a Boostrap table. First iteration was just divs and spans, but layout was too difficult.

Time to implement the game-end cron job. This is the stage of the game where points are awarded and the game state is reset. Distribution defiition will be:
```
{WINNER} RECIEVES {JACKPOT}
Any node that RECIEVES N points has N/2 points added to their score,
  and each of its K children RECIEVES N/(2K) points.
```
Okay, great implemented fine. This definition actually means that some points stay unawarded from the game jackpot (leaf nodes get N/2 points, and give none to children); that's okay, I think. Set up a quick cron job, and we're done.

I think this game concept didn't pan out the way I thought it would. I really can't imagine this being a popular game. Wow, I'm a bit depressed. Time to make a [dumb browser game](/2014/04/16/fire-minigame).

## Launch
It's GAE, so I can just push.
