---
layout: post
title: "MUR Part 2: Somehow Turned Into Minecraft?"
description: ""
category: 
tags:
- games
- multiplayer
- browser
- heroku
- html5_canvas
- github_pages
- websockets
- coffeescript
- bootstrap
---
{% include JB/setup %}

## Demo
[http://dabbler0.github.io/terra/](http://dabbler0.github.io/terra/)

## Conception
Hi, guys, it's been a while. This is the spiritual successor to the [multi-user roguelike](http://dabbler0.github.io/hack-per-day/2014/09/16/multi-user-roguelike-part-1/) I tried to make earlier. The three games I've probably wasted the most time on recently are Terraria, RotMG, and various roguelikes (ADoM, nethack, ToME). The idea here was to combine the three: a massively-multiplayer, explorative, player-driven RPG with Minecraft and D&D elements. It'll be rotatably isometric (in the RotMG style), with proper shadowcasting, and support for factions and complex skill trees.

<!--more-->

## Tech
It'll be browser-based, of course. I considered briefly using WebGL (since all my rendering will be rectangular textures with matrix ops applied), but my ArchLinux instance right now is a bit of mess and I haven't configured WebGL properly and don't have the patience to figure it out. So, HTML5 canvas it is, which is fine, since that's what I know how to do anyway. My first draft will be single-player, but when I do the server-side it'll be nodejs on socket.io 1.0, and I'll launch to Heroku. For textures, I will use my [own pixel editor](http://dabbler0.github.io/hack-per-day/2014/06/01/just-a-gorram-pixel-editor/).

## Hack
Rendering first, because that's the only way that I can motivate myself to do anything anymore. I've decided that all of my isometric tiles are going to be rectangular prisms. Isometric rendering of rectangular prisms is pretty easy; it takes a little bit of trigonometric guess-and-check but I have a rotating isometric cube within a couple minutes.

Time to lay down some OOP for doing this en masse. How about a `Tile` has a `Terrain` and may have an `Obstacle`, and `Terrain` and `Obstacle` own textures. Then a `Board` is a collection of `Tiles`, and the `Board` will eventually know how to do shadowcasting etc. Rendering involves looking at a radius of tiles around a position and drawing them all.

Okay, this works. But we get weird cases where the z-index ordering isn't right and it looks like some tiles are floating. So, time to figure out how to sort. When I have a bunch of tiles, I want the isometric protrusions to appear on top of the floor tiles, and I want "closer" isometric protrusions to be on top of "farther" ones. So I do rendering in two steps:
  1. Draw the floor tiles.
  2. Go through all the protrusions and perpendicular-project their position vectors onto the camera angle vector to get their "distance" from the viewer. Sort by this, then render the protrusions in order.

Okay, that works fine. I can get a simple demo up with orthogonal movement, too. Time to make a real player. To mark the player position, I'll just draw a red square in the middle of the screen. Now I need to prevent movement into blocks. This is a tad subtle, as I want the effect where if you run into a slanted wall you'll "slide" along it. So movement should happen along some kind of parallel component to the wall. Luckily, all the walls are axis-aligned, so this is as simple as removing components of the velocity vector. This works pretty instantly. Immediately, we run into the issue that it's nearly impossible to fit the player through a one-tile corridor. I decide that this is fine behavior for now and I'll fix it later.

This is supposed to be a roguelike-inspired game, so we need to pull in our shadowcasting code from the MUR codebase. The code is nicely modularized, so this goes off without much difficulty, and we get nice visual effects.

Time to make some interaction. First, I want to register some click handlers on tiles. This isn't difficult; I can get a raw mouse click coordinate relative to the canvas, rotate and translate it to fit the game grid, and round to get the clicked tile. My dummy behavior is to add or remove a stone obstacle on the tile, which is a nice effect. Great! We're ready for real tools.

This means we'll need to make an inventory and items system. I don't want to do any more rendering, so I'll just dispay these with the DOM. We'll need some additional OOP: a `Player` extends a `Mob`; every `Mob` and every `Tile` has an `Inventory`, which is a collection of `Item`s. For display convenience, I'll give `Inventory` a `change` events to which we can listen and update our DOM display. Each `Player` will have a record of which of its inventory items is selected for use. Each `Item` will have a method describing its behavior when used on a tile. For instance, I'll make the Pickaxe item, which removes a stone tile if it exists.

To have Minecraft-esque behavior, I'll give each `Obstacle` a `drops` property, which is an `Inventory` that gets dumped onto the `Tile`'s `Inventory` when the block is destroyed. The `Stone` obstacle, for instance (the only obstacle that at this point exists in the game) will drop a `Stone` item. Again, to imitate Micraft, the `Stone` item's use-on-tile behavior will be to _construct_ a `Stone` obstacle, then destroy itself (remove itself from the `Player`'s inventory), to give the illusion that the stone item was actually placed on the tile.

At first, to give the illusion of it taking time to destroy obstacles, I just give each item a probability of destroying an obstacle on any hit. I don't really like this because it's pretty clear that you're not making any progress by hitting blocks and not destroying them. So instead, we should give `Obstacle`s health, and reduce that health by some D&D-style roll every time we hit it. I make some new textures for Minecraft-style "cracks" and modify the `Obstacle` rendering to add these textures to itself if its health is in the right range. This gives good behavior, and I like the feel.

Let's make it a little more visually interesting by adding more obstacles, which I'll need to do before I impelement crafitng anyway. I'll make a tree obstacle. To do this, I need to separate the side textures of obstacles from the top texture, which isn't difficult. Okay, it kind of looks like a tree. I'll make different biomes in the map; I'll have a "caverns" biome to the north and a "forest" biome to the south. I'll bring in the MUR code for generating caverns (the Game of Life thing). It looks cool!

At this point I feel like I want to make the inventory rendering slicker. Instead of a vertical roguelike list, I will adopt RotMG's table-like inventory style. This will just be twenty little canvases, and I'll draw the textures on the canvases whe necessary. To get names and descriptions, the user will hover over them and get a Tooltipster tooltip. I'll put a yellow border around the selected item. Okay, that looks better. At the same time, I'll put in a health bar, which I won't use yet, but will figure out later.

Time to implement Minecraft-style crafting. New OOP: a `Recipe` has a `needs` property and a `creates` property, and can apply itself to an `Inventory`, replacing the items listed in `needs` with the items listed in `creates`. Every time the player inventory changes, I'll search through the list of all possible recipes, check if our inventory fulfills their `needs` list, and add buttons for them at the bottom (also small canvases). For now, the image on the button will just be the first thing in the `creates` list; this may change in the future. Okay, this works fine.

Time to make weapons. I'll do it like RotMG: every weapon shoots bullets in the direction of a click. Let's give each `Item` an optional `shoot` method, which generates a bullet with a velocity and a `strike` callback, which will normally damage the struck mob. I immediately have some problems with inserting the bullets, into the rendering sort, but ultimately treating them as protrusions in the two-stage rendering process seems to work fine.

Of course, I have no idea if this works at all yet, because there are no enemy mobs. If I'm to have enemy mobs, I'll need support for some kind of AI. So, `Mob` gets a `tick` callback on every tick, during which it can modify its velocity and examine the surrounding board. For now, I'll put in a dummy AI that just goes in random directions and uses its first inventory item in the direction of the player. I'll make some new sprites for a Warrior and a Rogue enemy and give them Swords and Daggers. I'll make `Mob`s like `Obstacle`s; when a `Mob` dies, it will dump its inventory on the nearest tile. Simultaneously, I need to make the HP bar, which I put in earlier and never used, work properly. I'll just update it on every tick. I don't run into any significant issues here, and I have a pretty fun-to-play game.

Before I move on, I want to make the game multiplayer. This is a little more daunting than I had anticipated, because network issues will be such a problem. To see a demo of the multiplayer result, you can look at [gaies.herokuapp.com](http://gaies.herokuapp.com). I'll do a write-up of the multiplayer thing in a bit.
