---
layout: page
title: ClaimBind
description: Android communication boundary에서 payload-carried claim과 실제 caller identity의 재검증 문제를 분석하는 연구 프로젝트
img: assets/img/12.jpg
importance: 1
category: research
---

# ClaimBind

ClaimBind는 Android communication boundary에서 전달되는 claim과 실제 caller identity 사이의 불일치 가능성을 분석하는 연구 프로젝트입니다.

## Goal

Android 앱, SDK, framework boundary에서 security-relevant decision이 발생하기 전에 payload-carried claim이 실제 호출자 identity에 적절히 rebind 또는 validation되는지 확인합니다.

## Current Scope

- Manifest boundary exposure
- Static source/sink indexing
- Boundary-specific precheck
- Candidate ranking
- Manual review
- Runtime validation
