---
layout: post
title: "클라우드프로그래밍 7주차 학습 정리"
date: 2026-04-16
categories: [hyu]
tags: [cloud-programming, assignment]
---

# 클라우드프로그래밍 7주차: Kubernetes 기본 구조와 클러스터 구성

## 1. 이번 주차에서 다룬 내용

7주차는 Docker 컨테이너를 여러 서버에서 운영하기 위한 Kubernetes의 기본 구조를 다룬다. Docker가 컨테이너를 실행하는 도구라면, Kubernetes는 여러 노드 위에서 컨테이너를 배치하고 상태를 유지하는 오케스트레이션 플랫폼이다.

이번 주차에서 다룬 내용은 다음과 같다.

- OpenStack과 Kubernetes 구조 비교
- Kubernetes의 특징
- Kubernetes 클러스터 구조
- 컨테이너와 파드
- 노드와 클러스터
- control plane과 worker node
- 실습 환경 구성
- kubeadm, kubectl, kubelet, containerd
- Calico 기반 네트워크 구성
- AWS EKS 등 다양한 Kubernetes 실행 환경

## 2. 왜 이 내용을 배우는가

Docker는 컨테이너 하나를 실행하는 데 유용하다. 하지만 실제 서비스에서는 컨테이너가 여러 개이고, 서버도 여러 대일 수 있다. 이때 컨테이너를 어느 서버에 배치할지, 실패하면 어떻게 다시 띄울지, 네트워크는 어떻게 연결할지, 배포는 어떻게 관리할지 결정해야 한다.

Kubernetes는 이런 문제를 해결하기 위한 플랫폼이다. 핵심은 “컨테이너를 직접 하나씩 실행한다”가 아니라 “원하는 상태를 선언하면 Kubernetes가 실제 상태를 맞춘다”는 사고방식이다.

## 3. 먼저 알아야 할 배경지식

- 컨테이너: 애플리케이션 실행 환경을 격리한 단위
- Docker 이미지: 컨테이너 실행을 위한 템플릿
- 클러스터: 여러 서버를 하나의 시스템처럼 묶은 구조
- 노드: 클러스터를 구성하는 서버
- 오케스트레이션: 배포, 스케줄링, 확장, 복구를 자동화하는 관리 방식
- YAML: Kubernetes 리소스 정의에 자주 사용하는 형식

### 보충 설명

Kubernetes를 처음 배울 때 가장 중요한 것은 단위를 바꾸는 것이다. Docker에서는 컨테이너를 직접 다루지만, Kubernetes에서는 보통 파드, 디플로이먼트, 서비스 같은 리소스를 다룬다.

## 4. 강의자료 기반 개념 정리

### 4.1 OpenStack과 Kubernetes

OpenStack은 가상머신, 네트워크, 스토리지 같은 클라우드 인프라를 구성하는 플랫폼이고, Kubernetes는 컨테이너화된 애플리케이션을 배포하고 관리하는 플랫폼이다.

둘은 완전히 같은 계층의 도구가 아니다. OpenStack 위에 가상머신을 만들고, 그 위에 Kubernetes 클러스터를 구성할 수도 있다.

### 4.2 Kubernetes의 특징

Kubernetes는 컨테이너화된 애플리케이션을 배포, 확장, 관리하기 위한 오픈소스 플랫폼이다. 주요 클라우드 서비스는 대부분 관리형 Kubernetes 환경을 제공한다.

Kubernetes의 장점은 표준성이다. 애플리케이션을 Kubernetes에서 동작하도록 만들면 로컬, 데이터센터, 클라우드 환경에서 비슷한 방식으로 배포할 수 있다.

### 4.3 클러스터와 노드

Kubernetes 클러스터는 여러 노드가 모여 구성된다.

| 개념 | 의미 |
|---|---|
| Cluster | Kubernetes 전체 실행 단위 |
| Node | 클러스터를 구성하는 서버 |
| Control Plane | 클러스터 상태를 제어하는 영역 |
| Worker Node | 실제 워크로드가 실행되는 서버 |

control plane은 사용자 애플리케이션이 주로 실행되는 곳이라기보다 클러스터 제어를 담당하는 곳이다. worker node는 실제 파드가 실행되는 서버다.

### 4.4 파드

파드는 Kubernetes에서 컨테이너를 실행하는 최소 단위다. 하나의 파드 안에는 하나 이상의 컨테이너가 들어갈 수 있다.

처음에는 “왜 컨테이너를 바로 실행하지 않고 파드로 감싸는가”가 헷갈릴 수 있다. Kubernetes는 컨테이너 하나보다 더 큰 실행 단위가 필요하다. 파드는 컨테이너와 네트워크, 볼륨 같은 실행 맥락을 함께 묶는다.

### 4.5 kubeadm, kubectl, kubelet, containerd

| 도구 | 역할 |
|---|---|
| kubeadm | Kubernetes 클러스터 초기화와 노드 조인 |
| kubectl | Kubernetes 클러스터를 제어하는 CLI |
| kubelet | 각 노드에서 파드 상태를 관리하는 에이전트 |
| containerd | 실제 컨테이너 실행을 담당하는 런타임 |

`kubectl`은 사용자가 클러스터와 대화하는 명령어이고, `kubelet`은 각 노드에서 실제 파드 상태를 맞추는 역할을 한다.

### 4.6 CNI와 Calico

Kubernetes에서 파드들이 서로 통신하려면 네트워크 플러그인이 필요하다. CNI는 컨테이너 네트워크를 구성하기 위한 인터페이스이고, Calico는 대표적인 CNI 플러그인 중 하나다.

CNI가 제대로 설치되지 않으면 노드가 준비 상태가 되지 않거나, 파드 간 통신이 되지 않을 수 있다.

## 5. 내가 몰랐을 가능성이 높은 개념

| 개념 | 왜 중요한가 | 이해 포인트 |
|---|---|---|
| Cluster | Kubernetes의 전체 실행 단위 | 여러 노드를 하나로 묶은 관리 시스템 |
| Node | 컨테이너가 실행되는 서버 | worker node와 control plane 구분 |
| Pod | Kubernetes 기본 실행 단위 | 컨테이너를 감싸는 최소 단위 |
| Control Plane | 클러스터 제어 영역 | 원하는 상태와 실제 상태를 조정 |
| kubeadm | 클러스터 생성 도구 | 초기화와 노드 조인에 사용 |
| kubectl | Kubernetes CLI | 클러스터 리소스 조회와 제어 |
| kubelet | 노드 에이전트 | 각 노드에서 파드 실행 상태 관리 |
| containerd | 컨테이너 런타임 | 실제 컨테이너 실행 담당 |
| CNI | 컨테이너 네트워크 인터페이스 | 파드 네트워크 구성 |
| Calico | CNI 플러그인 | 파드 간 네트워크 연결 제공 |

## 6. 실습하면서 내 것으로 만들 부분

7주차 실습은 명령어가 가장 많고 무겁다. 그래서 한 번에 외우려고 하면 안 된다. Kubernetes 클러스터 구축은 아래 흐름으로 나누어 보는 것이 낫다.

```text
노드 준비
→ 커널/네트워크 설정
→ containerd 설치와 설정
→ swap 비활성화
→ kubeadm/kubelet/kubectl 설치
→ control plane 초기화
→ kubectl 설정
→ CNI 설치
→ worker node join
→ 노드와 파드 상태 확인
```

### 6.1 노드 이름과 hosts 설정

```bash
# 노드 이름과 IP 매핑을 설정한다.
sudo vim /etc/hosts
```

Kubernetes 클러스터에서는 노드들이 서로 이름으로 통신해야 하는 경우가 있다. `/etc/hosts`는 이름과 IP를 수동으로 매핑하는 파일이다.

### 6.2 커널 모듈과 네트워크 설정

```bash
# root 권한으로 전환
sudo -i

# Kubernetes에 필요한 커널 모듈 설정 파일 작성
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

# 커널 모듈 로드
sudo modprobe overlay
sudo modprobe br_netfilter

# sysctl 설정 적용
sudo sysctl --system
```

`overlay`는 컨테이너 파일 시스템과 관련이 있고, `br_netfilter`는 브리지 네트워크 트래픽을 필터링하기 위해 필요하다. 이 부분은 명령어 자체보다 Kubernetes가 리눅스 네트워크 기능 위에서 동작한다는 점을 이해하는 것이 중요하다.

### 6.3 containerd 설치와 설정

```bash
# 패키지 목록 갱신 및 기본 패키지 설치
sudo apt-get update
sudo apt-get install ca-certificates curl

# Docker 저장소 키와 저장소 설정
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Docker 관련 패키지 설치
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# containerd 설정 파일 생성
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml > /dev/null

# containerd 설정 수정
sudo vim /etc/containerd/config.toml

# containerd 재시작 및 활성화
sudo systemctl restart containerd
sudo systemctl enable containerd
sudo systemctl status containerd
```

Kubernetes는 컨테이너를 직접 실행하지 않고 container runtime을 통해 실행한다. 여기서는 containerd가 그 역할을 한다.

### 6.4 swap과 방화벽 설정

```bash
# swap 사용 여부 확인
cat /proc/swaps

# swap 비활성화
sudo -i
swapoff --all

# ufw 상태 확인 및 비활성화
sudo ufw status
sudo ufw disable
```

Kubernetes는 일반적으로 swap이 꺼져 있어야 안정적으로 동작한다. 실습에서는 방화벽도 비활성화하지만, 운영 환경에서 방화벽을 무조건 끄는 것은 적절하지 않다. 실습 편의를 위한 설정으로 이해해야 한다.

### 6.5 kubelet, kubeadm, kubectl 설치

```bash
# Kubernetes 패키지 저장소 설정에 필요한 패키지 설치
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

# keyring 디렉터리 생성
sudo mkdir -p /etc/apt/keyrings

# Kubernetes 저장소 키 추가
sudo curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Kubernetes 저장소 추가
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Kubernetes 도구 설치
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl

# 자동 업데이트로 버전이 바뀌지 않게 고정
sudo apt-mark hold kubelet kubeadm kubectl

# 버전 확인
kubelet --version
kubectl version --output=yaml

# kubelet 재시작 및 활성화
sudo systemctl daemon-reload
sudo systemctl restart kubelet.service
sudo systemctl enable --now kubelet.service
```

세 도구의 역할은 다르다. `kubeadm`은 클러스터를 만들고, `kubectl`은 클러스터를 조작하고, `kubelet`은 노드에서 파드를 관리한다.

### 6.6 control plane 초기화

```bash
# 인증서 만료 확인
kubeadm certs check-expiration

# 필요한 이미지 목록 확인
kubeadm config images list

# 필요한 이미지 미리 다운로드
kubeadm config images pull
kubeadm config images pull --cri-socket /run/containerd/containerd.sock

# control plane 초기화
kubeadm init --apiserver-advertise-address=<master-ip> --pod-network-cidr=192.168.0.0/16 --cri-socket /run/containerd/containerd.sock
```

자료에는 `~~~~`처럼 실제 IP가 가려진 형태로 등장한다. 실제 실습에서는 `<master-ip>` 자리에 control plane 노드의 IP를 넣어야 한다.

초기화에 실패했을 때는 reset 흐름도 나온다.

```bash
sudo kubeadm reset cleanup-node
sudo systemctl restart kubelet
kubeadm init
```

### 6.7 kubectl 설정

```bash
# 일반 사용자 계정에서 kubectl을 쓰기 위한 설정
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

`kubectl`은 클러스터 API 서버와 통신해야 한다. 이때 필요한 인증 정보가 kubeconfig 파일이다.

### 6.8 Calico 설치와 상태 확인

```bash
# Calico 리소스 설치
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/<version>/manifests/tigera-operator.yaml

# custom-resources.yaml 적용
kubectl create -f custom-resources.yaml

# Calico 파드 상태 확인
watch kubectl get pods -n calico-system

# 노드 상태 확인
kubectl get node -o wide
kubectl get node
```

Calico는 파드 네트워크를 구성하기 위한 CNI 플러그인이다. CNI가 제대로 올라오지 않으면 노드가 `Ready`가 되지 않을 수 있다.

### 6.9 worker node join 흐름

```bash
# join 명령어 재생성
kubeadm token create --print-join-command

# worker node에서 실행
kubeadm join <master-ip>:<port> --token <token> --discovery-token-ca-cert-hash sha256:<hash> --cri-socket /run/containerd/containerd.sock
```

`kubeadm join`은 worker node를 control plane에 등록하는 명령어다. token과 hash는 클러스터마다 다르므로 자료의 값을 그대로 쓰면 안 된다.

### 6.10 명령어별로 정리

| 명령어 | 언제 쓰는가 | 내가 이해한 의미 |
|---|---|---|
| `sudo vim /etc/hosts` | 노드 이름과 IP 매핑을 설정할 때 | 클러스터 노드 이름 해석 준비 |
| `sudo modprobe overlay` | overlay 모듈을 로드할 때 | 컨테이너 파일 시스템 관련 준비 |
| `sudo modprobe br_netfilter` | 브리지 네트워크 필터링 준비 | Kubernetes 네트워크 요구사항 충족 |
| `sudo sysctl --system` | 커널 네트워크 설정 적용 | sysctl 설정 반영 |
| `containerd config default` | containerd 기본 설정 생성 | 런타임 설정 파일 준비 |
| `swapoff --all` | swap 비활성화 | kubelet 동작 조건 충족 |
| `sudo apt-mark hold` | 패키지 버전 고정 | Kubernetes 구성 요소 버전 변경 방지 |
| `kubeadm init` | control plane 초기화 | 클러스터 생성 시작 |
| `kubeadm join` | worker node 참여 | 노드를 클러스터에 등록 |
| `kubectl get nodes` | 노드 상태 확인 | 클러스터 노드 Ready 여부 확인 |
| `kubectl get pods -A` | 전체 파드 확인 | 시스템 파드 상태 확인 |
| `kubectl create -f` | YAML 리소스 적용 | Kubernetes 리소스 생성 |

### 6.11 헷갈렸던 지점

`kubeadm`, `kubectl`, `kubelet`은 이름이 비슷하지만 역할이 다르다. `kubeadm`은 클러스터를 만드는 도구, `kubectl`은 클러스터에 명령을 보내는 도구, `kubelet`은 각 노드에서 파드 상태를 맞추는 에이전트다.

또 하나 중요한 점은 Kubernetes를 설치했다고 바로 애플리케이션이 돌아가는 것이 아니라는 점이다. container runtime, kubelet, control plane, CNI가 모두 맞아야 클러스터가 정상 상태가 된다.

### 6.12 나중에 다시 볼 포인트

- Kubernetes는 컨테이너를 직접 관리하는 것이 아니라 파드를 관리한다.
- worker node는 실제 워크로드를 실행한다.
- control plane은 클러스터 상태를 제어한다.
- CNI가 없으면 파드 네트워크가 정상 동작하지 않는다.
- `kubectl`은 클러스터와 통신하는 핵심 CLI다.
- token, hash, IP 주소는 실습 환경마다 다르므로 그대로 복붙하면 안 된다.

## 7. 보충 설명

- 자료 기반 내용: OpenStack과 Kubernetes 구조 비교, Kubernetes 특징, 클러스터 구조, 파드, 노드, 실습 환경, kubeadm 기반 클러스터 구축, 다양한 Kubernetes 환경
- 보충 설명: Kubernetes는 Docker보다 훨씬 큰 시스템이다. 명령어를 따라 치는 것보다 “원하는 상태를 선언하면 control plane이 실제 상태를 맞춘다”는 구조를 이해해야 한다.
- 확실하지 않음: 실습에서 사용한 실제 IP 주소, 노드 수, Kubernetes 세부 버전은 자료에 예시로 보이지만 사용자의 실제 실습 환경과 동일한지는 확실하지 않음.

## 8. 한 줄 요약

Kubernetes는 여러 노드 위에서 컨테이너를 파드 단위로 배치하고, 원하는 상태를 유지하도록 관리하는 컨테이너 오케스트레이션 플랫폼이다.

## 9. 추가로 공부할 키워드

- Kubernetes
- Cluster
- Node
- Pod
- Control Plane
- Worker Node
- kubeadm
- kubectl
- kubelet
- containerd
- CNI
- Calico
