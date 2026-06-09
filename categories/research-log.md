---
layout: default
title: Research Log
permalink: /categories/research-log/
---

# Research Log

{% for post in site.categories.research-log %}
- [{{ post.title }}]({{ post.url | relative_url }}) — {{ post.date | date: "%Y-%m-%d" }}
{% endfor %}
