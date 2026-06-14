---
layout: post
title: "클라우드프로그래밍 14주차 학습 정리"
date: 2026-06-09
categories: [hyu]
tags: [cloud-programming, assignment]
---

# 클라우드프로그래밍 14주차: RBAC, 워크로드 배치, CRD와 오퍼레이터

## 1. 이번 주차에서 다룬 내용

14주차는 Kubernetes 운영의 보안과 확장성을 다룬다. 앞 주차까지는 애플리케이션을 배포하고, 설정하고, 로그와 모니터링을 붙이는 흐름이었다. 이번 주차는 한 단계 더 운영자 관점으로 들어간다.

운영 클러스터에서는 누구나 모든 리소스를 삭제하거나 Secret을 읽을 수 있으면 안 된다. 그래서 RBAC로 권한을 제한해야 한다. 또한 워크로드를 어떤 노드에 배치할지 조정하고, 부하에 따라 자동으로 확장해야 한다. 마지막으로 Kubernetes를 기본 리소스만 쓰는 것이 아니라 CRD와 Operator로 확장하는 방식도 다룬다.

이번 주차에서 다룬 내용은 다음과 같다.

- RBAC 기반 리소스 접근 제어
- 사용자 인증서와 컨텍스트
- Role, ClusterRole, RoleBinding, ClusterRoleBinding
- 서비스 계정 권한 제어
- 그룹과 서비스 계정에 롤 부여
- taint와 toleration을 이용한 스케줄링 제어
- nodeSelector와 node affinity
- 자동 스케일링과 HPA
- 사용자 정의 리소스 CRD
- 사용자 정의 컨트롤러
- Operator를 이용한 서드파티 컴포넌트 관리

## 2. Background

- RBAC: 역할 기반 접근 제어
- Role: 특정 네임스페이스 안에서 권한을 정의하는 리소스
- ClusterRole: 클러스터 전체 범위 또는 여러 네임스페이스에서 사용할 수 있는 권한 정의
- RoleBinding: 특정 주체에게 Role 또는 ClusterRole을 연결하는 리소스
- ServiceAccount: 파드 내부 애플리케이션이 Kubernetes API를 사용할 때 쓰는 계정
- Taint: 노드에 특정 파드가 오지 못하게 막는 조건
- Toleration: 파드가 특정 taint를 견딜 수 있게 하는 설정
- Affinity: 파드를 특정 노드나 파드 근처에 배치하도록 유도하는 규칙
- HPA: Horizontal Pod Autoscaler, 부하에 따라 파드 수를 자동 조정하는 기능
- CRD: Kubernetes API를 확장해 사용자 정의 리소스를 만드는 기능
- Operator: 특정 애플리케이션 운영 지식을 컨트롤러로 자동화한 패턴

### 보충 설명

Kubernetes는 단순 배포 도구가 아니라 API 기반 플랫폼이다. 그래서 권한, 스케줄링, 확장 리소스를 이해하지 못하면 운영 환경에서 위험해진다.

## 3. 개념 정리

### 3.1 RBAC

RBAC는 Role-Based Access Control의 약자로, 사용자가 어떤 리소스에 어떤 동작을 할 수 있는지 제한하는 방식이다.

운영 클러스터에서는 관리자 권한을 가진 사람이 많으면 위험하다. 관리자 계정이 탈취되면 워크로드 삭제, Secret 탈취, 클러스터 자원 남용 같은 문제가 생길 수 있다.

### 3.2 Role과 ClusterRole

Role은 특정 네임스페이스 안에서 권한을 정의한다. ClusterRole은 클러스터 범위 리소스나 여러 네임스페이스에서 사용할 수 있는 권한을 정의한다.

권한은 보통 `get`, `list`, `watch`, `create`, `update`, `delete` 같은 동작과 `pods`, `secrets`, `deployments` 같은 리소스를 조합해서 만든다.

### 3.3 RoleBinding과 ClusterRoleBinding

Role이나 ClusterRole은 권한 정의일 뿐이다. 실제 사용자나 서비스 계정에 권한을 부여하려면 Binding이 필요하다.

- RoleBinding: 특정 네임스페이스에서 주체와 Role/ClusterRole 연결
- ClusterRoleBinding: 클러스터 전체 범위에서 주체와 ClusterRole 연결

이름이 비슷해서 헷갈리지만, 핵심은 “권한 정의”와 “권한 부여”를 구분하는 것이다.

### 3.4 ServiceAccount

ServiceAccount는 파드 내부 애플리케이션이 Kubernetes API를 호출할 때 사용하는 계정이다.

사람이 `kubectl`로 클러스터에 접근하는 것처럼, 클러스터 내부 애플리케이션도 API 서버와 통신할 수 있다. 이때 아무 권한이나 주면 위험하므로 ServiceAccount에도 RBAC를 적용한다.

### 3.5 Taint와 Toleration

Taint는 노드에 붙이는 제한 조건이다. 특정 taint가 붙은 노드에는 그 taint를 toleration으로 허용한 파드만 배치될 수 있다.

이 구조는 특정 노드를 전용 워크로드용으로 남겨두거나, 일반 파드가 특정 노드에 배치되지 않게 막을 때 사용한다.

### 3.6 Node Selector와 Affinity

Node Selector는 가장 단순한 노드 선택 방식이다. 특정 레이블을 가진 노드에 파드를 배치한다.

Affinity는 더 세밀한 배치 규칙을 표현한다. 특정 노드에 반드시 배치해야 하는 조건이나, 특정 파드와 가까이 또는 멀리 배치하고 싶은 조건을 만들 수 있다.

### 3.7 HPA

HPA는 Horizontal Pod Autoscaler다. CPU 사용량 같은 지표를 보고 파드 복제본 수를 자동으로 조정한다.

수동으로 `kubectl scale`을 실행하는 방식은 순간 대응에는 가능하지만, 지속적인 부하 변화에는 자동 스케일링이 필요하다.

### 3.8 CRD와 Operator

CRD는 Kubernetes에 새로운 리소스 타입을 추가하는 기능이다. 예를 들어 `Todo`, `User`, `MySQL` 같은 사용자 정의 리소스를 만들 수 있다.

Operator는 이런 사용자 정의 리소스를 보고 실제 Kubernetes 리소스를 만들거나 수정하는 컨트롤러다. 특정 애플리케이션 운영 지식을 코드로 자동화한 형태라고 볼 수 있다.

## 4. 내가 몰랐을 가능성이 높은 개념

| 개념 | 왜 중요한가 | 이해 포인트 |
|---|---|---|
| RBAC | 클러스터 보안 | 사용자의 API 동작을 제한 |
| Role | 네임스페이스 권한 | 특정 공간 안의 권한 정의 |
| ClusterRole | 클러스터 권한 | 클러스터 범위 권한 정의 |
| RoleBinding | 권한 부여 | Role/ClusterRole을 주체에 연결 |
| ServiceAccount | 내부 애플리케이션 계정 | 파드가 API를 호출할 때 사용 |
| Taint | 노드 접근 제한 | 일반 파드가 특정 노드에 못 오게 함 |
| Toleration | Taint 허용 | 해당 제한을 견딜 수 있는 파드 표시 |
| Node Affinity | 배치 조건 | 파드를 원하는 노드 쪽으로 유도 |
| HPA | 자동 확장 | 지표 기반으로 파드 수 조정 |
| CRD | API 확장 | 사용자 정의 리소스 타입 추가 |
| Operator | 운영 자동화 | 애플리케이션 운영 로직을 컨트롤러로 구현 |

## 5. 실습하면서 내 것으로 만들 부분

14주차 실습은 Kubernetes 운영자가 다루는 기능이 많다. 아래처럼 세 흐름으로 나눠서 보는 것이 맞다.

```text
보안 흐름
→ RBAC API 확인
→ 사용자 인증서 생성
→ 컨텍스트 등록
→ Role/RoleBinding 적용
→ 권한 확인

배치/스케일링 흐름
→ 노드 taint 설정
→ toleration 적용
→ nodeSelector 적용
→ affinity 적용
→ HPA로 자동 스케일링 확인

확장 흐름
→ CRD 적용
→ 사용자 정의 리소스 생성
→ 컨트롤러 로그 확인
→ Operator로 서드파티 컴포넌트 관리
```

### 5.1 RBAC 기능과 기본 권한 확인 흐름

```bash
# 예제 코드 디렉터리 이동
cd ch17

# RBAC API 사용 가능 여부 확인
kubectl api-versions | grep rbac

# 관리자 관련 ClusterRole 확인
kubectl get clusterroles | grep admin

# cluster-admin 권한 상세 확인
kubectl describe clusterrole cluster-admin
```

RBAC를 공부할 때는 먼저 클러스터에 어떤 권한 리소스가 있는지 확인해야 한다.

### 5.2 사용자 인증서와 컨텍스트 흐름

```bash
# 인증서 생성기 배치
kubectl apply -f user-cert-generator.yaml

# 컨테이너 준비 대기
kubectl wait --for=condition=ContainersReady pod user-cert-generator

# 로그 확인
kubectl logs user-cert-generator --tail 3

# 인증서 파일 복사
kubectl cp user-cert-generator:/certs/user.key user.key
kubectl cp user-cert-generator:/certs/user.crt user.crt

# 인증 수단 등록
kubectl config set-credentials reader --client-key=./user.key --client-certificate=./user.crt --embed-certs=true

# 컨텍스트 생성
kubectl config set-context reader --user=reader --cluster $(kubectl config view -o jsonpath='{.clusters[0].name}')
```

컨텍스트는 특정 사용자 인증 정보로 클러스터에 접근하기 위한 설정이다.

### 5.3 RoleBinding으로 권한 부여 흐름

```bash
# sleep 리소스 배치
kubectl apply -f sleep/

# reader 사용자에게 default 네임스페이스 조회 권한 부여
kubectl apply -f role-bindings/reader-view-default.yaml

# reader 권한으로 파드 조회
kubectl get pods --as reader@kiamol.net

# 다른 네임스페이스 조회 시도
kubectl get pods -n kube-system --as reader@kiamol.net

# 삭제 권한 시도
kubectl delete -f sleep/ --as reader@kiamol.net
```

이 흐름은 “권한이 있으면 되고, 없으면 거부된다”는 RBAC의 기본 동작을 확인한다.

### 5.4 ServiceAccount 권한 확인 흐름

```bash
# 네임스페이스 생성
kubectl apply -f namespace.yaml

# 서비스 계정 확인
kubectl get serviceaccounts -n kiamol-ch17

# 현재 사용자 권한 확인
kubectl auth can-i "*" "*"

# 특정 서비스 계정 권한 확인
kubectl auth can-i "*" "*" --as system:serviceaccount:kiamol-ch17:default

# 특정 서비스 계정이 파드를 조회할 수 있는지 확인
kubectl auth can-i get pods -n kiamol-ch17 --as system:serviceaccount:kiamol-ch17:default
```

`kubectl auth can-i`는 RBAC 문제를 디버깅할 때 매우 중요하다. 어떤 주체가 어떤 동작을 할 수 있는지 바로 확인할 수 있다.

### 5.5 Taint와 Toleration 흐름

```bash
# 노드 taint 확인
cd ch19
kubectl get nodes -o jsonpath='{range.items[*]}{.metadata.name} {.spec.taints[*].key}{end}'

# sleep 파드 배치
kubectl apply -f sleep/sleep.yaml

# 모든 노드에 taint 부여
kubectl taint nodes --all kiamol-disk=hdd:NoSchedule

# 파드 상태 확인
kubectl get pods -l app=sleep

# toleration이 포함된 파드 적용
kubectl apply -f sleep/update/sleep2-with-tolerations.yaml

# 파드 상태 확인
kubectl get po -l app=sleep2
```

Taint는 노드가 파드를 거부하는 조건이고, toleration은 파드가 그 조건을 견딜 수 있다는 표시다.

### 5.6 nodeSelector와 affinity 흐름

```bash
# 노드 레이블 확인
kubectl get nodes --show-labels

# nodeSelector가 있는 파드 적용
kubectl apply -f sleep/update/sleep2-with-nodeSelector.yaml

# 파드 배치 확인
kubectl get pods -l app=sleep2

# node affinity가 있는 파드 적용
kubectl apply -f sleep/update/sleep2-with-nodeAffinity-required.yaml

# 파드 상태 확인
kubectl get po -l app=sleep2
```

nodeSelector는 단순 조건이고, affinity는 더 표현력이 높은 배치 규칙이다.

### 5.7 HPA 자동 스케일링 흐름

```bash
# pi 애플리케이션 배치
kubectl apply -f pi/

# 준비 상태 대기
kubectl wait --for=condition=ContainersReady pod -l app=pi-web

# HPA 확인
kubectl get hpa pi-cpu

# 파드 리소스 사용량 확인
kubectl top pods -l app=pi-web

# HPA 설정 업데이트
kubectl apply -f pi/update/hpa-cpu-v2.yaml

# HPA 상태 다시 확인
kubectl get hpa pi-cpu

# Deployment 복제본 수 확인
kubectl get deploy pi-web
```

HPA는 CPU 같은 지표를 보고 복제본 수를 조정한다. 단, metrics-server 같은 지표 수집 구성이 필요하다.

### 5.8 CRD와 사용자 정의 리소스 흐름

```bash
# CRD 적용
cd ch20
kubectl apply -f todo-custom/

# CRD 확인
kubectl get crd -l kiamol=ch20

# 사용자 정의 리소스 적용
kubectl apply -f todo-custom/items/

# 사용자 정의 리소스 조회
kubectl get todos

# CRD 업데이트
kubectl apply -f todo-custom/update/

# 사용자 정의 리소스 상세 확인
kubectl describe todo ch20
```

CRD를 만들면 Kubernetes API에 새로운 리소스 타입이 생긴다. 그래서 `kubectl get todos` 같은 명령이 가능해진다.

### 5.9 Operator 흐름

```bash
# 메시지 큐 Operator 리소스 적용
kubectl apply -f todo-list/msgq/

# NATS 리소스 확인
kubectl get nats

# 관련 파드 확인
kubectl get pods -l app=nats

# MySQL 관련 리소스 적용
kubectl apply -f todo-list/db/

# MySQL 사용자 정의 리소스 확인
kubectl get mysql

# StatefulSet 확인
kubectl get statefulset todo-db-mysql -o wide

# 애플리케이션 리소스 배치
kubectl apply -f todo-list/config/
kubectl apply -f todo-list/save-handler/ -f todo-list/web/

# 준비 상태 대기
kubectl wait --for=condition=ContainersReady pod -l app=todo-list

# 로그 확인
kubectl logs -l app=todo-list,component=save-handler
```

Operator는 사용자 정의 리소스를 보고 실제 필요한 리소스를 만들어준다. 단순 CRD는 데이터 타입을 추가하는 것이고, Operator는 그 리소스를 실제로 운영하는 컨트롤러까지 포함한다.

### 5.10 명령어별로 정리

| 명령어 | 언제 쓰는가 | 내가 이해한 의미 |
|---|---|---|
| `kubectl api-versions` | API 그룹 확인 | RBAC나 CRD 지원 여부 확인 |
| `kubectl describe clusterrole` | 권한 상세 확인 | 어떤 리소스와 동작이 허용되는지 확인 |
| `kubectl config set-credentials` | 인증 정보 등록 | kubectl에 사용자 인증 수단 추가 |
| `kubectl config set-context` | 컨텍스트 생성 | 특정 사용자/클러스터 조합으로 접근 설정 |
| `kubectl auth can-i` | 권한 확인 | 특정 주체가 동작 가능한지 검사 |
| `kubectl taint nodes` | 노드에 taint 부여 | 파드 스케줄링 제한 |
| `kubectl get nodes --show-labels` | 노드 레이블 확인 | nodeSelector/affinity 조건 확인 |
| `kubectl top pods` | 파드 리소스 사용량 확인 | HPA 판단 지표 확인 |
| `kubectl get hpa` | 자동 스케일링 상태 확인 | 목표 지표와 현재 복제본 확인 |
| `kubectl get crd` | 사용자 정의 리소스 정의 확인 | Kubernetes API 확장 상태 확인 |
| `kubectl get todos` | 사용자 정의 리소스 조회 | CRD로 추가된 리소스 사용 |
| `helm uninstall` | Helm 릴리스 제거 | Operator 같은 Helm 설치 구성 정리 |

### 5.11 헷갈렸던 지점

Role과 RoleBinding은 다르다. Role은 권한 정의이고, RoleBinding은 그 권한을 사용자나 서비스 계정에 연결하는 것이다.

Taint와 node affinity도 다르다. Taint는 노드가 파드를 밀어내는 조건이고, affinity는 파드가 특정 노드를 선호하거나 요구하는 조건이다.

CRD와 Operator도 구분해야 한다. CRD는 새로운 리소스 타입을 추가하는 것이고, Operator는 그 리소스를 보고 실제 운영 작업까지 수행하는 컨트롤러다.

### 5.12 나중에 다시 볼 포인트

- Role, ClusterRole, RoleBinding, ClusterRoleBinding을 정확히 구분할 수 있는가?
- ServiceAccount에 권한을 주는 이유는 무엇인가?
- `kubectl auth can-i`를 이용해 RBAC 문제를 확인할 수 있는가?
- taint/toleration과 affinity의 차이를 설명할 수 있는가?
- HPA가 동작하기 위해 필요한 조건은 무엇인가?
- CRD와 Operator의 차이는 무엇인가?

## 6. 보충 설명

- 자료 기반 내용: RBAC, 서비스 계정, taint/toleration, nodeSelector, affinity, HPA, CRD, Operator를 다룬다.
- 보충 설명: 운영 환경에서는 RBAC를 최소 권한 원칙으로 설계해야 하며, ServiceAccount 토큰과 Secret 접근 권한을 특히 조심해야 한다.
- 확실하지 않음: HPA 실습 결과는 metrics-server 설치 여부와 클러스터 환경에 따라 달라질 수 있다.

## 7. 한 줄 요약

14주차의 핵심은 Kubernetes를 운영 플랫폼으로 다루기 위해 권한, 배치, 자동 확장, API 확장 구조를 이해하는 것이다.

## 8. 추가로 공부할 키워드

- RBAC
- Role
- ClusterRole
- RoleBinding
- ServiceAccount
- Taint
- Toleration
- Node Affinity
- HPA
- CRD
- Operator
