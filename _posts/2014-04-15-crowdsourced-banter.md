---
layout: post
title: "Crowdsourced Banter"
description: ""
category: 
tags:
 - GAE
 - art
 - crowdsourcing
 - Python
 - coffeescript
 - browser
---
{% include JB/setup %}

## Demo

[http://crowdsourced-banter.appspot.com](http://crowdsourced-banter.appspot.com)

*Note*: there are currently NO conversation paths you can add to on this demo. This means you won't be able to do anything. Wait a day or so, I'll add some seeds.

## Conception
A hack inspired by [this xkcd][xkcd] and The Social Network. I want to do something webcomicky without doing anything. This'll be done by having the whole "comic" be a self-moderating forum where people suggest responses for a script of witty banter. Voting up/down will be done by Social Network-style comparison using [Elo's chess ranking algorithm][elo]. At the end of every day we'll prune all the bad stuff and ultimately get a tree of some good, dry witty banter.
<!--more-->

## Tech
This kind of simple data I/O is perfect for GAE's frameworks, so I'll be hosting there. Frontend is a standard kind of interaction, so Bootstrap is a good option for frontend styling. Frontend scripts will be in CoffeeScript, as always.

## Hack
First things first, let's make our database structure. Fundamental unit of data: a Line (as in a line of witty banter). A line knows its Elo rating, its text, the line that came before it, and all the lines that came after it (we need to build a doubly-linked tree). Great, no problems.

Let's build the GAE endpoints for rating and writing. Writing is trivial; push a new Line with given parent and text. Great, no issues. Rating is less trivial; we have to implement Elo's ranking algorithm, but this is well-documented so I had no issues here either.

Time for read op. We need a random lookup, and a lookup by id (for permalinks). Both are simple... right? I don't know -- for some reason `Line.get_by_id(n)` has stopped working for me (probably something dumb), but I can force this to work by using `ndb.Key().get()` instead, so I just swapped and stopped worrying about it. I'm hacking! For random lookup I'm just bringing stuff into RAM -- it's a hack, I don't expect data to be too big -- and choosing randomly from RAM array. Okay, so now lookup works pretty well. GAE is a breeze.

UI time. Let's load in some Bootstrap glyphicons. I want to do a big-square-button cover page, something like [pencilcode.net], so I'm just going to mimic their style as much as possible. They use table layouts, so I will too; they look fine. Cover page is done without problems.

Time for the write-endpoint. I'll write a simple AJAX script to load some random witty banter from our read endpoint... oh, hang on a sec, there's a mechanic we haven't implemented yet.

I want to get approximately one new depth of lines per day. I'll achieve this by having the set "seed points" for the day (lines that you can respond to in the crowdsource dsecion), and only letting people append to those (then advancing at the end of the day). So I need a flag in the database for whether the point is a daily "seed point" or not. Added without difficulties. I set up a cron job endpoint for advancing every 24hr. Great, we're on our way to completion.

Performance issue: read op sometimes takes a while. I'm not dealing with this, it's a network issue, so instead I'm just putting a loading glyph in. Oops, bootstrap doesn't have a loading glyph, so I'll use their animated progress bar instead. Quick frantic scripting for the Rating page, and we're done! We've got a product.

## Launch
All I need to do with GAE is push! It's live. This was a startingly easy hack (I give most of my thanks to GAE), which is good, because I had a lot less time today for hacking.
