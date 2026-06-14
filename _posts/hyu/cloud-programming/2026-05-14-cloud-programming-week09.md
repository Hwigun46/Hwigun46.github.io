---
layout: post
title: "클라우드프로그래밍 9주차 학습 정리"
date: 2026-05-14
categories: [hyu]
tags: [cloud-programming, assignment]
---

# 클라우드프로그래밍 9주차: Kubernetes 파드, 디플로이먼트, 서비스

## 1. 이번 주차에서 다룬 내용

9주차부터는 Kubernetes를 실제로 사용하는 흐름으로 들어간다. 7주차에서 Kubernetes 클러스터의 구조를 훑어봤다면, 이번 주차는 그 클러스터 위에 컨테이너를 어떻게 올리고, 어떻게 유지하고, 어떻게 네트워크로 연결하는지 보는 주차였다.

Docker에서는 컨테이너를 직접 실행했다. 하지만 Kubernetes에서는 보통 컨테이너를 직접 다루지 않는다. 컨테이너는 파드 안에서 실행되고, 파드는 디플로이먼트 같은 컨트롤러가 관리한다. 그리고 파드가 바뀌어도 안정적으로 접근할 수 있도록 서비스가 앞단에 놓인다.

이번 주차에서 다룬 내용은 다음과 같다.

- 파드가 컨테이너를 실행하는 방식
- `kubectl run`으로 파드를 직접 실행하는 흐름
- `kubectl get`, `describe`, `jsonpath`, `custom-columns` 출력 방식
- 파드 장애가 발생했을 때 Kubernetes가 컨테이너를 다시 만드는 방식
- 디플로이먼트와 컨트롤러 객체
- 레이블과 레이블 셀렉터
- YAML 매니페스트로 파드와 디플로이먼트를 정의하는 방식
- Kubernetes 리소스 삭제와 재생성 흐름
- 서비스로 파드에 안정적으로 접근하는 방식
- 내부 파드 통신과 외부 트래픽 전달

## 2. Background

- 컨테이너: 애플리케이션 실행 환경을 격리한 단위
- 파드: Kubernetes에서 컨테이너를 실행하는 최소 단위
- 디플로이먼트: 파드를 원하는 개수와 상태로 유지하는 컨트롤러
- 레이블: 리소스를 식별하기 위해 붙이는 key-value 정보
- 셀렉터: 특정 레이블을 가진 리소스를 찾는 조건
- 서비스: 파드 집합에 안정적인 네트워크 접근 지점을 제공하는 리소스
- 매니페스트: Kubernetes 리소스를 YAML로 정의한 파일

### 보충 설명

Kubernetes를 처음 볼 때는 `Pod`, `Deployment`, `Service`가 전부 비슷한 리소스처럼 느껴진다. 하지만 역할이 다르다.

- 파드는 컨테이너를 담는 실행 단위다.
- 디플로이먼트는 파드를 관리하는 관리자다.
- 서비스는 바뀌는 파드 앞에서 고정된 접근 지점을 제공한다.

내가 이해한 방식으로는 파드는 실제 매장 직원, 디플로이먼트는 직원 수와 상태를 관리하는 매니저, 서비스는 고객이 항상 찾아오는 안내 데스크에 가깝다.

## 3. 개념 정리

### 3.1 파드

파드는 Kubernetes에서 컨테이너를 실행하는 최소 단위다. 컨테이너는 파드 안에 포함되어 동작하고, 사용자는 파드를 통해 컨테이너를 관리한다.

파드 하나에는 컨테이너 하나만 들어갈 수도 있고, 여러 컨테이너가 함께 들어갈 수도 있다. 9주차에서는 우선 컨테이너 하나를 담은 단순한 파드를 실행하는 것부터 시작한다.

중요한 점은 Kubernetes가 컨테이너를 직접 노출하지 않고, 파드라는 실행 단위로 감싼다는 것이다. 이 구조 때문에 파드는 IP를 가지며, 파드 안의 컨테이너는 같은 실행 맥락을 공유할 수 있다.

### 3.2 `kubectl`

`kubectl`은 Kubernetes 클러스터와 대화하는 명령어다. Docker에서 `docker` 명령어로 컨테이너를 다뤘다면, Kubernetes에서는 `kubectl`로 파드, 디플로이먼트, 서비스, 네임스페이스 같은 리소스를 다룬다.

처음에는 `kubectl get pods` 정도만 외우게 되지만, 실제로는 `describe`, `logs`, `exec`, `apply`, `delete`, `port-forward`까지 이어져야 Kubernetes를 손으로 다룰 수 있다.

### 3.3 디플로이먼트

디플로이먼트는 파드를 직접 하나만 실행하는 방식보다 더 실무적인 방식이다. 파드가 삭제되거나 장애가 나면 디플로이먼트가 다시 파드를 만들어 원하는 상태를 맞춘다.

직접 파드를 만들면 그 파드가 죽었을 때 끝이다. 하지만 디플로이먼트로 만들면 Kubernetes가 “이 애플리케이션은 계속 실행되어야 한다”는 의도를 기억한다.

내가 이해한 방식으로 정리하면, 파드는 현재 실행 중인 결과이고 디플로이먼트는 유지해야 할 선언이다.

### 3.4 레이블과 셀렉터

Kubernetes는 많은 리소스를 다루기 때문에 이름만으로 관리하기 어렵다. 그래서 리소스에 레이블을 붙이고, 셀렉터로 특정 레이블을 가진 리소스를 찾는다.

예를 들어 `app=hello-kiamol-2`라는 레이블이 붙은 파드를 찾거나, 서비스가 특정 레이블을 가진 파드로 트래픽을 보내도록 할 수 있다.

레이블을 잘못 바꾸면 디플로이먼트가 원래 파드를 잃어버린 것처럼 보고 새 파드를 만들 수 있다. 이 부분이 처음에는 꽤 헷갈린다.

### 3.5 매니페스트

Kubernetes 리소스는 명령어로 직접 만들 수도 있지만, 보통은 YAML 매니페스트 파일로 정의한다.

명령어로 만든 리소스는 빠르게 테스트하기 좋지만, 반복 재현에는 약하다. 반대로 매니페스트는 리소스의 원하는 상태를 파일로 남기기 때문에 Git으로 관리하기 좋다.

### 3.6 서비스

서비스는 파드에 안정적으로 접근하기 위한 네트워크 리소스다.

파드는 삭제되고 다시 만들어질 수 있고, 그때 IP도 바뀔 수 있다. 따라서 다른 파드나 사용자가 파드 IP를 직접 바라보면 문제가 생긴다. 서비스는 특정 레이블을 가진 파드 집합 앞에 고정된 이름과 접근 지점을 제공한다.

## 4. 내가 몰랐을 가능성이 높은 개념

| 개념 | 왜 중요한가 | 이해 포인트 |
|---|---|---|
| Pod | Kubernetes의 기본 실행 단위 | 컨테이너를 직접 실행하지 않고 파드로 감싼다 |
| Deployment | 파드 상태 유지 | 파드가 죽어도 원하는 개수만큼 다시 만든다 |
| Label | 리소스 식별 | 이름보다 유연하게 리소스를 묶는다 |
| Selector | 리소스 선택 조건 | 서비스와 디플로이먼트가 파드를 찾는 기준 |
| Manifest | 선언형 관리 | 리소스 상태를 YAML로 기록한다 |
| Service | 안정적인 접근 지점 | 바뀌는 파드 IP 앞에 고정된 네트워크 이름을 둔다 |
| Port Forward | 임시 접근 | 클러스터 내부 파드를 로컬 포트로 연결한다 |
| JSONPath | 원하는 필드 출력 | 복잡한 리소스 정보에서 필요한 값만 뽑는다 |

## 5. 실습하면서 내 것으로 만들 부분

9주차 실습은 Kubernetes 명령어의 기본 감각을 만드는 구간이다. 여기서 중요한 것은 명령어를 한 줄씩 외우는 것이 아니라, “리소스를 만들고, 확인하고, 접근하고, 지우는 흐름”을 손에 익히는 것이다.

```text
파드 실행
→ 준비 상태 대기
→ 상태 확인
→ 상세 정보 확인
→ 필요한 정보만 뽑기
→ 포트 포워딩으로 접근
→ 디플로이먼트로 다시 실행
→ 서비스로 안정적인 접근 지점 만들기
```

### 5.1 파드를 직접 실행하고 확인하는 흐름

```bash
# 컨테이너 하나를 담은 파드를 실행한다.
kubectl run hello-kiamol --image=kiamol/ch02-hello-kiamol

# 파드가 Ready 상태가 될 때까지 기다린다.
kubectl wait --for=condition=Ready pod hello-kiamol

# 클러스터의 파드 목록을 확인한다.
kubectl get pods

# 특정 파드의 상세 정보를 확인한다.
kubectl describe pod hello-kiamol
```

이 흐름은 Kubernetes의 가장 기본 조작이다. `run`으로 만들고, `wait`로 준비 상태를 기다리고, `get`으로 목록을 보고, `describe`로 자세히 본다.

### 5.2 필요한 정보만 뽑아보는 흐름

```bash
# 파드의 기본 정보 확인
kubectl get pod hello-kiamol

# 원하는 컬럼만 지정해 출력
kubectl get pod hello-kiamol --output custom-columns=NAME:metadata.name,NODE_IP:status.hostIP,POD_IP:status.podIP

# JSONPath로 컨테이너 ID만 출력
kubectl get pod hello-kiamol -o jsonpath='{.status.containerStatuses[0].containerID}'
```

`kubectl get`은 단순 목록 확인 명령어로 끝나지 않는다. `custom-columns`와 `jsonpath`를 쓰면 리소스 내부의 필요한 필드만 뽑을 수 있다. 나중에 자동화 스크립트를 작성할 때 중요하다.

### 5.3 포트 포워딩으로 파드에 접근하는 흐름

```bash
# 로컬 8080 포트를 파드의 80 포트로 연결한다.
kubectl port-forward pod/hello-kiamol 8080:80
```

`port-forward`는 서비스를 정식으로 외부에 공개하기 전에 임시로 파드에 접근할 때 사용한다. 로컬에서 테스트할 때는 편하지만, 운영 환경의 외부 공개 방식으로 이해하면 안 된다.

### 5.4 디플로이먼트로 파드를 실행하는 흐름

```bash
# 디플로이먼트 생성
kubectl create deployment hello-kiamol-2 --image=kiamol/ch02-hello-kiamol

# 생성된 파드 확인
kubectl get pods

# 디플로이먼트의 레이블 확인
kubectl get deploy hello-kiamol-2 -o jsonpath='{.spec.template.metadata.labels}'

# 레이블로 파드 조회
kubectl get pods -l app=hello-kiamol-2
```

디플로이먼트는 파드를 직접 실행하는 것보다 중요한 방식이다. 파드 하나를 띄우는 것이 아니라, “이 이미지로 된 파드를 유지하라”는 선언을 클러스터에 남긴다.

### 5.5 레이블을 바꿔보는 흐름

```bash
# 파드의 레이블 목록 확인
kubectl get pods -o custom-columns=NAME:metadata.name,LABELS:metadata.labels

# 레이블 변경
kubectl label pods -l app=hello-kiamol-2 --overwrite app=hello-kiamol-x

# 다시 레이블 확인
kubectl get pods -o custom-columns=NAME:metadata.name,LABELS:metadata.labels
```

레이블을 바꾸면 디플로이먼트가 관리하던 파드를 더 이상 자기 파드로 보지 못할 수 있다. 그러면 새 파드를 만들기도 한다. 이 실습은 레이블이 단순 메모가 아니라 컨트롤러의 선택 기준이라는 점을 보여준다.

### 5.6 YAML 매니페스트로 배포하는 흐름

```bash
# 파드 매니페스트 적용
kubectl apply -f pod.yaml

# 디플로이먼트 매니페스트 적용
kubectl apply -f deployment.yaml

# 레이블로 파드 확인
kubectl get pods -l app=hello-kiamol-4
```

`apply`는 파일에 정의된 상태를 클러스터에 반영하는 명령어다. Git에서 파일을 기준으로 작업을 남기듯이, Kubernetes에서는 YAML 파일이 리소스 관리의 기준이 된다.

### 5.7 서비스로 파드에 접근하는 흐름

```bash
# sleep 파드 배치
kubectl apply -f sleep/sleep1.yaml -f sleep/sleep2.yaml

# 특정 파드의 IP 확인
kubectl get pod -l app=sleep-2 --output jsonpath='{.items[0].status.podIP}'

# 다른 파드에서 해당 파드로 ping
kubectl exec deploy/sleep-1 -- ping -c 2 $(kubectl get pod -l app=sleep-2 --output jsonpath='{.items[0].status.podIP}')

# 서비스 생성
kubectl apply -f sleep/sleep2-service.yaml

# 서비스 확인
kubectl get svc sleep-2

# 서비스 이름으로 접근
kubectl exec deploy/sleep-1 -- ping -c 1 sleep-2
```

파드 IP는 바뀔 수 있으므로 직접 의존하면 위험하다. 서비스 이름으로 접근하면 뒤의 파드가 바뀌어도 접근 지점은 유지된다.

### 5.8 명령어별로 정리

| 명령어 | 언제 쓰는가 | 내가 이해한 의미 |
|---|---|---|
| `kubectl run` | 간단한 파드를 바로 실행할 때 | 테스트용 파드를 빠르게 만드는 명령어 |
| `kubectl wait` | 파드가 준비될 때까지 기다릴 때 | 다음 작업 전에 Ready 상태를 확인 |
| `kubectl get pods` | 파드 목록을 볼 때 | 현재 실행 상태를 빠르게 확인 |
| `kubectl describe pod` | 파드 상세 상태를 볼 때 | 이벤트, 노드, 컨테이너 상태까지 확인 |
| `kubectl create deployment` | 디플로이먼트를 빠르게 만들 때 | 파드를 관리하는 컨트롤러 생성 |
| `kubectl apply -f` | YAML 정의를 적용할 때 | 파일에 적은 원하는 상태를 클러스터에 반영 |
| `kubectl label` | 리소스 레이블을 붙이거나 수정할 때 | 컨트롤러와 서비스의 선택 기준 조정 |
| `kubectl port-forward` | 로컬에서 파드나 서비스에 임시 접근할 때 | 클러스터 내부 포트를 내 컴퓨터 포트로 연결 |
| `kubectl exec` | 파드 내부에서 명령어를 실행할 때 | 컨테이너 안으로 들어가 확인 작업 수행 |
| `kubectl delete` | 리소스를 삭제할 때 | 직접 만든 리소스를 정리 |

### 5.9 헷갈렸던 지점

처음에는 파드를 삭제하면 애플리케이션이 사라질 것처럼 생각하기 쉽다. 하지만 디플로이먼트가 만든 파드는 삭제해도 다시 생긴다. 이유는 디플로이먼트가 “이 파드가 있어야 한다”는 원하는 상태를 유지하기 때문이다.

따라서 Kubernetes에서 지울 때는 내가 지금 파드를 지우는지, 파드를 만든 컨트롤러를 지우는지 구분해야 한다.

### 5.10 나중에 다시 볼 포인트

- 파드와 디플로이먼트의 차이를 설명할 수 있는가?
- 레이블과 셀렉터가 서비스와 디플로이먼트에서 어떻게 쓰이는가?
- 파드 IP를 직접 쓰는 것이 왜 위험한가?
- `kubectl get`, `describe`, `logs`, `exec`를 언제 구분해서 쓰는가?
- `kubectl apply -f` 방식이 왜 Git 기반 관리와 잘 맞는가?

## 6. 보충 설명

- 자료 기반 내용: 파드 실행, 디플로이먼트 생성, 매니페스트 적용, 서비스 기반 통신, 포트 포워딩 흐름을 다룬다.
- 보충 설명: 실무에서는 파드를 직접 생성하기보다 디플로이먼트, 잡, 스테이트풀셋 같은 컨트롤러를 통해 관리하는 경우가 많다.
- 확실하지 않음: 실제 실습 환경에서 LoadBalancer 서비스가 바로 외부 IP를 받는지는 클러스터 구성에 따라 다르다.

## 7. 한 줄 요약

9주차의 핵심은 Kubernetes에서 컨테이너를 직접 관리하는 것이 아니라, 파드와 디플로이먼트와 서비스를 통해 원하는 실행 상태와 접근 지점을 관리한다는 점이다.

## 8. 추가로 공부할 키워드

- Pod
- Deployment
- ReplicaSet
- Service
- Label Selector
- Manifest
- Port Forward
- JSONPath
