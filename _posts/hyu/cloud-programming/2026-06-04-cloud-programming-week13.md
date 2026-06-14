---
layout: post
title: "클라우드프로그래밍 13주차 학습 정리"
date: 2026-06-04
categories: [hyu]
tags: [cloud-programming, assignment]
---

# 클라우드프로그래밍 13주차: Kubernetes 운영 로그, 모니터링, Ingress

## 1. 이번 주차에서 다룬 내용

13주차는 Kubernetes 운영의 기본이 되는 관측 가능성을 다룬다. 애플리케이션이 잘 배포되었는지 확인하는 것과 실제 운영 중 문제를 찾는 것은 다르다. 운영에서는 로그를 모으고, 지표를 수집하고, 외부 트래픽을 적절히 라우팅해야 한다.

이번 주차는 크게 세 부분으로 나눌 수 있다. 첫째, Fluentd/Fluent Bit와 Elasticsearch/Kibana를 이용한 중앙화 로그 관리다. 둘째, Prometheus와 Grafana를 이용한 모니터링이다. 셋째, Ingress를 이용한 HTTP 트래픽 라우팅이다.

이번 주차에서 다룬 내용은 다음과 같다.

- Kubernetes 로그 관리의 한계
- 여러 네임스페이스와 파드 로그 수집
- Fluentd/Fluent Bit 기반 로그 수집기
- Elasticsearch와 Kibana를 이용한 로그 저장과 검색
- Prometheus 기반 메트릭 수집
- Grafana 대시보드
- exporter를 이용한 메트릭 수집
- Ingress와 Ingress Controller
- Host/Path 기반 HTTP 라우팅
- NGINX Ingress와 Traefik 비교

## 2. Background

- 로그: 애플리케이션이나 시스템이 남기는 실행 기록
- 중앙화 로그 관리: 여러 파드와 노드에서 발생한 로그를 한 곳으로 모으는 방식
- Fluentd/Fluent Bit: 로그를 수집하고 전달하는 도구
- Elasticsearch: 로그를 저장하고 검색하기 위한 저장소
- Kibana: Elasticsearch에 저장된 로그를 시각화하고 검색하는 UI
- Prometheus: 메트릭을 수집하는 모니터링 시스템
- Grafana: 수집된 지표를 시각화하는 대시보드 도구
- Ingress: 클러스터 외부 HTTP/HTTPS 요청을 내부 서비스로 라우팅하는 리소스
- Ingress Controller: Ingress 규칙을 실제 프록시 설정으로 반영하는 컴포넌트

### 보충 설명

`kubectl logs`는 문제를 빠르게 확인할 때는 유용하지만, 운영 환경 전체의 로그 관리 도구로는 부족하다. 파드 수가 늘고 네임스페이스가 나뉘면 일일이 로그를 보는 방식은 유지되지 않는다.

## 3. 개념 정리

### 3.1 Kubernetes 로그 관리

컨테이너 로그는 기본적으로 해당 컨테이너가 실행되는 노드의 파일 형태로 저장된다. `kubectl logs`로 특정 파드 로그를 볼 수 있지만, 여러 네임스페이스와 여러 파드를 한 번에 분석하기에는 한계가 있다.

운영 환경에서는 로그 수집기가 각 노드의 로그 파일을 읽어 중앙 저장소로 전달하는 구조를 사용한다.

### 3.2 Fluentd/Fluent Bit

Fluentd 또는 Fluent Bit는 컨테이너 로그를 수집해 다른 저장소로 전달하는 역할을 한다. Kubernetes에서는 보통 DaemonSet 형태로 각 노드에 배치한다.

각 노드에서 발생하는 로그를 수집해야 하므로, 노드마다 로그 수집기가 하나씩 실행되는 구조가 자연스럽다.

### 3.3 Elasticsearch와 Kibana

Elasticsearch는 수집된 로그를 저장하고 검색할 수 있게 해준다. Kibana는 Elasticsearch에 저장된 데이터를 사람이 보기 쉽게 검색하고 시각화하는 도구다.

로그 수집 흐름은 대략 다음과 같다.

```text
애플리케이션 파드 로그
→ 노드 로그 파일
→ Fluentd/Fluent Bit 수집
→ Elasticsearch 저장
→ Kibana 검색/시각화
```

### 3.4 Prometheus와 Grafana

로그가 “무슨 일이 있었는가”를 보는 기록이라면, 메트릭은 “현재 시스템 상태가 어떤가”를 숫자로 보는 지표다.

Prometheus는 애플리케이션이나 exporter가 노출하는 메트릭을 주기적으로 가져와 저장한다. Grafana는 Prometheus 같은 데이터 소스를 연결해 대시보드로 보여준다.

### 3.5 Exporter

Exporter는 기존 시스템이나 애플리케이션의 상태를 Prometheus가 읽을 수 있는 형식으로 바꿔주는 컴포넌트다.

예를 들어 웹 프록시나 데이터베이스가 내부 상태를 직접 Prometheus 형식으로 제공하지 않는다면, exporter가 중간에서 지표를 변환해 노출할 수 있다.

### 3.6 Ingress

Ingress는 외부 HTTP/HTTPS 트래픽을 내부 서비스로 라우팅하는 Kubernetes 리소스다.

Service는 파드에 안정적으로 접근하는 리소스지만, 여러 웹 서비스를 도메인이나 경로 기준으로 나눠 라우팅하려면 Ingress가 필요하다.

### 3.7 Ingress Controller

Ingress 리소스는 규칙일 뿐이다. 실제로 트래픽을 받아서 라우팅하려면 Ingress Controller가 필요하다. NGINX Ingress Controller, Traefik 등이 대표적이다.

Ingress를 만들었는데 동작하지 않는다면 Ingress Controller가 설치되어 있는지 먼저 확인해야 한다.

## 4. 내가 몰랐을 가능성이 높은 개념

| 개념 | 왜 중요한가 | 이해 포인트 |
|---|---|---|
| 중앙화 로그 | 운영 장애 분석 | 여러 파드 로그를 한 곳에서 검색 |
| Fluentd/Fluent Bit | 로그 수집 | 노드 로그를 읽어 중앙 저장소로 전달 |
| Elasticsearch | 로그 저장과 검색 | 수집된 로그를 색인하고 조회 |
| Kibana | 로그 시각화 | Elasticsearch 로그를 UI로 분석 |
| Prometheus | 메트릭 수집 | 주기적으로 지표를 수집 |
| Grafana | 대시보드 | 지표를 시각적으로 확인 |
| Exporter | 지표 변환 | 기존 시스템 지표를 Prometheus 형식으로 노출 |
| Ingress | HTTP 라우팅 | Host/Path 기준으로 서비스 연결 |
| Ingress Controller | 실제 라우팅 수행 | Ingress 규칙을 프록시에 반영 |

## 5. 실습하면서 내 것으로 만들 부분

13주차 실습은 운영 도구가 많아서 흐름으로 나눠야 한다.

```text
로그 흐름
→ 애플리케이션 로그 확인
→ 노드 로그 파일 확인
→ Fluent Bit 배치
→ Elasticsearch/Kibana 배치
→ 로그 수집 경로 확인

모니터링 흐름
→ Prometheus 배치
→ 애플리케이션 배치
→ 메트릭 수집 확인
→ Grafana 대시보드 연결
→ exporter 추가

Ingress 흐름
→ Ingress Controller 배치
→ Ingress 규칙 적용
→ Host/Path 라우팅 확인
→ Controller별 동작 비교
```

### 5.1 기본 로그 확인 흐름

```bash
# 예제 코드 디렉터리로 이동
cd ch13

# timecheck 애플리케이션 배치
kubectl apply -f timecheck/

# 개발 네임스페이스 파드 준비 대기
kubectl wait --for=condition=ContainersReady pod -l app=timecheck -n kiamol-ch13-dev

# 개발 네임스페이스 로그 확인
kubectl logs -l app=timecheck --all-containers -n kiamol-ch13-dev --tail 1

# 테스트 네임스페이스 파드 준비 대기
kubectl wait --for=condition=ContainersReady pod -l app=timecheck -n kiamol-ch13-test

# 테스트 네임스페이스 로그 확인
kubectl logs -l app=timecheck --all-containers -n kiamol-ch13-test --tail 1
```

이 방식은 작은 규모에서는 가능하지만, 파드가 많아지면 관리가 어렵다.

### 5.2 노드 로그 파일 확인 흐름

```bash
# sleep 디플로이먼트 배치
kubectl apply -f sleep.yaml

# 컨테이너 내부 쉘 접속
kubectl exec -it deploy/sleep -- sh

# 컨테이너 로그 파일 경로로 이동
cd /var/log/containers/

# timecheck 로그 파일 확인
ls timecheck*kiamol-ch13*_logger*

# 개발 네임스페이스 로그 마지막 줄 확인
cat $(ls timecheck*kiamol-ch13-dev_logger*) | tail -n 1

# 종료
exit
```

Kubernetes 로그도 결국 노드의 파일 시스템 어딘가에 쌓인다. 로그 수집기는 이 파일을 읽어서 외부 저장소로 보낸다.

### 5.3 Fluent Bit 배치와 설정 변경 흐름

```bash
# Fluent Bit 관련 리소스 배치
kubectl apply -f fluentbit/

# Fluent Bit 파드 준비 대기
kubectl wait --for=condition=ContainersReady pod -l app=fluent-bit -n kiamol-ch13-logging

# Fluent Bit 로그 확인
kubectl logs -l app=fluent-bit -n kiamol-ch13-logging --tail 2

# 로그 매칭 설정 변경
kubectl apply -f fluentbit/update/fluentbit-config-match.yaml

# DaemonSet 재시작
kubectl rollout restart ds/fluent-bit -n kiamol-ch13-logging

# 다시 준비 상태 확인
kubectl wait --for=condition=ContainersReady pod -l app=fluent-bit -n kiamol-ch13-logging
```

로그 수집기 설정을 바꾸면 DaemonSet을 재시작해서 새 설정이 반영되도록 해야 한다.

### 5.4 Elasticsearch와 Kibana 흐름

```bash
# Elasticsearch 배치
kubectl apply -f elasticsearch/

# Elasticsearch 준비 대기
kubectl wait --for=condition=ContainersReady pod -l app=elasticsearch -n kiamol-ch13-logging

# Kibana 배치
kubectl apply -f kibana/

# Kibana 준비 대기
kubectl wait --for=condition=ContainersReady pod -l app=kibana -n kiamol-ch13-logging

# Kibana 접속 URL 확인
kubectl get svc kibana -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:5601'

# Fluent Bit 출력 대상을 Elasticsearch로 변경
kubectl apply -f fluentbit/update/fluentbit-config-elasticsearch.yaml
kubectl rollout restart ds/fluent-bit -n kiamol-ch13-logging
```

이 흐름을 통해 로그가 중앙 저장소로 들어가고, Kibana에서 조회할 수 있는 구조를 만든다.

### 5.5 Prometheus와 Grafana 흐름

```bash
# Prometheus 배치
cd ch14
kubectl apply -f prometheus/

# Prometheus 준비 대기
kubectl wait --for=condition=ContainersReady pod -l app=prometheus -n kiamol-ch14

# Prometheus 접속 URL 확인
kubectl get svc prometheus -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:9090'

# Grafana 배치
kubectl apply -f grafana/

# Grafana 준비 대기
kubectl wait --for=condition=ContainersReady pod -l app=grafana -n kiamol-ch14-monitoring

# Grafana 접속 URL 확인
kubectl get svc grafana -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:3000'
```

Prometheus는 지표를 수집하고, Grafana는 그 지표를 보기 좋게 보여준다.

### 5.6 Exporter 추가 흐름

```bash
# 프록시에 exporter 추가
kubectl apply -f todo-list/update/proxy-with-exporter.yaml

# 파드 준비 대기
kubectl wait --for=condition=ContainersReady pod -l app=todo-proxy -n kiamol-ch14-test

# exporter 로그 확인
kubectl logs -l app=todo-proxy -n kiamol-ch14-test -c exporter

# Grafana 대시보드 업데이트
kubectl apply -f grafana/update/grafana-dashboard-todo-list-v2.yaml
kubectl rollout restart deploy grafana -n kiamol-ch14-monitoring
```

Exporter는 기존 애플리케이션 옆에서 Prometheus가 읽을 수 있는 지표를 만들어준다.

### 5.7 Ingress Controller와 Ingress 규칙 흐름

```bash
# 예제 디렉터리 이동
cd ch15

# 기본 애플리케이션 배치
kubectl apply -f hello-kiamol/

# 서비스 확인
kubectl get svc hello-kiamol

# NGINX Ingress Controller 배치
kubectl apply -f ingress-nginx/

# Ingress Controller 서비스 확인
kubectl get svc -n kiamol-ingress-nginx

# Ingress 규칙 적용
kubectl apply -f hello-kiamol/ingress/localhost.yaml

# Ingress 확인
kubectl get ingress
```

Ingress 규칙만으로는 트래픽이 흐르지 않는다. 반드시 Ingress Controller가 있어야 한다.

### 5.8 명령어별로 정리

| 명령어 | 언제 쓰는가 | 내가 이해한 의미 |
|---|---|---|
| `kubectl logs --all-containers` | 파드의 모든 컨테이너 로그 확인 | 멀티컨테이너 로그를 함께 확인 |
| `kubectl exec -it ... -- sh` | 컨테이너 내부 쉘 접속 | 파일 시스템이나 설정 직접 확인 |
| `kubectl rollout restart ds/...` | DaemonSet 재시작 | 로그 수집기 설정 반영 |
| `kubectl get svc ... -o jsonpath` | 서비스 접속 URL 추출 | LoadBalancer 주소를 명령어로 뽑기 |
| `kubectl scale deploy` | 파드 수 조정 | 부하나 테스트를 위해 복제본 변경 |
| `kubectl apply -f ingress-nginx/` | Ingress Controller 설치 | Ingress 규칙을 실제 라우팅으로 반영할 컴포넌트 배치 |
| `kubectl get ingress` | Ingress 규칙 확인 | HTTP 라우팅 규칙 조회 |
| `kubectl logs -c exporter` | exporter 로그 확인 | 메트릭 변환 컨테이너 상태 확인 |

### 5.9 헷갈렸던 지점

로그와 메트릭은 다르다. 로그는 사건의 기록이고, 메트릭은 상태의 숫자다. 장애 분석에서는 둘 다 필요하다.

Ingress와 Service도 다르다. Service는 파드 집합에 접근하는 내부적 안정 지점이고, Ingress는 외부 HTTP 요청을 Host/Path 기준으로 서비스에 연결하는 규칙이다.

### 5.10 나중에 다시 볼 포인트

- `kubectl logs`만으로 운영 로그 관리가 어려운 이유는 무엇인가?
- Fluent Bit는 왜 DaemonSet으로 배치되는가?
- Prometheus와 Grafana의 역할은 어떻게 다른가?
- exporter는 어떤 문제를 해결하는가?
- Ingress와 Ingress Controller는 왜 둘 다 필요한가?

## 6. 보충 설명

- 자료 기반 내용: 중앙화 로그 관리, Fluent Bit, Elasticsearch/Kibana, Prometheus/Grafana, Ingress 라우팅을 다룬다.
- 보충 설명: 운영 환경에서는 로그 보존 기간, 인덱스 정책, 알림 규칙, 대시보드 권한까지 함께 설계해야 한다.
- 확실하지 않음: 자료 제목에는 플루언트디가 등장하지만 실습 명령어는 `fluentbit` 디렉터리를 기준으로 진행된다. 두 도구는 로그 수집 계열 도구라는 점에서 연결되지만 구현체는 구분해야 한다.

## 7. 한 줄 요약

13주차의 핵심은 Kubernetes 운영에서 로그, 메트릭, Ingress를 통해 애플리케이션 상태를 보고 외부 트래픽을 제어하는 방법을 이해하는 것이다.

## 8. 추가로 공부할 키워드

- Fluentd
- Fluent Bit
- Elasticsearch
- Kibana
- Prometheus
- Grafana
- Exporter
- Ingress
- Ingress Controller
- NGINX Ingress
- Traefik

