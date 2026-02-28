---
layout: page
title: "Size-Fractionated Microbiome Series"
permalink: /series/size-fractionated/
---

{% assign posts = site.posts | where: "series", "size-fractionated" | sort: "order" %}

<p>This page lists all posts in the Size-Fractionated Microbiome Series in order.</p>

<ul>
{% for post in posts %}
  <li>
    <strong>Part {{ post.order }}:</strong>
    <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    <span style="opacity:0.75;">({{ post.date | date: "%b %d, %Y" }})</span>
  </li>
{% endfor %}
</ul>
