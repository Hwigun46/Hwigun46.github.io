---
layout: page
title: Articles
permalink: /articles/
---

전체 글을 큰 분류 기준으로 탐색하는 페이지입니다.

## Recent Posts

{% for post in site.posts limit:10 %}
- [{{ post.title }}]({{ post.url | relative_url }}) — {{ post.date | date: site.date_format }}
{% endfor %}

## HYU

{% for post in site.posts %}
{% if post.categories contains "hyu" %}
- [{{ post.title }}]({{ post.url | relative_url }}) — {{ post.date | date: site.date_format }}
{% endif %}
{% endfor %}

## Study

{% for post in site.posts %}
{% if post.categories contains "study" %}
- [{{ post.title }}]({{ post.url | relative_url }}) — {{ post.date | date: site.date_format }}
{% endif %}
{% endfor %}

## Research

{% for post in site.posts %}
{% if post.categories contains "research" %}
- [{{ post.title }}]({{ post.url | relative_url }}) — {{ post.date | date: site.date_format }}
{% endif %}
{% endfor %}
