---
layout: page
title: "Viromics Series"
permalink: /series/viromics/
---

{% assign posts = site.posts | where: "series", "viromics" | sort: "order" %}

<p>This page lists all posts in the Viromics Series in order.</p>

<ul>
{% for post in posts %}
  <li>
    <strong>Day {{ post.order }}:</strong>
    <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    <span style="opacity:0.75;">({{ post.date | date: "%b %d, %Y" }})</span>
  </li>
{% endfor %}
</ul>
