---
layout: page
title: Mathematics and Machines
tagline: What is "pure" today could be "applied" tomorrow
---

A blog about mathematics and its applications to computers and the "real world"

Complete list of posts: 

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

