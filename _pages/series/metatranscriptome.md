---
layout: page
title: "Metatranscriptome Analysis Series"
permalink: /series/metatranscriptome/
---

{% assign posts = site.posts | where: "series", "metatranscriptome" | sort: "order" %}

<p>
This series walks through metatranscriptomic analysis from raw RNA reads to biological interpretation, using reproducible workflows and toy datasets.
</p>

<ul>
{% for post in posts %}
  <li>
    <strong>Day {{ post.order }}:</strong>
    <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    <span style="opacity:0.75;">({{ post.date | date: "%b %d, %Y" }})</span>
  </li>
{% endfor %}
</ul>
