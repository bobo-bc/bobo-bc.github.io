---
title: "Kunbernetes k3s"
layout: page
permalink: /homelab/k3s/
---

## k3s Series

{% assign docs = site.homelab 
  | where: "series", "k3s" 
  | sort: "order" %}

{% for doc in docs %}
### {{ forloop.index }}. [{{ doc.title }}]({{ doc.url }})

{{ doc.excerpt }}

{% endfor %}
