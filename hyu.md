---
layout: page
title: HYU
permalink: /hyu/
---

학교 수업 과제와 주차별 학습 정리를 기록하는 공간입니다.

## Courses

- [Cloud Programming]({{ "/hyu/cloud-programming/" | relative_url }})
- [Cloud Usage]({{ "/hyu/cloud-usage/" | relative_url }})

## Recent HYU Posts

{% for post in site.posts %}
{% if post.categories contains "hyu" %}
- [{{ post.title }}]({{ post.url | relative_url }}) — {{ post.date | date: site.date_format }}
{% endif %}
{% endfor %}
