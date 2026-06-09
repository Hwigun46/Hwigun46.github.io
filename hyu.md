---
layout: page
title: HYU
permalink: /hyu/
---

# HYU

학교 수업 과제와 주차별 학습 정리를 기록하는 공간입니다.

## 클라우드프로그래밍

{% for post in site.posts %}
{% if post.categories contains "hyu" and post.tags contains "cloud-programming" %}
- [{{ post.title }}]({{ post.url | relative_url }}) — {{ post.date | date: "%Y-%m-%d" }}
{% endif %}
{% endfor %}

## 클라우드 활용

{% for post in site.posts %}
{% if post.categories contains "hyu" and post.tags contains "cloud-usage" %}
- [{{ post.title }}]({{ post.url | relative_url }}) — {{ post.date | date: "%Y-%m-%d" }}
{% endif %}
{% endfor %}
