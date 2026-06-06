---
layout: default
title: "Home"
---

Welcome to my site. Below you will find my latest notes and articles.

## Blog Posts

<ul>
  {% for post in site.posts %}
    <li>
      <strong>{{ post.date | date: "%Y-%m-%d" }}</strong> — 
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
