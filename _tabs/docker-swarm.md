---
layout: page
title: Docker Swarm Series
icon: fas fa-server
order: 3
permalink: /homelab/docker-swarm/
---

{% assign docs = site.homelab | where: "series", "docker-swarm" | sort: "order" %}

| #                     | Post                |
| --------------------- | ------------------- |
| {% for doc in docs %} | {{ forloop.index }} | [{{ doc.title }}]({{ doc.url }}) |
{% endfor %}