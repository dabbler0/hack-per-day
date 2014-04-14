---
layout: page
title: One Hack Per Day
---
{% include JB/setup %}

<ul class="posts">
  {% for post in site.posts %}
    <li>
      <h2 class="the-post-link"><a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></h2> <span>{{ post.date | date_to_string }}</span>
      <div class="the-excerpt">
        {{ post.excerpt }}
      </div>
      <a href="{{ BASE_PATH }}{{ post.url }}">Read more.</a>
    </li>
  {% endfor %}
</ul>
