---
layout: post
title: "클라우드프로그래밍 12주차 학습 정리"
date: 2026-06-02
categories: [hyu]
tags: [cloud-programming, assignment]
---

# 클라우드프로그래밍 12주차: Helm, 개발 워크플로, 자기수복형 애플리케이션

## 1. 이번 주차에서 다룬 내용

12주차는 Kubernetes를 운영 관점에서 더 편하게 다루기 위한 내용을 다룬다. Kubernetes 리소스를 YAML로 하나씩 관리할 수는 있지만, 애플리케이션이 커질수록 YAML 파일 수가 많아지고 환경별 설정도 복잡해진다. 이 문제를 줄이기 위해 Helm이 등장한다.

또한 개발 과정에서는 Docker Compose와 Kubernetes 배포 방식을 연결하고, 네임스페이스와 컨텍스트로 워크로드를 분리한다. 마지막으로 readiness probe와 liveness probe, Helm의 atomic 업데이트를 통해 장애 상황에서 애플리케이션을 안전하게 운영하는 흐름을 다룬다.

이번 주차에서 다룬 내용은 다음과 같다.

- Helm의 역할과 차트 구조
- Helm 리포지터리, 릴리스, values
- Helm으로 애플리케이션 설치, 업그레이드, 패키징
- 차트 간 의존 관계 모델링
- Docker 개발 워크플로와 Kubernetes 개발 워크플로
- 네임스페이스와 컨텍스트를 이용한 워크로드 분리
- readiness probe와 endpoint 제어
- liveness probe와 파드 재시작
- Helm `--atomic`을 이용한 안전한 업데이트
- Helm test를 이용한 배포 검증

## 2. Background

- Helm: Kubernetes 애플리케이션 패키지 관리자
- Chart: Kubernetes YAML과 설정값을 묶은 패키지
- Release: Chart를 클러스터에 설치한 인스턴스
- Values: Chart 설치 시 바꿀 수 있는 설정값
- Namespace: 클러스터 안에서 리소스를 논리적으로 분리하는 단위
- Context: `kubectl`이 사용할 클러스터, 사용자, 네임스페이스 조합
- Readiness Probe: 파드가 트래픽을 받을 준비가 되었는지 검사
- Liveness Probe: 파드가 살아 있는지 검사하고 필요하면 재시작

### 보충 설명

Helm은 Kubernetes를 대체하는 도구가 아니다. Kubernetes YAML을 더 쉽게 패키징하고 설치하고 업그레이드하기 위한 도구다.

Docker에서 이미지를 만들고 태그를 붙여 배포하듯이, Kubernetes에서는 여러 YAML을 Chart로 묶어 배포할 수 있다.

## 3. 개념 정리

### 3.1 Helm

Helm은 여러 Kubernetes YAML 정의를 하나의 차트로 묶어 설치, 업그레이드, 공유할 수 있게 해주는 도구다.

애플리케이션 하나를 배포하려면 Deployment, Service, ConfigMap, Secret, Ingress 등 여러 리소스가 필요하다. Helm은 이 리소스 묶음을 하나의 패키지처럼 관리한다.

### 3.2 Chart와 Release

Chart는 설치 가능한 패키지이고, Release는 그 Chart를 실제 클러스터에 설치한 결과다.

같은 Chart라도 이름과 values를 다르게 주면 여러 Release로 설치할 수 있다. 예를 들어 같은 웹 애플리케이션 Chart를 개발용, 테스트용, 운영용으로 다르게 설치할 수 있다.

### 3.3 Values

Values는 Chart에 전달하는 설정값이다. 서비스 포트, 복제본 수, 이미지 태그 같은 값을 설치 시점에 바꿀 수 있다.

```bash
helm install --set servicePort=8010 --set replicaCount=1 ch10-vweb kiamol/vweb --version 1.0.0
```

이 방식은 YAML을 직접 수정하지 않고도 배포 설정을 바꿀 수 있게 해준다.

### 3.4 Chart 의존성

Helm Chart는 다른 Chart에 의존할 수 있다. 예를 들어 애플리케이션이 프록시나 데이터베이스를 필요로 한다면, Chart 안에서 의존 Chart를 함께 관리할 수 있다.

의존성을 잘 정의하면 여러 구성 요소를 한 번에 설치할 수 있다.

### 3.5 개발 워크플로와 네임스페이스

개발 과정에서는 로컬 Docker Compose로 먼저 실행해보고, 이후 Kubernetes에 배포하는 흐름이 나온다.

또한 테스트 환경과 UAT 환경처럼 워크로드를 분리하려면 네임스페이스를 사용한다. `kubectl config set-context --current --namespace=...`를 사용하면 매번 `-n` 옵션을 쓰지 않아도 특정 네임스페이스를 기본 작업 공간으로 사용할 수 있다.

### 3.6 Readiness와 Liveness

readiness probe는 파드가 트래픽을 받을 준비가 되었는지 확인한다. 실패하면 Service endpoint에서 제외되어 트래픽을 받지 않는다.

liveness probe는 컨테이너가 살아 있는지 확인한다. 실패하면 Kubernetes가 컨테이너를 재시작한다.

둘을 구분하지 못하면 장애 대응 흐름을 잘못 이해하게 된다.

### 3.7 Helm atomic 업데이트

`helm upgrade --atomic`은 업그레이드가 실패하면 자동으로 이전 상태로 되돌리는 옵션이다.

운영 배포에서는 새 버전이 실패했을 때 수동으로 복구하는 시간이 길어질 수 있다. atomic 옵션은 이런 위험을 줄이는 데 사용된다.

## 4. 내가 몰랐을 가능성이 높은 개념

| 개념 | 왜 중요한가 | 이해 포인트 |
|---|---|---|
| Helm | Kubernetes 패키징 | 여러 YAML을 하나의 차트로 관리 |
| Chart | 설치 패키지 | 애플리케이션 리소스 묶음 |
| Release | 설치된 인스턴스 | Chart가 클러스터에 배포된 결과 |
| Values | 환경별 설정 | 설치 시점에 설정값 변경 |
| Dependency | 차트 간 관계 | 필요한 서브차트를 함께 관리 |
| Namespace | 워크로드 분리 | 개발, 테스트, 운영 환경 분리 |
| Context | kubectl 작업 환경 | 클러스터/사용자/네임스페이스 조합 |
| Readiness Probe | 트래픽 수신 준비 | 실패 시 endpoint에서 제외 |
| Liveness Probe | 컨테이너 생존 확인 | 실패 시 재시작 |
| Atomic Upgrade | 안전한 업데이트 | 실패 시 자동 롤백 |

## 5. 실습하면서 내 것으로 만들 부분

12주차 실습은 Helm과 운영 안정성 흐름으로 나누어 보는 것이 좋다.

```text
Helm 흐름
→ Helm 설치 확인
→ repo 추가
→ chart 검색
→ values 확인
→ install
→ upgrade
→ package
→ dependency build

개발/운영 흐름
→ Docker Compose로 로컬 실행
→ Kubernetes로 배포
→ namespace/context 분리
→ readiness/liveness 확인
→ Helm atomic update로 안전한 배포
```

### 5.1 Helm 설치와 리포지터리 흐름

```bash
# Linux에서 Helm 설치
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash

# 설치 확인
helm version

# 리포지터리 추가
helm repo add kiamol https://kiamol.net

# 로컬 리포지터리 캐시 업데이트
helm repo update

# 차트 검색
helm search repo vweb --versions
```

`helm repo add`는 원격 차트 저장소를 내 Helm 설정에 등록하는 명령어다. `helm repo update`는 그 저장소의 인덱스를 최신으로 갱신한다.

### 5.2 Chart 설치와 업그레이드 흐름

```bash
# 차트 기본값 확인
helm show values kiamol/vweb --version 1.0.0

# values를 일부 바꿔 설치
helm install --set servicePort=8010 --set replicaCount=1 ch10-vweb kiamol/vweb --version 1.0.0

# 설치된 릴리스 확인
helm ls

# 복제본 수를 변경해 업그레이드
helm upgrade --set servicePort=8010 --set replicaCount=3 ch10-vweb kiamol/vweb --version 1.0.0

# 관련 ReplicaSet 확인
kubectl get rs -l app.kubernetes.io/instance=ch10-vweb
```

Helm에서는 설치된 결과를 Release라고 부른다. `install`은 새 릴리스 생성이고, `upgrade`는 기존 릴리스 변경이다.

### 5.3 직접 만든 Chart 패키징 흐름

```bash
# 예제 디렉터리 이동
cd ch10

# Chart 문법 검사
helm lint web-ping

# Chart 설치
helm install wp1 web-ping/

# 릴리스 확인
helm ls

# values 확인
helm show values web-ping/

# values를 바꿔 설치
helm install --set targetUrl=kiamol.net wp2 web-ping/

# 로그 확인
kubectl logs -l app=web-ping --tail 1
```

`helm lint`는 Chart가 문법적으로 문제가 없는지 확인한다. 배포 전에 먼저 돌려보는 습관이 필요하다.

### 5.4 Chart 저장소와 패키지 업로드 흐름

```bash
# ChartMuseum 설치 예시
helm repo add stable https://charts.helm.sh/stable

# Chart 패키징
helm package web-ping

# 패키지를 저장소에 업로드
curl --data-binary "@web-ping-0.1.0.tgz" <chartmuseum-url>

# repo 갱신 후 검색
helm repo update
helm search repo web-ping
```

이 흐름은 내가 만든 Helm Chart를 다른 사람이 설치할 수 있게 공유하는 과정이다.

### 5.5 네임스페이스와 컨텍스트 흐름

```bash
# 네임스페이스 생성
kubectl create namespace kiamol-ch11-test

# 특정 네임스페이스에 리소스 적용
kubectl apply -f sleep.yaml --namespace kiamol-ch11-test

# 기본 네임스페이스에서 조회
kubectl get pods -l app=sleep

# 특정 네임스페이스에서 조회
kubectl get pods -l app=sleep -n kiamol-ch11-test

# 현재 컨텍스트의 기본 네임스페이스 변경
kubectl config set-context --current --namespace=kiamol-ch11-test

# 현재 컨텍스트 확인
kubectl config view
```

네임스페이스는 리소스를 나누는 공간이고, 컨텍스트는 `kubectl`이 기본으로 바라볼 작업 환경이다.

### 5.6 Readiness와 Liveness 확인 흐름

```bash
# numbers 애플리케이션 적용
cd ch12
kubectl apply -f numbers/

# 준비 상태 대기
kubectl wait --for=condition=ContainersReady pod -l app=numbers-api

# 서비스 endpoint 확인
kubectl get endpoints numbers-api

# API 호출
curl "$(cat api-url.txt)/rng"

# readiness가 추가된 버전 적용
kubectl apply -f numbers/update/api-with-readiness.yaml

# endpoint 변화 확인
kubectl get endpoints numbers-api

# liveness까지 추가된 버전 적용
kubectl apply -f numbers/update/api-with-readiness-and-liveness.yaml

# 파드 상태 확인
kubectl get pods -l app=numbers-api -o wide
```

readiness는 트래픽 수신 여부에 영향을 주고, liveness는 재시작 여부에 영향을 준다. 이 둘은 시험에서도 헷갈리기 쉽다.

### 5.7 Helm atomic 업데이트 흐름

```bash
# 기존 리소스 정리
kubectl delete all -l kiamol=ch12

# Helm으로 설치
helm install --atomic todo-list todo-list/helm/v1/todo-list/

# 실패 시 자동 롤백되는 업그레이드
helm upgrade --atomic --timeout 30s todo-list todo-list/helm/v2/todo-list/

# 테스트 실행
helm test todo-list

# 테스트 Job 로그 확인
kubectl logs -l job-name=todo-list-db-test
```

`--atomic`은 업그레이드 실패 시 자동으로 이전 상태로 되돌리는 옵션이다. `helm test`는 배포 후 검증 작업을 실행할 때 사용한다.

### 5.8 명령어별로 정리

| 명령어 | 언제 쓰는가 | 내가 이해한 의미 |
|---|---|---|
| `helm version` | Helm 설치 확인 | Helm CLI 사용 가능 여부 확인 |
| `helm repo add` | Chart 저장소 추가 | 원격 패키지 저장소 등록 |
| `helm repo update` | 저장소 인덱스 갱신 | 최신 Chart 목록 반영 |
| `helm search repo` | Chart 검색 | 설치 가능한 패키지 확인 |
| `helm show values` | Chart 기본 설정 확인 | 바꿀 수 있는 설정값 확인 |
| `helm install` | Chart 설치 | Kubernetes 리소스 묶음을 배포 |
| `helm upgrade` | Release 업데이트 | 설치된 애플리케이션 설정/버전 변경 |
| `helm lint` | Chart 검사 | 배포 전 문법과 구조 점검 |
| `helm package` | Chart 패키징 | 공유 가능한 `.tgz` 파일 생성 |
| `helm dependency build` | 의존 Chart 준비 | 필요한 하위 Chart 다운로드 |
| `kubectl config set-context` | 기본 네임스페이스 변경 | kubectl 작업 환경 조정 |
| `kubectl get endpoints` | 서비스 연결 대상 확인 | 실제 트래픽을 받을 파드 확인 |

### 5.9 헷갈렸던 지점

Helm Chart와 Docker Image는 둘 다 패키징이라는 느낌이 있지만 대상이 다르다. Docker Image는 컨테이너 실행 환경을 패키징한다. Helm Chart는 Kubernetes 리소스 정의를 패키징한다.

readiness와 liveness도 자주 섞인다. readiness는 “트래픽 받아도 되는가”이고, liveness는 “죽은 것으로 보고 재시작해야 하는가”다.

### 5.10 나중에 다시 볼 포인트

- Helm Chart와 Release의 차이를 설명할 수 있는가?
- `values.yaml`은 어떤 문제를 해결하는가?
- 네임스페이스와 컨텍스트의 차이를 설명할 수 있는가?
- readiness와 liveness를 명확히 구분할 수 있는가?
- `helm upgrade --atomic`이 왜 안전한 배포와 연결되는가?

## 6. 보충 설명

- 자료 기반 내용: Helm, Chart, Release, values, 네임스페이스, 컨텍스트, readiness/liveness, Helm atomic 업데이트를 다룬다.
- 보충 설명: 실무에서는 Helm 외에도 Kustomize, Argo CD, Flux 같은 도구와 함께 GitOps 방식으로 관리하는 경우가 많다.
- 확실하지 않음: 수업 실습 환경에서 ChartMuseum이나 LoadBalancer가 실제 외부 IP를 바로 받는지는 클러스터 구성에 따라 달라진다.

## 7. 한 줄 요약

12주차의 핵심은 Kubernetes 리소스를 Helm으로 패키징하고, 네임스페이스와 probe와 atomic 업데이트를 통해 배포와 운영 안정성을 높이는 것이다.

## 8. 추가로 공부할 키워드

- Helm
- Chart
- Release
- Values
- Namespace
- Context
- Readiness Probe
- Liveness Probe
- Atomic Upgrade
- Helm Test
