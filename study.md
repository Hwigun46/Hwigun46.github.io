---
layout: page
title: Study
permalink: /study/
---

보안, Android, Web, CS 기본기와 논문 리뷰를 정리하는 공간입니다.

## Paper Review

{% for post in site.posts %}
{% if post.categories contains "study" and post.tags contains "paper-review" %}
- [{{ post.title }}]({{ post.url | relative_url }}) — {{ post.date | date: site.date_format }}
{% endif %}
{% endfor %}

## Concepts

{% for post in site.posts %}
{% if post.categories contains "study" and post.tags contains "concept" %}
- [{{ post.title }}]({{ post.url | relative_url }}) — {{ post.date | date: site.date_format }}
{% endif %}
{% endfor %}

## Android

{% for post in site.posts %}
{% if post.categories contains "study" and post.tags contains "android" %}
- [{{ post.title }}]({{ post.url | relative_url }}) — {{ post.date | date: site.date_format }}
{% endif %}
{% endfor %}

## Web

{% for post in site.posts %}
{% if post.categories contains "study" and post.tags contains "web" %}
- [{{ post.title }}]({{ post.url | relative_url }}) — {{ post.date | date: site.date_format }}
{% endif %}
{% endfor %}

## Security

{% for post in site.posts %}
{% if post.categories contains "study" and post.tags contains "security" %}
- [{{ post.title }}]({{ post.url | relative_url }}) — {{ post.date | date: site.date_format }}
{% endif %}
{% endfor %}

## CS

{% for post in site.posts %}
{% if post.categories contains "study" and post.tags contains "cs" %}
- [{{ post.title }}]({{ post.url | relative_url }}) — {{ post.date | date: site.date_format }}
{% endif %}
{% endfor %}
