---
layout: page
title: Research
permalink: /research/
nav: true
nav_order: 4
---

# Research

연구 아이디어, 실험 기록, 프로젝트별 진행 상황을 정리하는 공간입니다.

## Research Logs

{% assign research_posts = site.posts | where_exp: "post", "post.categories contains 'research-log'" %}
{% for post in research_posts %}
- [{{ post.title }}]({{ post.url | relative_url }}) — {{ post.date | date: "%Y-%m-%d" }}
{% endfor %}

## Projects

{% assign sorted_projects = site.projects | sort: "importance" %}
{% for project in sorted_projects %}
- [{{ project.title }}]({{ project.url | relative_url }}) — {{ project.description }}
{% endfor %}
