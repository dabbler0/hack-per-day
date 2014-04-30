---
layout: post
title: "Background Music"
description: ""
category: 
tags:
- art
- coffeescript
- browser
- github_pages
- music
- html5_canvas
---
{% include JB/setup %}

## Demo
[http://dabbler0.github.io/midiprov](http://dabbler0.github.io/midiprov)

## Conception
Part of the point of my future auto-mashup project will be to take a step towards generative music. But music doesn't have to be artistic to be pleasant, and I kind of wanted a soft, unambitious background music to play while I study that doesn't interfere with thinking but nevertheless keeps changing. So I decided to make some randomly-generated jazz-like sounds.
<!--more-->

## Tech
This is going to be static, in CoffeeScript, built on [MIDI.js][midijs].

## Hack
Okay, first thing to do is lay down a chord progression. I write a simple note-to-frequency translator as a test, then wrote a chord-notation-to-chord-frequencies translator.

My first iteration will be all piano, playing pentatonic fragments on top of the full chord. This means I'll also need a chord-to-pentatonic-fragment translator (chord notation -> array of MIDI tones). So I implemented this as well, without much difficulty. To keep things from getting muddy or inaudible, I will mod the pentatonic fragments to within the middle-C range
and the chord voicings to within the octatve.

Time to hook this up to MIDI.js. I'll just use their prebuilt grand piano MIDI file. I'll have it run through the Satin Doll jazz chart and play the chords with random tones from a pentatonic fragment on top. Great! This works fine.

Okay, but a big part of jazz music is patterns and motifs, and this sounds totally chaotic. So I'll just have some significant probability of one fragment just being the previous fragment transposed to the new chord. Great! That works fine too.

To randomize rhythm, I will have a "continue last note" token be a possibility in the fragment, which means to just fail to end the previous note. This sounds all right, so I will keep it.

Time to add drums. MIDI.js comes prepackaged with a synth drum MIDI, so I'll just have the synth drum go off twice a measure. This works great as well.

Okay, now I want to change the melody instrument to alto sax. This isn't difficult. It becomes apparent at this point, however, that I had a typo with both the keyboard voicing and improv that failed to end the notes soon enough -- I didn't notice with piano because the envelope drops off so fast. I fixed this without difficulty.

A couple generation algorithm tweaks: I like it when the saxophone sustains for a long time, and when it has a lot of quick notes, but not in-between; so I'll make two different "types" of measures for the two possibilities. Also, I want "themes" to sound more like each other (when a measure is repeated directly afterward), so I'm actually not going to mod pentatonics to within middle-C. To allow middle-C-range notes when they are not otherwise allowed, however, I will also extend the pentatonic fragment to the pentatonic an octave below.

Okay, time to make a better frontend. I want to do a simple, unobstrusive lightshow-esque thing with a black/white/gold colour scheme. As always, HTML5 canvas. Boilerplate OOP: a `CircleVis` can render itself and animate itself. I'm aiming for a "shimmering scales" sort of look. `CircleVis` will animate themselves smaller, more transparent, and closer to goldenrod. This isn't difficult. I put everything up and it runs great!

This was pretty simple and I guess it was more art than hacking. Anyway, enjoy.

## Launch
It's static, so I can use github pages.
