---
layout: default
title: Home
---

# Hwigun Blog

학교 과제, Android Security 학습, 연구 기록을 정리하는 블로그입니다.

## Sections

- [Cloud Programming](/categories/cloud-programming/)
- [Cloud Usage](/categories/cloud-usage/)
- [Android Security](/categories/android-security/)
- [Research Log](/categories/research-log/)

## Recent Posts

{% for post in site.posts limit:10 %}
- [{{ post.title }}]({{ post.url | relative_url }}) — {{ post.date | date: "%Y-%m-%d" }}
{% endfor %}
