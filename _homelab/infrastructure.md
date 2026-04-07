---
title: "Infrastructure"
layout: page
permalink: /homelab/infrastructure/
---

## Infrastructure Series

{% assign docs = site.homelab 
  | where: "series", "infrastructure" 
  | sort: "order" %}

{% for doc in docs %}
### {{ forloop.index }}. [{{ doc.title }}]({{ doc.url }})

{{ doc.excerpt }}

{% endfor %}
