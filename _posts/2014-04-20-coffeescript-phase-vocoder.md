---
layout: post
title: "CoffeeScript Phase Vocoder"
description: ""
category: 
tags:
- nodejs
- real_world
- sound
- coffeescript
---
{% include JB/setup %}

## Demo
No online demo today, sorry! You can install the npm tool with `npm install audioshift`.

## Conception
I've been trying to implement a phase vocoder for a while, first in C, then off-the-cuff in CoffeeScript. Today I'm going to try to actually do it, and implement Nasca Octavian PAUL's known-to-work [Paulstretch][paulstretch] algorithm. This will be the first step towards a future CoffeeScript beat detection and ultimately automatic mixing project.
<!--more-->

# Tech
I'm going to be using [fftw3] for my FFTs. This means I'm going to have to build some node.js bindings for it. It's going to be a node.js app in CoffeeScript.

## Hack
Okay, first thing is to build node.js bindings for fftw3. First I'm going to see if anyone else has done this for me -- yep, [Brian Padalino][bpadalino] seems to have done it as an exercise. Unfortunately, though he build it for node.js **&lt;0.5.0**, which uses a different event system than node.js **0.10.0**, which I'll be using. So I have to modify this to work. Node.js migrated from `EIO` to `libuv`; I don't know what the difference is, but "how to upgrade" is well-documented, so I went ahead and upgraded Brian's code to node 0.10.0. I wrote a few unit tests and the fft seems to be working fine. Great, time to move on.

Okay, time to implement the Paulstretch algorithm. It's a little amusing, because the most frustrating piece of writing my phase vocoder earlier was getting the phases to align, and one of Paul's comments is literally "Discard the original phases and put random phases". Welp, I seem pretty dumb.

First iteration is going to try to be a line-by-line translation of Paul's Python into CoffeeScript. This gets annoying fast, because Python has a bunch of Matlab-like syntactic sugar for multiplying vectors, and Paul seems to rely heavily on the `scipy` and `numpy` packages, which are very mature and have no CoffeeScript equivalent ([numericjs] comes close but doesn't cut it). So I gave up on this approach and started implementing his algorithm freehand.

Brief explanation of Paul's algorithm:
```
1. Perform short-time Fourier transform, with windows mostly overlapping (step size << window size)
2. Randomize phases in STFT windows
3. Overlap-add STFT windows with a new step size.
```

I can implement this pretty quickly. FFTW's old quirks came back and tripped me up a little bit -- e.g. FFTW does not normalize transforms, so instead of passing unity test it sends `n -> n * windowSize`.

Bug fix: **node.js ObjectMode writable streams are buggy**. `write(null)` holds special meaning (EOF), which is reasonable, but `write(0)` holds the same meaning. So if I by chance ever write 0 the stream stops. I'm hacking, so I'll solve this by adding `Math.random() * 0.0001` to everything.

I briefly flirted with porting this to the browser for a demo but quickly gave up. The WebAudio api is a huge pain to deal with and is not my focus today. Additionally, pure js ffts (I used numeric.js's implementation) are orders of magnitude slower than fftw3. So I gave up on that.

What became immediately clear when I did this experiment, however, was that my stream i/o is actually buggy. My reader stream loads asynchronously, so my calls to `read()` **often return null**. FFTW3 interpreted this as 0, but numeric.js is very aware. So it becomes clear that I want to migrate to a node.js `Transform` stream model. I reimplement the thing as a `Transform` stream pretty quickly.

This sets me up for the next step, which is pitch transposition. This is easy; all I need to do is stretch a bit further, then pipe the output to a resampler. I write a simple linear-interpolation resampler in under twenty lines and hook it up. Great, things work. I can pitch-translate numbers from Dr. Horrible's sing-along blog (I waste some time doing this).

Okay, time to wrap this as an npm module. I've never actually done this before, but it's pretty well-documented. I'll use [optimist] as my option parser (dunno whether that's a good design decision or not). `node-gyp` is pretty well-supported by npm, I think, so I won't touch that. Command-line interface goes up pretty quickly, and I can publish.

## Launch
I'm launching to the npm official repo. npm makes this very easy.

[bpadalino]: https://github.com/bpadalino/node_fftw/
[paulstretch]: http://hypermammut.sourceforge.net/paulstretch/
[optimist]: https://github.com/substack/node-optimist
[fftw3]: http://www.fftw.org/
[numericjs]: http://www.numericjs.com/
