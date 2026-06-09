---
layout: page
title: Research
permalink: /research/
---

# Research

연구 아이디어, 실험 기록, 프로젝트별 진행 상황을 정리하는 공간입니다.

## ClaimBind

{% for post in site.posts %}
{% if post.categories contains "research" and post.tags contains "claimbind" %}
- [{{ post.title }}]({{ post.url | relative_url }}) — {{ post.date | date: "%Y-%m-%d" }}
{% endif %}
{% endfor %}

## eBPF

{% for post in site.posts %}
{% if post.categories contains "research" and post.tags contains "ebpf" %}
- [{{ post.title }}]({{ post.url | relative_url }}) — {{ post.date | date: "%Y-%m-%d" }}
{% endif %}
{% endfor %}
