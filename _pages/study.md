---
layout: page
title: Study
permalink: /study/
nav: true
nav_order: 3
---

# Study

보안, Android, Web, CS 기본기를 정리하는 공간입니다.

## Security

{% assign security_posts = site.posts | where_exp: "post", "post.categories contains 'security'" %}
{% for post in security_posts %}
- [{{ post.title }}]({{ post.url | relative_url }}) — {{ post.date | date: "%Y-%m-%d" }}
{% endfor %}

## Android

{% assign android_posts = site.posts | where_exp: "post", "post.categories contains 'android'" %}
{% for post in android_posts %}
- [{{ post.title }}]({{ post.url | relative_url }}) — {{ post.date | date: "%Y-%m-%d" }}
{% endfor %}

## Web

{% assign web_posts = site.posts | where_exp: "post", "post.categories contains 'web'" %}
{% for post in web_posts %}
- [{{ post.title }}]({{ post.url | relative_url }}) — {{ post.date | date: "%Y-%m-%d" }}
{% endfor %}

## CS

{% assign cs_posts = site.posts | where_exp: "post", "post.categories contains 'cs'" %}
{% for post in cs_posts %}
- [{{ post.title }}]({{ post.url | relative_url }}) — {{ post.date | date: "%Y-%m-%d" }}
{% endfor %}
