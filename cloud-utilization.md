---
layout: page
title: Cloud Utilization
permalink: /hyu/cloud-utilization/
---

클라우드 활용 과목의 주차별 학습 정리 페이지입니다.

## Weekly Notes

{% for post in site.posts %}
{% if post.categories contains "hyu" and post.tags contains "cloud-usage" %}
- [{{ post.title }}]({{ post.url | relative_url }}) — {{ post.date | date: site.date_format }}
{% endif %}
{% endfor %}
