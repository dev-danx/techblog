---
layout: home
title: Tech Blog
---

# Tech Blog

A blog about .NET, distributed systems, and software engineering.

## Recent Articles

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
      <span class="post-date">{{ post.date | date: "%B %d, %Y" }}</span>
    </li>
  {% endfor %}
</ul>

## Topics

- **.NET** - C#, F#, and the .NET ecosystem
- **Distributed Systems** - Microservices, Orleans, actor frameworks
- **DevOps** - CI/CD, Kubernetes, cloud-native
