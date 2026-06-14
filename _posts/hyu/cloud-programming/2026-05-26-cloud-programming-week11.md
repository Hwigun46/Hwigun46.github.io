---
layout: post
title: "클라우드프로그래밍 11주차 학습 정리"
date: 2026-05-26
categories: [hyu]
tags: [cloud-programming, assignment]
---

# 클라우드프로그래밍 11주차: 멀티컨테이너 파드, StatefulSet, 롤아웃과 롤백

## 1. 이번 주차에서 다룬 내용

11주차는 Kubernetes 고급 내용으로 넘어간다. 9주차와 10주차가 파드, 디플로이먼트, 서비스, 볼륨 같은 기본 리소스를 익히는 과정이었다면, 이번 주차는 파드를 더 실무적으로 사용하는 방법을 다룬다.

핵심은 세 가지다. 첫째, 하나의 파드 안에 여러 컨테이너를 넣어 역할을 나누는 방식이다. 둘째, StatefulSet으로 상태가 있는 애플리케이션을 안정적으로 다루는 방식이다. 셋째, Deployment의 롤아웃과 롤백을 통해 애플리케이션을 안전하게 업데이트하는 방식이다.

이번 주차에서 다룬 내용은 다음과 같다.

- 파드 안의 여러 컨테이너가 통신하는 방식
- 공유 볼륨과 localhost 기반 통신
- 초기화 컨테이너를 이용한 시작 전 준비 작업
- 어댑터 컨테이너를 이용한 로그와 관리 방식 표준화
- StatefulSet의 안정적 이름과 순서 보장
- StatefulSet에서 초기화 컨테이너 활용
- VolumeClaimTemplate을 이용한 파드별 저장소 요청
- Deployment 롤링 업데이트
- 롤아웃 상태 확인과 롤백
- 롤링 업데이트 설정값

## 2. Background

- 멀티컨테이너 파드: 하나의 파드 안에 여러 컨테이너가 함께 실행되는 구조
- 공유 볼륨: 같은 파드 안의 컨테이너들이 함께 사용할 수 있는 저장 공간
- 초기화 컨테이너: 본 컨테이너 실행 전에 먼저 실행되는 컨테이너
- 어댑터 컨테이너: 기존 애플리케이션의 출력이나 인터페이스를 표준 형태로 바꾸는 컨테이너
- StatefulSet: 상태가 있는 애플리케이션을 위한 Kubernetes 컨트롤러
- 롤링 업데이트: 기존 파드를 조금씩 새 버전으로 교체하는 배포 방식
- 롤백: 문제가 생긴 배포를 이전 버전으로 되돌리는 작업

### 보충 설명

파드는 단순히 컨테이너 하나를 담는 껍데기가 아니다. 여러 컨테이너가 네트워크와 볼륨을 공유하는 실행 단위다. 이 점을 이해해야 초기화 컨테이너, 사이드카, 어댑터 패턴이 자연스럽게 이해된다.

## 3. 개념 정리

### 3.1 파드와 컨테이너 통신

파드는 하나 이상의 컨테이너가 공유하는 네트워크와 파일 시스템을 제공한다. 같은 파드 안의 컨테이너들은 같은 파드 IP를 사용하고, 서로 통신할 때 `localhost`를 사용할 수 있다.

다만 같은 파드 안에 있어도 프로세스는 분리되어 있다. 각 컨테이너는 자신의 이미지, 환경 변수, 프로세스를 가진다. 그래서 서로 다른 역할의 컨테이너를 하나의 실행 단위로 묶을 수 있다.

### 3.2 공유 볼륨

같은 파드 안의 여러 컨테이너는 하나의 볼륨을 서로 다른 방식으로 마운트할 수 있다. 예를 들어 한 컨테이너는 쓰기 가능으로 마운트하고, 다른 컨테이너는 읽기 전용으로 마운트할 수 있다.

이 구조는 메인 컨테이너가 파일을 만들고, 보조 컨테이너가 그 파일을 읽거나 변환하는 방식에 적합하다.

### 3.3 초기화 컨테이너

초기화 컨테이너는 애플리케이션 컨테이너가 시작되기 전에 실행되는 컨테이너다. 설정 파일 생성, 외부 의존성 확인, 초기 데이터 준비 같은 작업에 사용할 수 있다.

초기화 컨테이너가 성공해야 본 컨테이너가 실행된다. 실패하면 파드는 정상 준비 상태로 가지 않는다.

### 3.4 어댑터 컨테이너

어댑터 컨테이너는 기존 애플리케이션이 제공하는 로그나 출력을 운영 환경에서 다루기 쉬운 형태로 바꾸는 역할을 한다.

애플리케이션 코드를 직접 수정하지 않고, 옆에 보조 컨테이너를 붙여 표준화된 인터페이스를 제공할 수 있다는 점이 중요하다.

### 3.5 StatefulSet

StatefulSet은 상태가 있는 애플리케이션을 배포할 때 사용한다. Deployment와 달리 파드 이름과 생성 순서, 저장소 연결을 안정적으로 관리한다.

예를 들어 데이터베이스처럼 각 인스턴스의 정체성이 중요한 경우에는 단순한 Deployment보다 StatefulSet이 더 적합하다.

### 3.6 VolumeClaimTemplate

StatefulSet에서는 각 파드가 자신만의 저장소를 가져야 할 수 있다. 이때 VolumeClaimTemplate을 사용하면 파드마다 PVC를 자동으로 생성할 수 있다.

즉, `todo-db-0`, `todo-db-1` 같은 파드가 각각 자신의 데이터 볼륨을 가진다.

### 3.7 롤아웃과 롤백

Deployment는 새 버전으로 업데이트할 때 기존 파드를 한 번에 모두 죽이지 않고 조금씩 교체할 수 있다. 이것이 롤링 업데이트다.

문제가 생기면 `rollout undo`로 이전 버전으로 되돌릴 수 있다. 운영 환경에서는 이 흐름을 이해하지 못하면 배포 장애 대응이 어렵다.

## 4. 내가 몰랐을 가능성이 높은 개념

| 개념 | 왜 중요한가 | 이해 포인트 |
|---|---|---|
| Multi-container Pod | 파드의 실무 패턴 | 여러 컨테이너가 하나의 실행 단위를 이룸 |
| localhost 통신 | 파드 내부 통신 | 같은 파드 안 컨테이너끼리는 localhost 사용 가능 |
| Init Container | 시작 전 준비 | 본 컨테이너 실행 전에 선행 작업 수행 |
| Adapter Container | 운영 표준화 | 기존 출력 형식을 관리하기 쉽게 변환 |
| StatefulSet | 상태 있는 앱 관리 | 안정적 이름, 순서, 저장소 보장 |
| VolumeClaimTemplate | 파드별 저장소 요청 | StatefulSet 파드마다 PVC 생성 |
| Rolling Update | 무중단 배포의 기본 | 파드를 조금씩 새 버전으로 교체 |
| Rollback | 장애 대응 | 이전 배포 버전으로 되돌림 |

## 5. 실습하면서 내 것으로 만들 부분

11주차 실습은 한 줄 명령어보다 패턴 이해가 중요하다. 흐름은 아래처럼 나눠서 보는 것이 좋다.

```text
멀티컨테이너 파드
→ 같은 파드 안 컨테이너 확인
→ 공유 볼륨으로 파일 교환
→ localhost로 컨테이너 간 통신

초기화/어댑터 패턴
→ init container로 사전 작업
→ adapter container로 로그나 API 표준화

상태 저장과 배포
→ StatefulSet 배치
→ 파드 이름과 저장소 확인
→ Deployment 롤아웃
→ 문제 발생 시 롤백
```

### 5.1 멀티컨테이너 파드와 공유 볼륨 흐름

```bash
# 예제 코드 디렉터리로 이동
cd ch07

# 두 컨테이너를 가진 파드 정의 적용
kubectl apply -f sleep/sleep-with-file-reader.yaml

# 파드 상세 정보 확인
kubectl get pod -l app=sleep -o wide

# 파드 안의 컨테이너 이름 확인
kubectl get pod -l app=sleep -o jsonpath='{.items[0].status.containerStatuses[*].name}'

# 특정 컨테이너에서 공유 볼륨에 파일 쓰기
kubectl exec deploy/sleep -c sleep -- sh -c 'echo ${HOSTNAME} > /data-rw/hostname.txt'

# 같은 컨테이너에서 파일 확인
kubectl exec deploy/sleep -c sleep -- cat /data-rw/hostname.txt

# 다른 컨테이너에서 같은 파일 읽기
kubectl exec deploy/sleep -c file-reader -- cat /data-ro/hostname.txt
```

`-c` 옵션은 파드 안에 여러 컨테이너가 있을 때 어떤 컨테이너에서 명령을 실행할지 지정한다.

### 5.2 같은 파드 안에서 localhost 통신 확인

```bash
# 서버 컨테이너가 포함된 파드 적용
kubectl apply -f sleep/sleep-with-server.yaml

# 파드 상태 확인
kubectl get pods -l app=sleep

# 파드 안 컨테이너 이름 확인
kubectl get pod -l app=sleep -o jsonpath='{.items[0].status.containerStatuses[*].name}'

# sleep 컨테이너에서 같은 파드의 server 컨테이너로 접근
kubectl exec deploy/sleep -c sleep -- wget -q -O - localhost:8080

# server 컨테이너 로그 확인
kubectl logs -l app=sleep -c server
```

같은 파드 안의 컨테이너는 같은 네트워크 네임스페이스를 공유하므로 `localhost`로 통신할 수 있다.

### 5.3 초기화 컨테이너 흐름

```bash
# 초기화 컨테이너가 포함된 파드 적용
kubectl apply -f sleep/sleep-with-html-server.yaml

# 컨테이너 이름 확인
kubectl get pod -l app=sleep -o jsonpath='{.items[0].status.containerStatuses[*].name}'

# init container 로그 확인
kubectl logs -l app=sleep -c init-html

# server 컨테이너에서 초기화 결과 확인
kubectl exec deploy/sleep -c server -- ls -l /data-ro
```

초기화 컨테이너는 본 컨테이너가 시작하기 전에 필요한 파일이나 설정을 준비한다.

### 5.4 어댑터 컨테이너 흐름

```bash
# timecheck 애플리케이션 배치
kubectl apply -f timecheck/timecheck.yaml

# 로그 확인
kubectl logs -l app=timecheck

# 애플리케이션 로그 파일 확인
kubectl exec deploy/timecheck -- cat /logs/timecheck.log

# 어댑터 컨테이너가 포함된 버전 적용
kubectl apply -f timecheck/timecheck-with-logging.yaml

# 준비 상태 대기
kubectl wait --for=condition=ContainersReady pod -l app=timecheck,version=v3

# logger 컨테이너 로그 확인
kubectl logs -l app=timecheck -c logger
```

어댑터 컨테이너는 애플리케이션을 직접 고치지 않고도 로그 형식이나 운영 인터페이스를 바꿀 수 있게 해준다.

### 5.5 StatefulSet 흐름

```bash
# DB 관련 리소스 적용
kubectl apply -f todo-list/db/

# StatefulSet 확인
kubectl get statefulset todo-db

# StatefulSet이 만든 파드 확인
kubectl get pods -l app=todo-db

# 특정 파드의 hostname 확인
kubectl exec pod/todo-db-0 -- hostname

# 특정 파드 로그 확인
kubectl logs todo-db-1 --tail 1
```

StatefulSet은 파드 이름이 안정적으로 유지된다. `todo-db-0`, `todo-db-1`처럼 순서가 있는 이름이 만들어진다.

### 5.6 롤아웃과 롤백 흐름

```bash
# 디플로이먼트 상태 확인
kubectl get deploy

# 롤아웃 상태 확인
kubectl rollout status deploy/todo-web

# 롤아웃 히스토리 확인
kubectl rollout history deploy/todo-web

# 이전 버전으로 롤백
kubectl rollout undo deploy/todo-web

# 특정 리비전으로 롤백
kubectl rollout undo deploy/todo-web --to-revision=2
```

롤아웃은 배포의 흐름을 보고, 롤백은 문제가 생겼을 때 되돌리는 흐름이다.

### 5.7 명령어별로 정리

| 명령어 | 언제 쓰는가 | 내가 이해한 의미 |
|---|---|---|
| `kubectl exec -c` | 멀티컨테이너 파드에서 특정 컨테이너에 명령 실행 | 파드 안의 대상 컨테이너 지정 |
| `kubectl logs -c` | 특정 컨테이너 로그 확인 | 멀티컨테이너 파드에서 로그 대상 지정 |
| `kubectl get statefulset` | StatefulSet 상태 확인 | 상태 있는 워크로드 관리 상태 확인 |
| `kubectl exec pod/... -- hostname` | 파드 정체성 확인 | StatefulSet 파드의 안정적 이름 확인 |
| `kubectl rollout status` | 배포 진행 상태 확인 | 새 버전 배포가 끝났는지 확인 |
| `kubectl rollout history` | 배포 이력 확인 | 이전 리비전 목록 확인 |
| `kubectl rollout undo` | 배포 롤백 | 문제가 있는 버전을 되돌림 |
| `kubectl wait` | 컨테이너 준비 대기 | 다음 단계 전 준비 상태 확인 |

### 5.8 헷갈렸던 지점

멀티컨테이너 파드는 여러 애플리케이션을 그냥 한 파드에 몰아넣는 구조가 아니다. 같은 생명주기를 가져야 하고, 같은 네트워크나 볼륨을 공유하는 것이 의미 있는 컨테이너를 함께 두는 구조다.

StatefulSet도 모든 데이터베이스에 무조건 쓰는 정답은 아니다. 상태가 필요하고, 안정적인 이름과 저장소가 필요한 경우에 적합하다.

### 5.9 나중에 다시 볼 포인트

- 같은 파드 안 컨테이너가 `localhost`로 통신할 수 있는 이유는 무엇인가?
- 초기화 컨테이너와 일반 컨테이너의 차이는 무엇인가?
- 어댑터 컨테이너는 어떤 문제를 해결하는가?
- StatefulSet은 Deployment와 무엇이 다른가?
- 롤링 업데이트와 롤백 명령어를 실제로 설명할 수 있는가?

## 6. 보충 설명

- 자료 기반 내용: 멀티컨테이너 파드, 초기화 컨테이너, 어댑터 컨테이너, StatefulSet, 롤아웃과 롤백을 다룬다.
- 보충 설명: 멀티컨테이너 파드 패턴은 사이드카, 어댑터, 앰배서더 패턴으로 확장해서 공부하면 좋다.
- 확실하지 않음: 실제 실습 환경에서 StatefulSet 저장소가 어떤 스토리지 클래스로 연결되는지는 클러스터 환경에 따라 달라질 수 있다.

## 7. 한 줄 요약

11주차의 핵심은 파드를 단순 실행 단위가 아니라 여러 컨테이너, 저장소, 배포 전략을 함께 묶는 운영 단위로 이해하는 것이다.

## 8. 추가로 공부할 키워드

- Multi-container Pod
- Init Container
- Adapter Container
- Sidecar Pattern
- StatefulSet
- VolumeClaimTemplate
- Rolling Update
- Rollback
