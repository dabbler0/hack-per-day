---
layout: post
title: "Just a Gorram Pixel Editor"
date: 2014-06-01 20:00:00
description: ""
category: 
tags:
- browser
- coffeescript
- html5_canvas
- github_pages
---
{% include JB/setup %}

Demo
----
[http://dabbler0.github.io/jgpe/]([http://dabbler0.github.io/jgpe/)

Conception
----------
  The concept for this one is a little dated; a while ago I was looking for a really simple pixel sprite editor and couldn't find one. It looks like someone has [made one here](http://pixieengine.com/) now. Anyway, I felt like this task was inanely simple and it was pretty ridiculous someone had not just made a pixel editor I can use right away, so I made one.
<!--more-->

Tech
----
  It's going to be in CoffeeScript, running on HTML5 canvas. This'll be convenient, 'cos then we can get image data with `toDataURL()`. Bootstrap UI.

Hack
----
  All right, let's lay the main UI elements down. Width and height, and brush colour are parameters, there's the main canvas, and there's a download button. Great. First functionality: resize the canvas based on width and height parameters. Done without issues.

  Okay, next we want to bind to click on the canvas and draw a square where we've clicked. Also not a problem. I also want to be able to click and drag to draw, so we'll do that instead, so we bind to mousedown/mousemove/mouseup.

  I want a better colour picker than entering hex. [This one][colourpicker] looks good. I'll drop it in. No issues there either.

  Okay, time to allow for download. This is easy; we'll just pop a window with the data URL and have the user save it. Also done pretty quickly. Let's allow for multiple formats -- looks like html5 canvas support jpg, webp, and png.

  I want a way to change the size of a single "pixel"; luckily I have "pixel" size abstracted as a constant in the code right now, so I'll just make it mutable and add an input (call it "Fatbit size" or something).

  Okay, one of the other key functionalities of a pixel editor is a fill function. I'll implement a quick BFS filler; since I didn't have the foresight to save editor state, I'll just literally look at pixels on the canvas to see what the colours are for fill-stopping. "Transparent" looks like "black" but without opacity, so I'll check for that as well. Great, that's done too.

  I need an eraser; this should be pretty easy. But first I want to be able to tell the difference between "white" and "empty". I'll use the checkerboard background and just set it as a css repeating background for the main canvas. I'll generate this image using the editor itself (cool). Looks like css allows us to scale the image background with the `background-scale` property; so I'll modify that dynamically based on the Fatbit Size parameter. Great, looks good. Now that this is done, laying down an eraser is pretty trivial; I'll just clear rectangles.

Launch
------
  It's static, so github pages.

[colourpicker]: http://www.eyecon.ro/colorpicker/
