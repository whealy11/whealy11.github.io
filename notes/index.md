---
layout: default
title: Notes
permalink: /notes/
---

# Notes

Short explanations of prerequisite ideas and notation.

## Prerequisites

{% for item in site.pages %}
  {% if item.kind == "prerequisite" %}
- [{{ item.title }}]({{ item.url }})
  {% endif %}
{% endfor %}

