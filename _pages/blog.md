layout: default
permalink: /blog/
title: Blog
nav: true
nav_order: 3

pagination:
enabled: false
collection: posts
permalink: /page/:num/
per_page: 5
sort_field: date
sort_reverse: true
trail:
before: 1
after: 3

---

<div class="post">

{% assign blog_name_size = site.blog_name | size %}
{% assign blog_description_size = site.blog_description | size %}

{% if blog_name_size > 0 or blog_description_size > 0 %}

  <div class="header-bar">
    <h1>{{ site.blog_name }}</h1>
    <h2>{{ site.blog_description }}</h2>
  </div>
{% endif %}

<div class="card p-3 mb-4">
  <h2>About This Blog</h2>
  <p>
    Welcome to <strong>Daily Bioinformatics from Jojy’s Desk</strong> — my living notebook of 
    daily bioinformatics work, microbial ecology, MAGs, MTX/MG, HPC troubleshooting, and coding.
  </p>

  <ul>
    <li>Metagenome & metatranscriptome analysis</li>
    <li>MAGs, viruses, CAZymes, energy metabolism markers</li>
    <li>Functional redundancy (FRed) modeling</li>
    <li>Hybrid-assembly & whole-genome workflows</li>
    <li>Machine-learning MAG binning (GPU/CPU)</li>
    <li>Daily troubleshooting, R/Python tips, figures</li>
  </ul>

  <p>I post short updates daily. Scroll down for the latest posts ↓</p>
</div>

<ul class="post-list">
  {% assign postlist = site.posts %}
  {% for post in postlist %}
    <li>
      <h3><a class="post-title" href="{{ post.url | relative_url }}">{{ post.title }}</a></h3>
      <p>{{ post.description }}</p>
      <p class="post-meta">{{ post.date | date: '%B %d, %Y' }}</p>
    </li>
  {% endfor %}
</ul>

</div>
