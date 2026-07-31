---
layout: page
title: "Mycobiome Analysis Series"
permalink: /series/mycobiome/
---

{% assign posts = site.posts | where: "series", "mycobiome" | sort: "order" %}

<p>
A step-by-step series covering shotgun metagenomic mycobiome analysis, from fungal read detection and taxonomic profiling to functional characterization and genome-resolved mycobiome workflows.
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
