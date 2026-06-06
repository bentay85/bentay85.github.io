---
layout: default
title: "Home"
---

Welcome to Ben's Tech Notebook, a site dedicated to tracking my projects from initial concept to functional execution. This is an archive of deployment runbooks, architectural breakdowns, and hands-on solutions of my projects. By documenting the exact steps and configurations discovered along the way, this notebook ensures every build is reproducible for future reference.  

<ul>
  {% for post in site.posts %}
    <li>
      <strong>{{ post.date | date: "%Y-%m-%d" }}</strong> — 
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
