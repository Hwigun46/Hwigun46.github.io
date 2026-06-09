---
layout: page
title: Research
permalink: /research/
---

완료된 연구 프로젝트와 공개 가능한 최종 산출물을 정리하는 공간입니다.

진행 중인 연구 아이디어, 미검증 후보, disclosure 전 finding은 이 페이지에 작성하지 않습니다.

## Completed Projects

{% for post in site.posts %}
{% if post.categories contains "research" and post.tags contains "completed-project" %}
- [{{ post.title }}]({{ post.url | relative_url }}) — {{ post.date | date: site.date_format }}
{% endif %}
{% endfor %}
