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
    <h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
    {{ post.excerpt | markdownify | strip_html }}
{% endfor %}
</ul>
