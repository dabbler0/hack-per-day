---
layout: post
title: "Motion Tracking Experiment"
description: ""
category: 
tags:
- coffeescript
- browser
- real_world
- computer_vision
- getUserMedia
- html5_canvas
---
{% include JB/setup %}

## Demo

[http://dabbler0.github.io/track](http://dabbler0.github.io/track)

## Conception

This has almost certainly been done before, but I wanted to get some experience with image processing. The idea is to make a motion tracking interface that can identify a moving object. There are bunch of standard solutions to this, the most widely-used one being optic flow, but I'm starting simple and implementing absolute diffs and mean-shift colour tracking.

<!-- more -->

## Tech
This is going to be static, built in CoffeeScript on the getUserMedia web api.

## Hack

Okay, so first I need to figure out how to capture media and put in in an html5 canvas, where I know how to work. This is well-documented, so I just put some boilerplate code in and things work fine. I can do simple transformations like image inversion.

Okay, let's figure out how we're going to do tracking. A couple preliminary experiments: r/g/b image split, image delta. Image delta seems most promising (absolute diffs) right now.

First iteration tracker does the following: compute absolute diffs, then find the average absolute diff in the top and bottom halves of the image. If the bottom half passes a threshold and the top half does as well within a short time frame (200ms), consider this "swipe up". My demo UI right now is a "curtain" animation, which rolls up on swipe up and down on swipe down. This works okay.

But this is not motion tracking, and I want to get a little bit more precise. The standard approach to OCR segmentation in situations like this seems to be to use vertical/horizontal histograms to identify an object. So maybe the first thing to do here would be to build a vertical histogram of the absolute difference data. I built a little visualization of this and peaks unfortunately seem a little ambiguous when motion is too fast. When motion is slow, though, peaks are right where we want them to be. So now we have two options.

The first option would be to use a segmentation approach and try to find the longest contiguous non-"zero" reigon of absolute difference. The issue is that for monochromatic moving objects this is not very big, and it is difficult to define what "zero" is because there's so much background noise (every webcam vibrates a tiny bit). So instead what I'm going to do is find the average value from the histogram, and pretend that that's the centroid of the object (which it isn't). This is a tad inaccurate, so I artificially inflated the weight of the histogram by squaring it. This is totally non-rigorous, but it seems to work all right for motion detection.

I reimplemented the same thing for the other axis. Now I have a motion tracking algorithm that kind of works. I want to register gestures again, like I had before. The first thing to do is to cut noise. I realized at this point in hacking that I was actually sampling video faster than it streamed in, so often times two consecutive images were identical. In these cases I don't want to do any motion detection. In fact, just as general noise-resistance I probably want to cut any data gotten from histograms that are too flat. So I have a minimum motion threshold for capture.

Okay, so now I'm going to have "gesture" be a bunch of captured points, each that happens within 200ms of the last (200ms is arbitrary). My simple gestures will be up/down/right/left; for these I only need to record max/min x/y values for my gesture, not all the poinst. So I did that. I set up some arbitrary thresholds for up/down and right/left, and fired some gesture things. My UI demo at this point is a black square that can move around the screen by gestures.

I found that one of the biggest sources of noise in this hack is the upper arm and shoulder of the hand that's gesturing. It generates a lot of absolute difference. Maybe I want to try a different object tracking algorithm -- how about coulor-based mean-shift?

I'm totally unfamiliar with standard mean-shift algorithms, but I decided to do mine with pixel probabilities based on color. Simple UI -- put your object in a little square in the middle of the screen, I capture average color, then I start tracking the object. There are issues here -- often times there are other objects nearby that attract mean-shift much _more_ than the object itself, especially when the object is moving quickly. So I decided to put this away for a while and go back to motion detection.

Okay, back at motion detection I want to make a more compelling demo. I'll make a slideshow navigable with the motion detection. Here I'll use [Reveal.js][reveal]. This goes up pretty quickly, and I'm done. I'm a little frustrated with this hack because it doesn't work quite properly, but that's because I've been deliberately avoiding the state-of-the-art algorithms. So I'll do more some other day.


## Launch
It's static, so this goes up with Github Pages.

[reveal]: http://lab.hakim.se/reveal-js/#/
