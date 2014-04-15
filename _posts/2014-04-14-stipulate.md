---
layout: post
title: "Stipulate"
description: ""
category: 
tags:
- chemistry
- math
- scripts
- languages
- documents
- JISON
- browser
- coffeescript
- github_pages
---
{% include JB/setup %}

## Demo
[http://dabbler0.github.io/stipulate/](http://dabbler0.github.io/stipulate/)

## Conception
  A hack inspired by chemistry class. I get annoyed doing chem homework because I have to retype my expression in two different places -- my typeset document and my calculator. I also have a bunch of auxiliary scripts (find molar mass, etc.) that my calculator of course doesn't recognize, so I also have to have a Python shell open simultaneously. I want to consolidate my scripts, my calculator, and my typesetter into one thing, so I can just type my expressions and have it "show my work" for me.
<!--more-->

## Tech
  I'm writing a typesetting language, so I'll need a parser. Today I'm going to try the [PEG.js parser generator][pegjs] (*update*: I switched to [JISON]). For typesetting I'm going to use [MathJax][mathjax].  It's going to be a static webapp, written in CoffeeScript.

## Hack
  First things first, language specification. Here's the first draft of the Stipulate language:
```
Chem lookup functions:
  mass(chemical)
  H(reaction)
  Hf(chemical)
  Ka(acid)
  Kb(base)

Maths:
  -2
  2 + 3
  2 - 3
  2 / 3
  2 ^ 3
  2e3
  log(2)
  rt(2, 2)

Every Stipulate statement is of the form

variable = (expression)

And is rendered as two steps, one typesetting the expression verbatim and the other subsituting values. For instance:

[OH-] = Kw / 0.1 M
[H+] = Ka(HCl) * [OH-]

Is rendered to

[OH-] = Kw / 0.1 = 1e-14 / 0.1 M = 1e-13 M
[H+] = Ka(HC2H3O2) * [OH-] = 1.8e-5 * 1e-13 M = 1.8e-18 M

A Stipulate document can also have HEADERS and COMMENTS.

A header looks like a Markdown header:

TITLE
=========

A COMMENT is rendered verbatim in plaintext, and looks like
a CoffeeScript comment.

# Then we need to find the value of [OH-]:

```

  After fiddling with the parser, I made a couple language tweaks:
```
## TITLE

mm:{ChEmICaL}
```

  The second one will be rendered as a subscript. I have no real experience writing formal grammars, so I got frustrated immediately when PEG.js didn't support left recursion, which seemed necessary for left-precedence operators like * and / (we need order of ops for rendering, not just computation). So I switched to [JISON], and still had some precedence issues there, but I worked them out pretty quickly.

  Still in the JISON mood, I decided to write the parser for the molar-mass script. This is a much simpler grammar, and I also wrote this in JISON with little difficulty. Then I found [this table][mmtable] of atomic molar masses online and hardocded it (because I'm too lazy to have an amd-style thing), and linked it with the parser to finish the molar-mass script. Great!

  Okay, time for typesetting. I got simple stuff working pretty quickly, but I wanted to do something like LaTeX's `\intertext` for COMMENTS, which MathJax doesn't support. So I just killed equation alignment and used `\textrt`, and things worked pretty well.

  Okay, time for the meat: render and compute. Here I want to lay down some OOP boilerplate for the parse tree, just for clarity. I'll have a type for each operation, and each node will know how to recursively RENDER, and recursively COMPUTE. This was done with little difficulty.

  Then I want to have the intermediary step of subsitution. It's a hack, so what I do here is literally rip through the tree and replace values -- horrible practice, because it makes the Model mutate for a View operation -- but this Model is getting thrown out anyway as soon as this render is over, so it's okay.

  Great! So I kind of have something working. Now, one of the things I do in chem a lot is ask [WolframAlpha] to solve equations for me. But solving equations is trivial, we can just use [Newton's method][nmethod] in, like, 50 lines of code. So I wrote a quick solver in coffeescript and invented the following syntax for Stipulate:
```
SOLVE (varname) | (left_expression) = (right_expression)
```
  Which will set (varname) to the appropriate value in the document scope, and also render nicely.

  This worked will, except that the normal terminate condition for Newton's method is when `right-left` gets close enough to 0. But this is bad, because in Chemistry we're working with a lot of values on the order of `10^-14`, alongisde values on the order of `10^14`, so there's no universal scale. So I'll do a different terminate condition -- when the determined point stops moving. But this had problems too, because the determined point stopped moving at a bunch of other random places for some reason. So I did both conditions -- you must both be close to 0 and be moving slowly in order for me to terminate the Solver. The solver's still only around 60 SLOC -- great!

  Then I found a rendering issue: because parens are transparent in my parser, the following code:
```
(a + b) * (c + d)
```
  Gets rendered to:
```
a + b * c + d
```
  *Even though it computes correctly*. I briefly considered doing something ICE-editor like with parents telling children to wrap themselves in parens. But instead I decided that it would be decent behavior just to have parens in whever there were parens in the srouce code. So I did that, with little difficulty.

  UI time. I put the textarea on the left and the JAX on the right. This seems to work just fine.

  Performance issue: Solver is slow, and typesetting is slow when the document gets to around a page. So, instead of updating on input, I will just poll and update very 2 seconds.

  Demo time. I'll put in a default document.

  *TODO:* I ditched the idea to have this do dimensional analysis with units. That'll be very easy to implement, though. Also, I want to be able to export this to a pdf (might require some server-side). Hacks for tomorrow, or the weekend!

## Launch
  It's a static page, so I can use Github Pages.

[mathjax]: http://www.mathjax.org
[pegjs]: http://pegjs.majda.cz/
[JISON]: http://zaach.github.io/jison/
[mmtable]: http://www.lenntech.com/periodic/mass/atomic-mass.htm
[WolframAlpha]: http://www.wolframalpha.com/
[nmethod]: http://en.wikipedia.org/wiki/Newton's_method
