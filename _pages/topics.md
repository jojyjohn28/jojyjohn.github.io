---
layout: page
title: "Topics"
permalink: /topics/
nav: true
nav_order: 4
---

{% assign cats = site.categories %}
{% assign sorted = cats | sort %}

<p>Browse posts by topic (categories).</p>

{% for c in sorted %}
{% assign name = c[0] %}
{% assign posts = c[1] %}

  <h2 id="{{ name }}">{{ name | replace: "-", " " | capitalize }}</h2>
  <ul>
    {% for post in posts %}
      <li>
        <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
        <span style="opacity:0.75;">({{ post.date | date: "%b %d, %Y" }})</span>
      </li>
    {% endfor %}
  </ul>
{% endfor %}
