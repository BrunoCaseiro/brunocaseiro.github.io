---
layout: single
author_profile: true
title: Posts
permalink: /posts.html
header:
  overlay_image: /assets/images/banner.jpg
---

<ul>
{% for post in site.posts %}
  {% assign currentdate = post.date | date: "%Y" %}
  {% if currentdate != date %}
    <h2>{{ currentdate }}</h2>
    {% assign date = currentdate %} 
  {% endif %}
    <h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
{% endfor %}
</ul>
