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

## Notation

{% for item in site.pages %}
  {% if item.kind == "notation" %}
- [{{ item.title }}]({{ item.url }})
  {% endif %}
{% endfor %}
