---
layout: tab
title: Docker Swarm
icon: fa-solid fa-layer-group
order: 5
permalink: /swarm/
---

# Docker Swarm Tutorial

{% assign swarm_pages = site.pages | where_exp:"p","p.path contains 'swarm/'" | sort: "title" %}

<ul>
{% for page in swarm_pages %}
  <li><a href="{{ page.url }}">{{ page.title }}</a></li>
{% endfor %}
</ul>
