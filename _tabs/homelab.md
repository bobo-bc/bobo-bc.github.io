---
layout: page
title: Homelab
icon: fas fa-server
order: 5
permalink: /homelab/infrastructure/
---

## Infrastructure Series

{% assign series_posts = site.homelab | where: "series", "infrastructure" | sort: "order" %}

<ol>
  {% for post in series_posts %}
    <li><a href="{{ post.url | relative_url }}">{{ post.title }}</a></li>
  {% endfor %}
</ol>