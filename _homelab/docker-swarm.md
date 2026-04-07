---
title: "Infrastructure"
layout: page
permalink: /homelab/docker-swarm/
---

## Infrastructure Series

{% assign docs = site.homelab 
  | where: "series", "docker-swarm" 
  | sort: "order" %}

{% for doc in docs %}
### {{ forloop.index }}. [{{ doc.title }}]({{ doc.url }})

{{ doc.excerpt }}

{% endfor %}
