---
layout: post
title: "Black on White (minigame)"
description: ""
category: 
tags: []
---
{% include JB/setup %}

## Demo
[http://dabbler0.github.io/black-on-white](http://dabbler0.github.io/black-on-white)

## Conception
Everybody seems to like [Phaser.js][phaser], but I've never used it. Today I feel like the JS speech recognition experiment was not so flashy, so I feel like I owe you a game. It'll be a pathological arcade-style thing, and I'll write it Phaser to see how that goes.
<!--more-->

## Tech
As you saw in the conception, this is going to use Phaser. It'll be static, in CoffeeScript.

## Hack
This is a super simple game, so most of this hacking is just going to be copying stuff from Phaser tutorials. I'm going to use their Space Invaders one along with their Pong one, since that's kind of what this game is.

The first frustrating thing about Phaser is that it requires texture files for rendering. So I need to make some images. I shop through a couple different image-generation options (there are a bunch of open-source
pixel-editors, and I can also just grab image data from by beloved html5 canvas constructions), but none of them are appealing and I get frustrated enough that I decide to make all my sprites into [this picture of a duck][duck].
I implement simple paddle motion as per documentation and it kind of works.

The second frustrating thing about Phaser is that their examples like to use a lot of global variables and global static function declarations. I don't like this, so I do a weird hack
where I closure only the options object that I pass to Phaser. Oh well, it works all right.

All right, fine, time to get rid of the ducks as sprites. I have decided to use [svgedit] for image generation. It works well enough. I generate simple geometric shapes for droplets and the paddles. I can
get the paddles to move around fine.

Time to make the droplets fall. Phaser has a well-documented standard way of doing timed loops, so I just write myself into their standard and things seem to work. Droplets will fall every half-second.

Time to do collision detection and scoring. Phaser can do collision detection for me, so I just do it as they say. Scoring is simple. On lose, I'll just pop up an alert for now. Great, things work.

Okay, let's have a real lose condition. I am a little frustrated cycling through the ways to halt the main game loop, and ultimately just wrap the game loop in a
check to see whether the game is over yet. I'll render some text on game end; phaser has this well-documented.

Great, things kind of work. I'll display the score; that's trivial. Now I want keydown to restart the game. Phaser, although it has
`once()` handlers for many input objects, does not have a `once()` handler for the keydown event, so I just have to
bind an event listener by hand. Oh well, that's fine; when I fire the listener I'll null it out. Okay, the game's done.

As a gameplay tweak, I'll have droplets speed up as log of score.

Okay, now the game's really done. This was mainly meant as a tutorial for me; my first foray into Phaser. But enjoy anyway!

## Launch
It's static, so I can use github pages.
