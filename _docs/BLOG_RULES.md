# Blog Writing Rules

이 문서는 hwigun.log 블로그 글 작성 규칙을 정리한 내부 문서입니다.
Jekyll post 작성 시 파일명, front matter, category, tag 규칙을 헷갈리지 않기 위해 사용합니다.

---

## 1. Post 파일명 규칙

Jekyll post 파일명은 반드시 아래 형식을 따른다.

```text
YYYY-MM-DD-title-slug.md
```

예시:

```text
2026-06-09-cloud-programming-week01.md
2026-06-09-android-intent-basic.md
2026-06-09-claimbind-static-analysis-log.md
```

규칙:

* 날짜는 `YYYY-MM-DD` 형식으로 작성한다.
* 파일명은 영어 소문자와 하이픈 `-`을 사용한다.
* 한글 파일명은 사용하지 않는다.
* 공백은 사용하지 않는다.
* 제목이 길어도 파일명은 짧고 의미 있게 작성한다.

좋은 예:

```text
2026-06-09-cloud-programming-week01.md
2026-06-10-docker-basic.md
2026-06-11-android-intent-basic.md
```

나쁜 예:

```text
클라우드프로그래밍 1주차.md
2026_06_09_cloud.md
week 1 정리.md
```

---

## 2. Post 저장 위치

모든 글은 `_posts/` 아래에 작성한다.

현재 디렉토리 구조는 다음을 기준으로 한다.

```text
_posts/
├── hyu/
│   ├── cloud-programming/
│   └── cloud-usage/
├── study/
│   ├── security/
│   ├── android/
│   ├── web/
│   ├── cs/
│   ├── cloud/
│   └── system/
└── research/
    ├── claimbind/
    ├── ebpf/
    ├── android-security/
    ├── paper-review/
    └── experiment-log/
```

디렉토리는 사람이 관리하기 위한 분류이고, 실제 블로그 분류는 front matter의 `categories`, `tags`로 결정한다.

---

## 3. Front Matter 기본 규칙

모든 post 파일 상단에는 반드시 YAML front matter를 작성한다.

기본 형식:

```yaml
---
layout: post
title: "글 제목"
date: YYYY-MM-DD
categories: [큰분류]
tags: [세부분류, 추가태그]
---
```

주의:

* `layout: post`는 반드시 작성한다.
* `title`은 블로그 화면에 보이는 제목이다.
* `date`는 파일명 날짜와 맞춘다.
* `categories`는 큰 분류만 사용한다.
* `tags`는 세부 분류를 표현한다.

---

## 4. Category 규칙

category는 큰 구역만 사용한다.

허용 category:

```yaml
categories: [hyu]
categories: [study]
categories: [research]
```

사용하지 말 것:

```yaml
categories: [HYU, cloud-programming]
categories: [study, android-security]
categories: [research, claimbind]
```

이렇게 하면 sidebar category가 지저분해진다.

---

## 5. Tag 규칙

tag는 세부 section을 표현한다.

### HYU

클라우드프로그래밍:

```yaml
categories: [hyu]
tags: [cloud-programming, assignment]
```

클라우드 활용:

```yaml
categories: [hyu]
tags: [cloud-usage, assignment]
```

### Study

Android:

```yaml
categories: [study]
tags: [android, security]
```

Web:

```yaml
categories: [study]
tags: [web, security]
```

CS:

```yaml
categories: [study]
tags: [cs, operating-system]
```

System:

```yaml
categories: [study]
tags: [system, linux]
```

Cloud:

```yaml
categories: [study]
tags: [cloud, aws]
```

### Research

ClaimBind:

```yaml
categories: [research]
tags: [claimbind, android-security]
```

eBPF:

```yaml
categories: [research]
tags: [ebpf, system-security]
```

Paper Review:

```yaml
categories: [research]
tags: [paper-review, android-security]
```

Experiment Log:

```yaml
categories: [research]
tags: [experiment-log, static-analysis]
```

---

## 6. HYU 글 템플릿

### 클라우드프로그래밍

파일 위치:

```text
_posts/hyu/cloud-programming/YYYY-MM-DD-title-slug.md
```

템플릿:

```markdown
---
layout: post
title: "클라우드프로그래밍 N주차 학습 정리"
date: YYYY-MM-DD
categories: [hyu]
tags: [cloud-programming, assignment]
---

## 학습 주제

## 핵심 개념

## 실습 내용

## 정리

## 느낀 점
```

### 클라우드 활용

파일 위치:

```text
_posts/hyu/cloud-usage/YYYY-MM-DD-title-slug.md
```

템플릿:

```markdown
---
layout: post
title: "클라우드 활용 N주차 학습 정리"
date: YYYY-MM-DD
categories: [hyu]
tags: [cloud-usage, assignment]
---

## 학습 주제

## 핵심 개념

## 실습 내용

## 정리

## 느낀 점
```

---

## 7. Study 글 템플릿

파일 위치 예시:

```text
_posts/study/android/YYYY-MM-DD-android-intent-basic.md
```

템플릿:

```markdown
---
layout: post
title: "Android Intent 기본 구조"
date: YYYY-MM-DD
categories: [study]
tags: [android, security]
---

## 학습 목적

## 핵심 개념

## 구조 이해

## 예시

## 정리
```

---

## 8. Research 글 템플릿

파일 위치 예시:

```text
_posts/research/claimbind/YYYY-MM-DD-static-analysis-log.md
```

템플릿:

```markdown
---
layout: post
title: "ClaimBind Static Analysis Log"
date: YYYY-MM-DD
categories: [research]
tags: [claimbind, android-security]
---

## Goal

## Context

## Method

## Observation

## Current Verdict

## Next Step
```

연구 글에서는 runtime evidence가 없으면 강한 표현을 사용하지 않는다.

사용 가능한 표현:

```text
candidate
hypothesis
needs runtime validation
inconclusive
observed statically
```

피해야 할 표현:

```text
vulnerability confirmed
exploitable
proven
definitely vulnerable
```

---

## 9. 작성 후 확인 명령어

새 글 작성 후:

```bash
git status
git add -A
git commit -m "Add blog post"
git push origin main
```

로컬에서 Jekyll을 실행할 경우:

```bash
bundle exec jekyll serve
```

브라우저에서 확인:

```text
http://localhost:4000
```

---

## 10. 원칙

* 파일명은 영어 slug로 작성한다.
* 제목은 한글로 작성해도 된다.
* category는 큰 분류만 사용한다.
* tag로 세부 section을 나눈다.
* 과제 제출용 글은 HYU 하위 page에서 확인 가능해야 한다.
* 연구 글은 과장하지 않고 evidence와 limitation을 함께 적는다.

---

## Research / Study Policy

이 블로그에서 `Research`와 `Study`는 명확히 구분한다.

### Study

아래 항목은 모두 `Study`에 작성한다.

- 논문 리뷰
- 개념 공부
- Android / Web / CS / System / Cloud 학습 기록
- 연구 아이디어를 만들기 위한 배경 학습
- 공개 가능한 수준의 학습 노트

Front matter 예시:

~~~yaml
categories: [study]
tags: [paper-review, android-security]
~~~

또는:

~~~yaml
categories: [study]
tags: [concept, android]
~~~

### Research

`Research`에는 완료된 프로젝트만 작성한다.

작성 가능한 항목:

- 논문 제출 또는 프로젝트 종료 후 정리
- 공개 가능한 최종 결과
- 실험 결과가 정리된 completed project
- 발표 / 포스터 / 코드 / 논문과 연결 가능한 산출물

작성하지 말 것:

- 진행 중인 취약점 후보
- runtime 검증 전 claim
- disclosure 전 민감한 finding
- 미완성 연구 아이디어
- 내부 실험 로그
- 과장된 vulnerability claim

Research 글의 기본 원칙:

~~~text
완료된 것만 쓴다.
검증된 것만 쓴다.
공개 가능한 것만 쓴다.
과장하지 않는다.
~~~

### Paper Review 사고 흐름

논문 리뷰는 단순 요약으로 끝내지 않는다.

반드시 아래 지점을 본다.

1. 저자가 문제를 어떻게 정의했는가
2. 기존 연구의 gap을 어떻게 만들었는가
3. 핵심 insight가 무엇인가
4. contribution이 무엇인가
5. novelty / scalability / practical impact / automation 중 어디에 강점이 있는가
6. 실험 설계가 claim을 충분히 지지하는가
7. 저자들이 직접 말하지 않은 hidden limitation은 무엇인가
8. story line이 어떻게 구성되어 있는가
9. 내 연구나 학습에 가져올 수 있는 것은 무엇인가
