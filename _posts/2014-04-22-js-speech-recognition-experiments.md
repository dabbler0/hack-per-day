---
layout: post
title: "JS Speech Recognition Experiments"
description: ""
category: 
tags:
- sound
- signal_analysis
- coffeescript
- getUserMedia
- webaudio
- browser
- github_pages
---
{% include JB/setup %}

## Demo
[http://dabbler0.github.io/webfft/](http://dabbler0.github.io/webfft/)

## Conception
So I've successfully made a phase vocoder. Sometime in the future I'll want to try beat detection, but what seems a little easier is naive speech recognition -- just distinguishing between two vowel sounds, say "eh" and "oo". Apparently the standing way to do this is cosine similarity on a fourier transform. So that's what I'll be implementing today.
<!--more-->

## Tech
It'll be a static webpage in CoffeeScript, on the `getUserMedia` and `AudioContext` apis.

## Hack
Okay, first thing to do is to get a real-time DFFT of sound data. Locally I did this with fftw, but I don't have that here. At first I used [jsfft], which was pretty fast, but then read the AudioContext specs and found that the browser should have a native implementation, `analyserNode`, so I used that. I hooked this up to a canvas and pretty quickly had real-time frequency power plots of the data (`analyserNode` does not provide phase data, but that's okay, since all I'm doing is speech recognition and not vocoding).

Okay, time to implement cosine similarity. This is very easy. I'm going to try to be disciplined here and use typed arrays wherever possible, because our window sizes are going to be constant.

This tests all right. The issue is that cosine similarity treats each frequency independently, so the algorithm becomes _very_ sensitive to pitch changes. So I'm flirting with moving to [Proscutes distance][pdistance] as a measure.

Okay, I'm going to try Proscutes Distance. Boilerplate OOP: a `ProscutesFunction` can be translated and scaled. I'm only doing partial analysis because I don't expect data to have rotations or reflections applied. Implementation of Proscutes is not difficult. However, I am getting rather inaccurate results, so I am going to move back to cosine similarity.

Okay, back at cosine similarity. Time to implement some activation thresholds. This is hack, and I don't really want to rigorise this, so after staring a little while at `-log(1 - cosine_similarity)` I've determined that we want to trigger whenever this value goes above 5. So I put this up. To kind of "dehackify" it some, I'll measure this relative to the average `-log(1 - cosine_similarity)` value -- my arbitrary threshold will then be `-log(1 - cosine_similarity) > 0.5 + avg(-log(1 - cosine_similarity))`. To compute this average, I'm going to be a little lazy and just have a geometric rolling average; each new average is a weighted average of the last average and the new value. This has some advantages; for instance, if ambient sound changes, the system can adjust. Mainly, though I didn't want to do proper sampling, so I did this instead. It throws off the beginning of collection, but that's okay.

Okay, demo time. I'll just throw up a simple Bootstrap UI and have some "recording" buttons. I'll keep the FFT visualization. I'll have "RED" and "BLUE" trigger a color change -- at first I had it change the page background color, but this was awful, so now it changes the FFT visualization color. Roll up a table of the similarity values, and we're done! Great.

## Launch
It's static, so I can launch to github pages.
