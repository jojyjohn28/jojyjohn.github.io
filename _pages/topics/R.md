---
layout: page
title: "Topic: R"
permalink: /topics/R/
---

{% assign posts = site.categories.R | sort: "date" | reverse %}

<ul>
{% for post in posts %}
  <li>
    <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    <span style="opacity:0.75;">({{ post.date | date: "%b %d, %Y" }})</span>
  </li>
{% endfor %}
</ul>
