---
layout: post
title: "클라우드프로그래밍 10주차 학습 정리"
date: 2026-05-21
categories: [hyu]
tags: [cloud-programming, assignment]
---

# 클라우드프로그래밍 10주차: ConfigMap, Secret, 볼륨, 스케일링

## 1. 이번 주차에서 다룬 내용

10주차는 Kubernetes 애플리케이션을 실제 서비스답게 다루기 위한 중급 내용이다. 단순히 파드를 띄우는 수준을 넘어서, 설정값을 어떻게 주입할지, 데이터는 어디에 저장할지, 파드 수를 어떻게 늘리고 줄일지를 다룬다.

처음 Kubernetes를 배우면 “컨테이너를 실행한다”에만 집중하게 된다. 하지만 실제 애플리케이션은 설정이 필요하고, 데이터가 남아야 하고, 트래픽이 늘면 복제본을 늘릴 수 있어야 한다. 이번 주차는 그 세 가지 문제를 한 번에 묶어 다룬다.

이번 주차에서 다룬 내용은 다음과 같다.

- ConfigMap과 Secret을 이용한 설정 주입
- 환경 변수로 설정값 전달
- ConfigMap을 파일처럼 마운트하는 방식
- 컨테이너 파일 시스템과 데이터 소멸 문제
- `emptyDir`, `hostPath`, `subPath`
- PersistentVolume과 PersistentVolumeClaim
- ReplicaSet과 Deployment 기반 스케일링
- `kubectl scale`을 이용한 수동 스케일링
- DaemonSet을 이용한 노드별 파드 배치

## 2. Background

- 환경 변수: 실행 시점에 애플리케이션 설정값을 전달하는 방식
- ConfigMap: 민감하지 않은 설정 데이터를 저장하는 Kubernetes 리소스
- Secret: 비밀번호, 토큰 같은 민감 데이터를 저장하는 Kubernetes 리소스
- Volume: 컨테이너 파일 시스템 밖에 데이터를 두기 위한 저장 공간
- PersistentVolume: 클러스터에서 제공하는 실제 저장소 리소스
- PersistentVolumeClaim: 애플리케이션이 저장 공간을 요청하는 리소스
- ReplicaSet: 지정한 개수의 파드를 유지하는 컨트롤러
- DaemonSet: 각 노드마다 파드를 실행하는 컨트롤러

### 보충 설명

컨테이너는 원래 쉽게 만들고 쉽게 버리는 실행 단위다. 그래서 컨테이너 안에 설정과 데이터를 직접 박아 넣으면 운영이 어려워진다.

ConfigMap과 Secret은 설정을 이미지 밖으로 빼는 방식이고, Volume과 PVC는 데이터를 컨테이너 밖으로 빼는 방식이다.

## 3. 개념 정리

### 3.1 ConfigMap과 Secret

Kubernetes에서 컨테이너에 설정값을 주입할 때 주로 쓰는 리소스는 ConfigMap과 Secret이다.

ConfigMap은 일반 설정값을 저장하고, Secret은 비밀번호나 토큰처럼 민감한 값을 저장한다. 둘 다 스스로 애플리케이션을 실행하지는 않는다. 단지 데이터를 저장하고, 파드가 그 데이터를 환경 변수나 파일 형태로 읽을 수 있게 한다.

내가 이해한 방식으로는 ConfigMap과 Secret은 애플리케이션 설정을 담는 외부 설정 파일함에 가깝다. 이미지를 다시 만들지 않고도 실행 설정을 바꿀 수 있게 해준다.

### 3.2 환경 변수 주입

가장 단순한 설정 주입 방식은 파드 정의에 환경 변수를 직접 넣는 것이다.

```yaml
spec:
  containers:
  - name: sleep
    image: kiamol/ch03-sleep
    env:
    - name: KIAMOL_CHAPTER
      value: "04"
```

이 방식은 간단하지만 설정값이 많아지거나 환경별로 달라지면 YAML이 지저분해진다. 그래서 ConfigMap을 별도 리소스로 만들고, 파드에서 참조하는 방식이 더 자연스럽다.

### 3.3 ConfigMap 파일 마운트

ConfigMap은 환경 변수뿐 아니라 파일처럼 컨테이너에 마운트할 수 있다. 애플리케이션이 `appsettings.json` 같은 설정 파일을 읽는 구조라면 이 방식이 잘 맞는다.

다만 ConfigMap을 마운트한 파일은 일반 파일처럼 보이지만, 실제로는 Kubernetes가 관리하는 설정 데이터다. 컨테이너 안에서 수정해도 원하는 방식으로 영구 변경된다고 생각하면 안 된다.

### 3.4 컨테이너 파일 시스템과 `emptyDir`

컨테이너 내부 파일 시스템은 컨테이너 생명주기에 영향을 받는다. 컨테이너가 다시 만들어지면 내부에 기록한 파일이 사라질 수 있다.

`emptyDir`은 파드 생명주기 동안 유지되는 임시 볼륨이다. 같은 파드 안에서 컨테이너가 재시작되어도 유지될 수 있지만, 파드가 삭제되면 함께 사라진다.

### 3.5 `hostPath`와 `subPath`

`hostPath`는 노드의 특정 경로를 파드에 마운트하는 방식이다. 노드의 파일 시스템을 직접 연결하기 때문에 강력하지만 위험하다.

`subPath`는 볼륨 전체가 아니라 특정 하위 경로만 마운트할 때 사용한다. 필요한 경로만 제한적으로 연결할 수 있다는 점에서 실습에서 자주 등장한다.

### 3.6 PersistentVolume과 PersistentVolumeClaim

PersistentVolume은 클러스터가 제공하는 저장소이고, PersistentVolumeClaim은 애플리케이션이 저장소를 요청하는 문서다.

이 구조가 중요한 이유는 애플리케이션이 저장소의 실제 구현을 직접 몰라도 되기 때문이다. 파드는 “이 정도 크기의 저장소가 필요하다”고 PVC로 요청하고, Kubernetes는 조건에 맞는 PV와 연결한다.

### 3.7 ReplicaSet, Deployment, DaemonSet

ReplicaSet은 지정한 수의 파드를 유지한다. Deployment는 ReplicaSet을 관리하면서 롤아웃까지 다룬다. DaemonSet은 각 노드마다 하나씩 파드를 실행할 때 사용한다.

| 컨트롤러 | 주로 쓰는 상황 |
|---|---|
| ReplicaSet | 동일한 파드 개수 유지 |
| Deployment | 일반 애플리케이션 배포와 스케일링 |
| DaemonSet | 로그 수집기, 프록시처럼 노드마다 필요한 파드 실행 |

## 4. 내가 몰랐을 가능성이 높은 개념

| 개념 | 왜 중요한가 | 이해 포인트 |
|---|---|---|
| ConfigMap | 설정 분리 | 이미지 재빌드 없이 설정 변경 |
| Secret | 민감 정보 분리 | 비밀번호, 토큰을 이미지에 넣지 않음 |
| `env` | 환경 변수 주입 | 컨테이너 실행 시점 설정 전달 |
| `emptyDir` | 파드 생명주기 임시 저장 | 파드가 사라지면 데이터도 사라짐 |
| `hostPath` | 노드 파일 시스템 접근 | 강력하지만 보안상 조심해야 함 |
| PV | 실제 저장소 | 클러스터가 제공하는 저장 공간 |
| PVC | 저장소 요청 | 파드가 필요한 저장소를 요청 |
| ReplicaSet | 파드 개수 유지 | 원하는 복제본 수를 맞춤 |
| DaemonSet | 노드별 실행 | 모든 노드에 하나씩 파드 배치 |

## 5. 실습하면서 내 것으로 만들 부분

10주차 실습은 흐름이 많다. 크게 세 가지로 나눠서 보는 것이 좋다.

```text
설정 주입 흐름
→ 환경 변수 확인
→ ConfigMap 생성
→ ConfigMap을 환경 변수로 주입
→ ConfigMap을 파일로 마운트

데이터 유지 흐름
→ 컨테이너 내부 파일 생성
→ 재시작 후 사라지는지 확인
→ emptyDir 확인
→ hostPath 확인
→ PV/PVC로 영구 저장 확인

스케일링 흐름
→ ReplicaSet 확인
→ Deployment 복제본 수 변경
→ Service 부하분산 확인
→ DaemonSet으로 노드별 배치 확인
```

### 5.1 환경 변수와 ConfigMap 흐름

```bash
# 예제 코드 디렉터리로 이동
cd ch04

# 설정값 없이 sleep 파드 배치
kubectl apply -f sleep/sleep.yaml

# 파드가 준비될 때까지 대기
kubectl wait --for=condition=Ready pod -l app=sleep

# 컨테이너 환경 변수 확인
kubectl exec deploy/sleep -- printenv HOSTNAME KIAMOL_CHAPTER

# 환경 변수가 추가된 정의 적용
kubectl apply -f sleep/sleep-with-env.yaml

# 다시 환경 변수 확인
kubectl exec deploy/sleep -- printenv HOSTNAME KIAMOL_CHAPTER
```

이 흐름은 컨테이너 내부 환경 변수가 어떻게 바뀌는지 보는 실습이다. `apply`를 통해 디플로이먼트 정의를 바꾸면 새 파드가 만들어지고, 그 파드에 새로운 환경 변수가 들어간다.

### 5.2 ConfigMap을 만들고 참조하는 흐름

```bash
# literal 값으로 ConfigMap 생성
kubectl create configmap sleep-config-literal --from-literal=kiamol.section='4.1'

# ConfigMap 목록 확인
kubectl get cm sleep-config-literal

# ConfigMap 상세 확인
kubectl describe cm sleep-config-literal

# ConfigMap 값을 환경 변수로 읽는 파드 정의 적용
kubectl apply -f sleep/sleep-with-configMap-env.yaml

# KIAMOL로 시작하는 환경 변수 확인
kubectl exec deploy/sleep -- sh -c 'printenv | grep "^KIAMOL"'
```

`kubectl create configmap`은 설정 데이터를 별도 리소스로 만드는 명령어다. 이후 파드 정의에서 `configMapKeyRef`로 특정 키를 읽어온다.

### 5.3 ConfigMap을 파일로 사용하는 흐름

```bash
# env 파일로 ConfigMap 생성
kubectl create configmap sleep-config-env-file --from-env-file=sleep/ch04.env

# ConfigMap 확인
kubectl get cm sleep-config-env-file

# ConfigMap을 사용하는 파드 적용
kubectl apply -f sleep/sleep-with-configMap-env-file.yaml

# 환경 변수 확인
kubectl exec deploy/sleep -- sh -c 'printenv | grep "^KIAMOL"'
```

파일 기반 ConfigMap은 설정이 많을 때 유용하다. 환경 변수 하나씩 넣는 것보다 파일 하나를 ConfigMap으로 만드는 방식이 관리하기 쉽다.

### 5.4 컨테이너 파일 시스템과 `emptyDir` 확인 흐름

```bash
# sleep 파드 배치
kubectl apply -f sleep/sleep.yaml

# 컨테이너 내부에 파일 생성
kubectl exec deploy/sleep -- sh -c 'echo ch05 > /file.txt; ls /*.txt'

# 현재 컨테이너 ID 확인
kubectl get pod -l app=sleep -o jsonpath='{.items[0].status.containerStatuses[0].containerID}'

# 컨테이너 프로세스 종료
kubectl exec -it deploy/sleep -- killall5

# 새 컨테이너 ID 확인
kubectl get pod -l app=sleep -o jsonpath='{.items[0].status.containerStatuses[0].containerID}'

# 파일이 남아 있는지 확인
kubectl exec deploy/sleep -- ls /*.txt
```

컨테이너 내부 파일에 의존하면 위험하다는 점을 확인하는 흐름이다.

```bash
# emptyDir 볼륨이 있는 파드 적용
kubectl apply -f sleep/sleep-with-emptyDir.yaml

# 볼륨 경로 확인
kubectl exec deploy/sleep -- ls /data

# 볼륨에 파일 생성
kubectl exec deploy/sleep -- sh -c 'echo ch05 > /data/file.txt; ls /data'

# 컨테이너 재시작 후 파일 확인
kubectl exec deploy/sleep -- cat /data/file.txt
```

`emptyDir`은 파드 안에서 컨테이너 재시작 정도는 견딜 수 있지만, 파드 자체가 사라지면 데이터가 사라진다.

### 5.5 PV와 PVC 흐름

```bash
# 노드에 레이블 부여
kubectl label node $(kubectl get nodes -o jsonpath='{.items[0].metadata.name}') kiamol=ch05

# PV 생성
kubectl apply -f todo-list/persistentVolume.yaml

# PV 확인
kubectl get pv

# PVC 생성
kubectl apply -f todo-list/postgres-persistentVolumeClaim.yaml

# PVC와 PV 연결 상태 확인
kubectl get pvc
kubectl get pv
```

PV/PVC는 저장소를 애플리케이션과 분리해서 관리하는 핵심 구조다. 파드는 PVC를 사용하고, PVC는 조건에 맞는 PV와 연결된다.

### 5.6 ReplicaSet과 Deployment 스케일링 흐름

```bash
# whoami 애플리케이션 배치
cd ch06
kubectl apply -f whoami/

# ReplicaSet 확인
kubectl get replicaset whoami-web

# 파드 삭제
kubectl delete pods -l app=whoami-web

# ReplicaSet이 다시 파드를 만드는지 확인
kubectl get pods -l app=whoami-web

# 복제본 수 3개로 변경된 정의 적용
kubectl apply -f whoami/update/whoami-replicas-3.yaml

# 파드 확인
kubectl get pods -l app=whoami-web
```

ReplicaSet은 파드가 몇 개 있어야 하는지 계속 맞춘다. 그래서 파드를 지워도 다시 생긴다.

```bash
# 디플로이먼트 복제본 수를 명령어로 조정
kubectl scale --replicas=4 deploy/pi-web

# ReplicaSet 확인
kubectl get rs -l app=pi-web

# 다시 YAML 정의 기준으로 복제본 수 조정
kubectl apply -f pi/web/update/web-replicas-3.yaml
```

`kubectl scale`은 빠르게 수동 조정할 때 편하지만, Git으로 관리하는 환경에서는 YAML 정의와 불일치가 생길 수 있다.

### 5.7 DaemonSet 흐름

```bash
# DaemonSet 적용
kubectl apply -f pi/proxy/daemonset/nginx-ds.yaml

# 엔드포인트 확인
kubectl get endpoints pi-proxy

# 기존 Deployment 삭제
kubectl delete deploy pi-proxy

# DaemonSet 확인
kubectl get daemonset pi-proxy

# DaemonSet이 만든 파드 확인
kubectl get po -l app=pi-proxy
```

DaemonSet은 모든 노드에 파드를 하나씩 배치할 때 사용한다. 로그 수집기나 노드 프록시처럼 노드별로 반드시 떠 있어야 하는 구성에 적합하다.

### 5.8 명령어별로 정리

| 명령어 | 언제 쓰는가 | 내가 이해한 의미 |
|---|---|---|
| `kubectl create configmap` | ConfigMap을 명령어로 만들 때 | 설정 데이터를 Kubernetes 리소스로 분리 |
| `kubectl describe cm` | ConfigMap 상세를 볼 때 | 어떤 키와 값이 들어 있는지 확인 |
| `kubectl exec deploy/... -- printenv` | 컨테이너 환경 변수 확인 | 설정이 실제로 주입됐는지 확인 |
| `kubectl exec ... -- ls` | 컨테이너 내부 파일 확인 | 마운트나 파일 생성 여부 확인 |
| `kubectl get pv` | PersistentVolume 확인 | 클러스터 저장소 상태 확인 |
| `kubectl get pvc` | PersistentVolumeClaim 확인 | 애플리케이션 저장소 요청 상태 확인 |
| `kubectl get rs` | ReplicaSet 확인 | 복제본 관리 상태 확인 |
| `kubectl scale` | 복제본 수를 즉시 변경할 때 | 실행 중인 워크로드 처리 용량 조정 |
| `kubectl get daemonset` | DaemonSet 확인 | 노드별 파드 배치 상태 확인 |
| `kubectl label node` | 노드에 레이블을 붙일 때 | 특정 노드 선택 기준 생성 |

### 5.9 헷갈렸던 지점

ConfigMap과 Secret은 값을 저장하지만, 그 자체로 애플리케이션에 자동 적용되는 것은 아니다. 파드 정의에서 명시적으로 참조해야 한다.

PV와 PVC도 비슷하다. PV를 만들었다고 파드가 바로 쓰는 것이 아니라, PVC를 통해 요청하고 파드에서 그 PVC를 마운트해야 한다.

### 5.10 나중에 다시 볼 포인트

- ConfigMap과 Secret은 무엇이 다르고 어디에 쓰는가?
- 환경 변수 주입과 파일 마운트 방식은 언제 구분해서 쓰는가?
- `emptyDir`, `hostPath`, PV/PVC의 생명주기 차이는 무엇인가?
- ReplicaSet과 Deployment의 관계는 무엇인가?
- DaemonSet은 Deployment와 무엇이 다른가?

## 6. 보충 설명

- 자료 기반 내용: ConfigMap, Secret, 볼륨, PV/PVC, ReplicaSet, Deployment, DaemonSet을 다룬다.
- 보충 설명: 운영 환경에서는 Secret을 사용하더라도 암호화, 접근 제어, 외부 Secret Manager 연동까지 고려해야 한다.
- 확실하지 않음: 실습 클러스터의 스토리지 클래스나 LoadBalancer 동작 방식은 환경에 따라 다를 수 있다.

## 7. 한 줄 요약

10주차의 핵심은 애플리케이션 실행 자체보다 설정, 데이터, 복제본 수를 Kubernetes 리소스로 분리해서 관리하는 방법을 이해하는 것이다.

## 8. 추가로 공부할 키워드

- ConfigMap
- Secret
- Volume
- `emptyDir`
- `hostPath`
- PersistentVolume
- PersistentVolumeClaim
- ReplicaSet
- DaemonSet
