---
layout: post
title: "Fire Minigame"
date: 2014-04-16 22:40:00
description: ""
category: 
tags:
- games
- html5_canvas
- browser
- coffeescript
- github_pages
---
{% include JB/setup %}

## Demo
[http://dabbler0.github.com/fire-game](http://dabbler0.github.com/fire-game)

## Conception
I had just finished the [Unite](/2014/04/16/unite-game/) game and was a little disappointed that the game concept didn't pan out. So I assauged myself by making a super, super simple browser game, based loosely on Fruit Ninja.

<!--more-->

## Tech
It's going to be another HTML5 canvas coffeescript game, fully static.

## Hack
OOP boilerplate: we need a Sprite class, from which inherits the Target class. The Player will just be a special case of Sprite; there is only one instance so any OOP there would just be dogmatic.

Okay, player motion first. This time I want to do something a little more organized with keypresses, so I set up a kind of key-binding system that would trigger on the animation tick. Well, sort of. Anyway, player paddle moves left and right. Great.

Moving enemies. First iteration just has enemies moving at constant speed to the left or right (straight). Collision detection -- easy, since it's all rectangular -- implemented without problems, and I can remove them as necessary. `Array.splice()` in CoffeeScript is awful, because of `Array.length` caching in `for..in`, so I'm just doing something like Python filter() and reinstantiating the array at every remove. No performance issues seen so far.

Okay, I want the screen to be a bit more chaotic, so I'm making enemies bounce up and down. Easy, no problems. I want to add the avoid blocks (reduce lives if hit) -- they're just other instances of Target, but stored somewhere else. I'll just duplicate all the lookups for the enemies for the avoids.

I want the game to get harder as it goes on, so I came up with some arbitrary formalism for how fast enemies/avoids come out. My first iteration was `1/(1+Math.sqrt(score))`, but that seemed to ramp up a little too fast, so I changed to `1/(1+Math.log(1+score))`. Game seems fine.

Great, the game's basically done. I just need to render score and lives. That's trivial. I had some stylistic issues rendering lives, 'cos nothing looked right, and eventually landed on the "health-bar" look; I dunno if that was the right design decision, but oh well.

Memory leak: sprites that go offscreen aren't unlinked. I'll run a little garbage-collection routine every second to filter those out. No problem.

Okay, time to wrap the game in a GAME OVER and ANY KEY TO START. No problems there. The games done! Time to launch.

## Launch
It's static, so github pages does the trick.
