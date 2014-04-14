---
layout: page
title: One Hack Per Day
---
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
