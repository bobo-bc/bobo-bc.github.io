---
layout: page
title: k3s Series
icon: fas fa-server
order: 6
permalink: /homelab/k3s/
---

{% assign docs = site.homelab | where: "series", "k3s" | sort: "order" %}

| #                     | Post                |
| --------------------- | ------------------- |
| {% for doc in docs %} | {{ forloop.index }} | [{{ doc.title }}]({{ doc.url }}) |
{% endfor %}