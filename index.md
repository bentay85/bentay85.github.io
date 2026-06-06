---
layout: default
title: "Home"
---

Welcome to Ben's Tech Notebook—a central repository dedicated to tracking technical projects from initial concept to functional execution. This site serves as an open-source archive of deployment runbooks, architectural breakdowns, and hands-on solutions to complex engineering challenges. By documenting the exact steps, edge cases, and configurations discovered along the way, this notebook ensures every build is reproducible, modular, and optimized for future reference.

### The Register  

<ul>
  {% for post in site.posts %}
    <li>
      <strong>{{ post.date | date: "%Y-%m-%d" }}</strong> — 
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
