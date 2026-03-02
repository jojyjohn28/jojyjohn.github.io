---
layout: page
title: "Applied Statistics for Microbiome Data"
permalink: /series/applied-stats/
---

{% assign posts = site.posts | where: "series", "applied-stats" | sort: "order" %}

<p>This page lists all posts in the Applied Statistics for Microbiome Data Series in order.</p>

<ul>
{% for post in posts %}
  <li>
    <strong>Day {{ post.order }}:</strong>
    <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    <span style="opacity:0.75;">({{ post.date | date: "%b %d, %Y" }})</span>
  </li>
{% endfor %}
</ul>
