---
title: "Infrastructure Series"
layout: page
permalink: /homelab/infrastructure/
---

{% assign docs = site.homelab | where: "series", "infrastructure" | sort: "order" %}

<div class="series-index">
  {% for doc in docs %}
  <div class="series-card">
    <span class="series-order">{{ forloop.index }}</span>
    <div class="series-card-content">
      <a href="{{ doc.url }}"><h4>{{ doc.title }}</h4></a>
      {% if doc.description %}<p>{{ doc.description }}</p>{% endif %}
    </div>
  </div>
  {% endfor %}
</div>