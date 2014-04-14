---
layout: post
title: "Autoannotate (Latin)"
---

# {{ page.title }}

[http://vm-0.dabbler.kd.io:8080](http://vm-0.dabbler.kd.io:8080)


## Conception
  Another hack from Latin class. I expect over a third of my time doing Latin homework is spent turning pages in my book to look at line notes or the dictionary. I can fix the second one by automating word lookup.

## Tech
  It's going to be a webapp, frontend in CoffeeScript, backend in nodejs + CoffeeScript. For a dictionary, I'm going to use William Whitaker's [words] tool.

## Hack
  So, my first thought with this hack was to fork Whitaker's tool and port it to CoffeeScript. Unfortunately, it is written in Ada, a language I do not understand and do not have the time to learn for this hack. So, I am just going to shell out to his tool.

  The first task is to make a RESTFUL api version of his tool. I set this up without much difficulty -- I just shell out and spit the result at the client. I flirted briefly with coming up with a well-defined JSON format for it, then decided that I didn't know what format he outputted things in, and that the end-user would probably like his format better than any I could come up with anyway.

  Next task is to write the frontend. There's going to be a textarea there, and after you input, it will somehow annotate for you. I decided to use jquery tooltipster for the annotations, and show annotations on click or hover (ultimately deciding on hover). My first UI iteration had the textarea in the top half of the screen and the annotated text in the bottom half, and update on textarea `input`. I decided that this was ugly and informationally redundant, and collapsed it all into the square you see now.

  I decided that I didn't want to have to type all the words in myself, since Cicero's Oratio In Catilinam I (the text we're working with at the moment) is a very clearly out-of-copyright text that I can scrape with no problem. I scraped it from [this archive](http://www.thelatinlibrary.com/cicero/cat1.shtml) with no problems. For serving, I just line-break-separated it in a file and had the server serve the nth line of the file as the text.

  So now I've successfully annotated all of In Catilinam I! All I need to do now is

## Launch

  ...right? Well, this is the most difficult part of this hack, because I have this dumb dependency written in Ada, and I need cloud hosting. Heroku and GAE are out for this reason, and nothing else I know is free. Right?

  Well, sort of. I registered for a [Koding] account a while back while I was working on an IDE to check out how other people did it. And it turns out Koding gives us all free VMs. Which is great for this project, because I need a VM where I have free reign. So I'm running off Koding.

  I may want to actually port Words to CoffeeScript sometime later, but that will require me to either learn more Latin or learn Ada, which is more of a "week" thing than a "couple hours".

[words]: http://www.archives.nd.edu/words.htm
[Koding]: http://koding.com
