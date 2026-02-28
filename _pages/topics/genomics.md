---
layout: page
title: "Topic: Genomics"
permalink: /topics/genomics/
---

{% assign posts = site.categories.genomics | sort: "date" | reverse %}

<p>This page lists all posts related to genome assembly, annotation, comparative genomics, and genome-resolved workflows.</p>

<ul>
{% for post in posts %}
  <li>
    <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    <span style="opacity:0.75;">({{ post.date | date: "%b %d, %Y" }})</span>
  </li>
{% endfor %}
</ul>
