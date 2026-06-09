---
layout: page
title: HYU
permalink: /hyu/
nav: true
nav_order: 2
---

# HYU

학교 수업 과제와 주차별 학습 정리를 기록하는 공간입니다.

## 클라우드프로그래밍

{% assign cloud_programming_posts = site.posts | where_exp: "post", "post.categories contains 'cloud-programming'" %}
{% for post in cloud_programming_posts %}
- [{{ post.title }}]({{ post.url | relative_url }}) — {{ post.date | date: "%Y-%m-%d" }}
{% endfor %}

## 클라우드 활용

{% assign cloud_usage_posts = site.posts | where_exp: "post", "post.categories contains 'cloud-usage'" %}
{% for post in cloud_usage_posts %}
- [{{ post.title }}]({{ post.url | relative_url }}) — {{ post.date | date: "%Y-%m-%d" }}
{% endfor %}
