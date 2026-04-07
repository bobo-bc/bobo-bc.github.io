---
title: "Infrastructure"
layout: page
permalink: /homelab/talos/
---

## Infrastructure Series

{% assign docs = site.homelab 
  | where: "series", "talos" 
  | sort: "order" %}

{% for doc in docs %}
### {{ forloop.index }}. [{{ doc.title }}]({{ doc.url }})

{{ doc.excerpt }}

{% endfor %}
