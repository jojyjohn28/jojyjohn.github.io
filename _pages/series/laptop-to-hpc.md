---
layout: page
title: "From Laptop to HPC: Scaling Computational Biology Workflows"
permalink: /series/laptop-to-hpc/
---

{% assign posts = site.posts | where: "series", "laptop-to-hpc" | sort: "order" %}

<p>
Modern bioinformatics quickly outgrows a personal laptop. This series walks step-by-step through the transition from local analyses to high-performance computing (HPC) clusters used in genomics and microbiome research.
</p>

<p>
Across six short posts, we cover the practical foundations of computational biology infrastructure — from running your first command on a cluster to building scalable, reproducible workflows.
</p>

<h3>Series Overview</h3>

<ul>
{% for post in posts %}
  <li>
    <strong>Day {{ post.order }}:</strong>
    <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    <span style="opacity:0.75;">({{ post.date | date: "%b %d, %Y" }})</span>
  </li>
{% endfor %}
</ul>

<h3>What You'll Learn</h3>

<ul>
<li>Why laptops struggle with large genomics datasets</li>
<li>How HPC clusters distribute compute resources</li>
<li>Software installation on shared systems</li>
<li>Running jobs using SLURM</li>
<li>Parallelizing analyses across many samples</li>
<li>Building reproducible pipelines with workflow managers</li>
</ul>

<p>
The goal is to make HPC less intimidating and help researchers move from small local analyses to scalable bioinformatics workflows.
</p>
