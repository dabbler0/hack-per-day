---
layout: post
title: "Animated Chessboard BFS"
tags:
 - visualization
 - coffeescript
 - gh_pages
 - html5_canvas
 - browser
---
{% include JB/setup %}

Demo
----
[http://dabbler0.github.io/bfsanim](http://dabbler0.github.io/bfsanim)

Conception
----------

I managed to waste around 7 hours this weekend playing Achaea, which is the worst thing. So
I came here to regroup.

This one was done mainly for a class I'm taking. We've got to teach a mini-lesson on graphs,
so I figured one of the flashy motivational things would be an animation for searches. Here we find
the shortest path from the upper-left to lower-right corner of the chessboard with different movement
rules and obstacles.
<!--more-->

Tech
----

Back with the standard techs -- CoffeeScript, html5 canvas, gh-pages.

Hack
----

I want some visuals. Let's draw the chessboard background first. This is simple, and is done in a few minutes.

Still in the visuals mindset, I want to do some kind of path rendering. First refactor the background-drawing code into a redraw function, and come up with a path renderer that draws lines and dots along the board for a set of coordinates. Also pretty easy, and we're done.

Okay, time for more meaty stuff. Let's implement the model.

OOP first. We'll have classes `Node` and `Graph`, where a `Node` knows its adjacent nodes and some data and a `Graph` is just a container for a set of nodes. This goes up quickly but can't really be tested.

Now we need to programmatically construct the graph for a chessboard. Let's do this by first representing the board the traditional way (2d array), linking adjacent cells, and then flattening. We will store the coordinates of a `Node` in its data. This goes up surprisingly smoothly and is right on the first compile.

Time to attempt to find a shortest path. I decided that this should be attached to the `Graph` class, because I want a guarantee that the two nodes are actually part of the same graph context.

A chessboard is small, but if I want to scale this, I probably want to store BFS paths as a tree per se for memory conservation reasons. So let's make an upward-pointing `Path` tree for our BFS. Now all we need to do is attach nodes to the tree from the graph in BFS order. Each `Path` node should have a toArray() method that strings together the path from the node to the root.

In order to do proper BFS order I need to know whether a cell has been visited yet. I don't want to attach a flag to the cell itself because two searches should be able to run in parallel -- the state should be local to the search. The easiest way to do this without paying lots of computation time is to give a public object id to `Node`s and keep a hashtable locally. So I do this as well.

Hopefully all of the plumbing will work out, and when I compose the function it will render. I call the renderer on the arrayified result of the BFS. To make things interesting, I find the shortest path between `[0, 0]` and `[5, 6]`. This also goes up surprisingly smoothly, and is correct on the first compile!

Great, time to add obstacle support. This is data inherent to the model, so I'll attach a "blocked" class to a `Node`, and stop the BFS from going over blocked nodes. Before I test this I want some rendering, so I'll also modify the renderer to render blocked nodes in red. Great, that's done, and it works. To get closer to my final product, I shift the destination back to `[7, 7]` (lower-right corner). To make things more interesting, I also make diagonal squares adjacent, so that path length isn't mostly fixed.

Okay, time to make it animated. Animation in JavaScript is a pain because everything immediately has to become recursive. So I refactor my BFS to turn all the iterative state into parameters and turn the loop into a tail-recursive setTimeout call.

I need something to render inside the animation -- let's start with "all the considered paths". I already have a path-rendering function, so I just need to refactor it -- I refactor the path-drawing part into `drawPath_raw` (separate from the background-drawing part). Then I call that on all the considered paths in the animation tick. Voila! An animated BFS.

At this point I declare myself basically done. I also want to add: other adjacency rules, display the graph, display "visited" squares.

Other adjacency rules: At first I want to do this by having the user type a JavaScript function taking a coordinate and returning the adjacent coordinates. This is for a class though, and my classmates don't know JavaScript. So instead, I'll invent a declarative syntax -- a set of newline-delimited space-separated pairs of numbers saying the offsets of the adjacent squares. For example, a knight's move is:
```
+1 +2
-1 +2
+1 -2
-1 -2
+2 +1
-2 +1
+2 -1
-1 -1
```
I write the tiny parser for this quickly, and refactor the adjacency rule out of the board generation. I don't want to refactor anything else, so when the adjacency rule changes, I just rebuild the graph. However, this loses "blocked" node state, so I have to first run through the graph, grab them, then rebuild the graph, then reinsert "blocked" state. This is a big hack, but that's also the point of this exercise, so that's okay.

Displaying visited squares is not a big problem. I don't want to write different functions for animated rendering and nonanimated rendering, so I just have the background renderer always accept a "visited" object, and ignore if it it doesn't exist.

Showing the graph isn't a big problem either, just another renderer tweak. I do this too, put the state in global variables, and slap on a bootstrap frontend for changing everything. That's it! I'm done.

Launch
-----

It's static, so we're using gh-pages, of course.
