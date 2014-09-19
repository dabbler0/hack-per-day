---
layout: page
title: One Hack Per Day
description: "I'm Anthony Bau. I like AI, NLP, signal processing, automation in general, and CS education. My lingua prima is CoffeeScript; I also know JavaScript, Python, C++, Java, and Scheme.<br/>
This is a place for me to post the little hacks that I do in my free time. Every hack is prototyped within about three hours."
---
{% include JB/setup %}

<ul class="posts">
  {% for post in site.posts %}
    <li>
      <h2 class="the-post-link"><a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></h2> <span>{{ post.date | date_to_string }}</span> &raquo; 
      <div class="the-post-excerpt">
        {{ post.excerpt }}
      </div>
    </li>
  {% endfor %}
</ul>
