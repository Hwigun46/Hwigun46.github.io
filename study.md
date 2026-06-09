---
layout: page
title: Study
permalink: /study/
---

# Study

보안, Android, Web, CS 기본기를 정리하는 공간입니다.

## Security

{% for post in site.posts %}
{% if post.categories contains "study" and post.tags contains "security" %}
- [{{ post.title }}]({{ post.url | relative_url }}) — {{ post.date | date: "%Y-%m-%d" }}
{% endif %}
{% endfor %}

## Android

{% for post in site.posts %}
{% if post.categories contains "study" and post.tags contains "android" %}
- [{{ post.title }}]({{ post.url | relative_url }}) — {{ post.date | date: "%Y-%m-%d" }}
{% endif %}
{% endfor %}

## Web

{% for post in site.posts %}
{% if post.categories contains "study" and post.tags contains "web" %}
- [{{ post.title }}]({{ post.url | relative_url }}) — {{ post.date | date: "%Y-%m-%d" }}
{% endif %}
{% endfor %}

## CS

{% for post in site.posts %}
{% if post.categories contains "study" and post.tags contains "cs" %}
- [{{ post.title }}]({{ post.url | relative_url }}) — {{ post.date | date: "%Y-%m-%d" }}
{% endif %}
{% endfor %}
