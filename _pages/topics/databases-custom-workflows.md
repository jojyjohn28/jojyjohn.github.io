---
layout: page
title: "Topic: Databases & Custom Workflows"
permalink: /topics/databases-custom-workflows/
---

{% assign posts = site.categories.databases | sort: "date" | reverse %}

<p>This page lists all posts related to building, customizing, and using reference databases for bioinformatics workflows, including GTDB, HUMAnN3, KEGG/UniRef mapping, custom gene catalogues, and environmental metagenomics databases.</p>

<ul>
{% for post in posts %}
  <li>
    <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    <span style="opacity:0.75;">({{ post.date | date: "%b %d, %Y" }})</span>
  </li>
{% endfor %}
</ul>
