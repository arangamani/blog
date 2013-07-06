---
layout: page
title: Welcome to my Blog!
tagline: A place to scribble about things that interest me
---
{% include JB/setup %}

## Recent Posts

Check out some of my posts here:

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
