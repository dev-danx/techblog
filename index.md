---
layout: home
title: Welcome
---

# Welcome to My Blog

This is a blog powered by **GitHub Pages** and Jekyll.

## Recent Posts

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
      <span class="post-date">{{ post.date | date: "%B %d, %Y" }}</span>
    </li>
  {% endfor %}
</ul>
