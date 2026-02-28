---
layout: page
title: "Topic: Machine Learning"
permalink: /topics/machine-learning/
---

{% assign posts = site.categories.machine-learning %}

<ul>
{% for post in posts %}
  <li>
    <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    <span style="opacity:0.75;">({{ post.date | date: "%b %d, %Y" }})</span>
  </li>
{% endfor %}
</ul>
