---
title: "Infrastructure"
layout: page
permalink: /homelab/k3s/
---

## Infrastructure Series

{% assign docs = site.homelab 
  | where: "series", "k3s" 
  | sort: "order" %}

{% for doc in docs %}
### {{ forloop.index }}. [{{ doc.title }}]({{ doc.url }})

{{ doc.excerpt }}

{% endfor %}
