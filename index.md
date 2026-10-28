---
layout: default
title: Home
---

# From Code to Hardware

Practical notes on embedded software, bare-metal development, debugging, and hardware-software integration.

## Blog Posts

<ul>
{% for post in site.posts %}
  <li>
    <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    <small> — {{ post.date | date: "%d %B %Y" }}</small>
  </li>
{% else %}
  <li>No posts published yet.</li>
{% endfor %}
</ul>
