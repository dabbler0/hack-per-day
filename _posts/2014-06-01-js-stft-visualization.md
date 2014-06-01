---
layout: post
title: "JS STFT visualization"
description: ""
category: 
tags:
- nodejs
- node-webkit
- sound
- coffeescript
- real_world
---
{% include JB/setup %}

Demo
----
  No online demo, sorry! It's a node-webkit app, you can get it on [github here][gh].

Conception
----------
  I got a request to do some stuff trying to synth grand piano audio. The first thing I want to do in this situation is see what grand piano audio looks like. So I'll make a quick hack to do a 3D rendering of its STFT, and then decide what the best regression is for it.
<!--more-->

Tech
----
  I already have my node.js bindings for fftw3, so I'll use them. I'll use Greg Ross's implementation of a JavaScript 3D surface plotter (http://javascript-surface-plot.googlecode.com/svn/trunk/googleVizApi.html) for the visualization. Since I need both WebGL/canvas and fftw3, I'll do this in node-webkit.

Hack
----
  First things first, I need to generate the table of frame:frequency:power. This should be pretty easy; I can basically copy the code from [audioshift][audioshift] (update: I cannot; see below). I would be slightly worried about performance here, but audioshift seemed to work out fine, so I'll leave it as is.

  Okay, time to pipe this into my JS 3D surface plotter. Looks simple enough -- I simply need to fill out a Google Viz API table, and pass that table to Greg Ross's thing. I can almost copy his tutorial verbatim. Great!

  Currently my implementation requires the number of STFT frames to take to be passed to it as a parameter. That's a bit inconvenient, so I'll do the natural thing and just read the input file until it ends, and generate STFT frames until then. All right good. Is it done?

  Turns out it's not done, because somehow *all of my stft frames ended up the same*. My first instinct here is that something mutable kept getting passed around -- maybe all of the rows of my power table were actually pointers to the same array? Nope, I pretty explicity reinstantiate each of them. Okay second instinct is that I have problems with asynchronicity. My CS looks perfectly fine, though. I am pretty clearly passing *different* data to `fftw.execute()` and getting the *same* results.

  So clearly, the issue is in Brian Padalino's fftw3 bindings. I probably should not have trusted these in the first place, because some of his comments even said "I don't know what this does." Anyway, here is the issue: to use fftw3, you copy the input data into a preassigned memory location, then call the transformation function, then copy output data out of a different memory location that the transformation function put it into. Brian puts the input data in *synchronously*, at the *moment you call* `execute()`, then *asynchronously* calls the transformation function. So what ends up happening is the input data keeps getting overwritten `n` times, then the `execute()` function is called `n` times on the same data, and the output of that same input data is copied `n` times. I don't want to think about how to strucutre this properly, so I'm just going to make it synchronous.

  Making it synchronous is actually easier than it sounds; I can literally delete about 30 lines of code and not modify the rest and it becomes synchronous. Great, so now things work!

Launch
------
  There's no launch for this. It's really a personal tool. However, if you want to use it, just download the repo; it includes a binary `app.nw`.

[audioshift]: http://dabbler0.github.io/hack-per-day/2014/04/20/coffeescript-phase-vocoder/
[gh]: github.com/dabbler0/audioviz
