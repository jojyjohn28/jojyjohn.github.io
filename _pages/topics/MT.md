---
layout: page
title: "Topic: Metatranscriptomics"
permalink: /topics/MT/
---

<h2>🧬 Metatranscriptomics Posts</h2>

<p>
This section contains posts related to metatranscriptomics workflows, transcript-level functional profiling,
RNA-based microbial activity analysis, differential expression, pathway interpretation,
and bioinformatics pipelines used for environmental and microbiome transcriptomics studies.
</p>

{% assign posts = site.categories.MT | sort: "date" | reverse %}

{% if posts.size > 0 %}

<ul>
{% for post in posts %}
  <li>
    <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    <span style="opacity:0.75;">({{ post.date | date: "%b %d, %Y" }})</span>
  </li>
{% endfor %}
</ul>
{% else %}
<p>No metatranscriptomics posts added yet.</p>
{% endif %}
