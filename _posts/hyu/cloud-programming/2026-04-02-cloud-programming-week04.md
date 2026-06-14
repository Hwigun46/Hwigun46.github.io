---
layout: post
title: "클라우드프로그래밍 4주차 학습 정리"
date: 2026-04-02
categories: [hyu]
tags: [cloud-programming, assignment]
---

# 클라우드프로그래밍 4주차: Docker 개념과 컨테이너 실행 환경

## 1. 이번 주차에서 다룬 내용

4주차부터는 Docker와 컨테이너를 본격적으로 다룬다. 앞 주차까지가 클라우드 인프라의 바탕이었다면, 이번 주차는 애플리케이션을 어떻게 일관된 환경에서 실행할 것인가에 초점이 있다.

이번 주차에서 다룬 내용은 다음과 같다.

- 컨테이너의 등장 배경
- 모놀리식 구조와 마이크로서비스 구조
- DevOps와 컨테이너의 관계
- 컨테이너와 가상머신의 차이
- Docker의 특징과 아키텍처
- 컨테이너를 원격 컴퓨터처럼 사용하는 방법
- Docker Hub 이미지 사용
- Dockerfile 작성과 이미지 빌드
- 이미지 레이어와 레이어 캐시
- 컨테이너를 이용한 웹 사이트 호스팅

## 2. Background

- 운영체제: 프로세스와 자원을 관리하는 시스템 소프트웨어
- 프로세스: 실행 중인 프로그램
- 라이브러리 의존성: 프로그램 실행에 필요한 외부 코드와 버전
- 이미지: 컨테이너 실행에 필요한 파일 시스템과 설정 묶음
- 컨테이너: 이미지로부터 실행된 격리된 프로세스 환경
- 포트 매핑: 호스트 포트와 컨테이너 포트를 연결하는 방식

### 보충 설명

컨테이너는 가상머신보다 가볍지만, 가상머신과 같은 개념은 아니다. 가상머신은 게스트 OS를 포함하지만, 컨테이너는 호스트 OS 커널을 공유하면서 프로세스와 파일 시스템을 격리한다.

## 3. 개념 정리

### 3.1 컨테이너

컨테이너는 애플리케이션 실행에 필요한 환경을 독립적으로 운용할 수 있도록 격리해주는 운영체제 수준의 기술이다.

애플리케이션은 OS, 라이브러리, 런타임 버전에 의존한다. 서로 다른 버전을 요구하는 애플리케이션을 하나의 서버에서 실행하면 충돌이 생길 수 있다. 컨테이너는 실행 환경을 분리해서 이 문제를 줄인다.

내가 이해한 방식으로는 컨테이너는 애플리케이션을 실행하기 위한 밀봉된 실행 상자다. 상자 안에 필요한 파일과 설정을 넣어두고, 어디서 실행하든 비슷하게 동작하게 만드는 것이 목적이다.

### 3.2 Docker

Docker는 컨테이너를 생성, 실행, 배포, 관리하기 위한 플랫폼이다.

컨테이너 기술 자체만으로는 이미지 관리, 실행 명령, 로그 확인, 네트워크 연결, 빌드 자동화가 불편하다. Docker는 이런 작업을 표준화된 명령과 구조로 제공한다.

### 3.3 이미지와 컨테이너

Docker를 처음 배울 때 가장 헷갈리는 것이 이미지와 컨테이너의 차이다.

| 개념 | 의미 | 비유 |
|---|---|---|
| 이미지 | 컨테이너 실행에 필요한 템플릿 | 프로그램 설치 파일 |
| 컨테이너 | 이미지를 실행한 상태 | 실행 중인 프로그램 |

이미지는 실행 전 상태이고, 컨테이너는 실행 중인 상태다. `docker image build`는 이미지를 만들고, `docker container run`은 이미지를 컨테이너로 실행한다.

### 3.4 Dockerfile

Dockerfile은 이미지를 만들기 위한 스크립트다. 서버에 직접 접속해서 하나씩 설치하던 작업을 파일로 기록해두는 방식이라고 볼 수 있다.

이 방식이 중요한 이유는 재현성이다. Dockerfile이 있으면 같은 이미지와 같은 실행 환경을 다시 만들 수 있다.

### 3.5 이미지 레이어와 캐시

Docker 이미지는 여러 레이어로 구성된다. Dockerfile의 각 단계가 레이어를 만들고, 변경되지 않은 레이어는 캐시로 재사용된다.

따라서 Dockerfile을 작성할 때 자주 바뀌지 않는 명령을 위쪽에 두고, 자주 바뀌는 소스 복사를 아래쪽에 두면 빌드 시간을 줄일 수 있다.

## 4. 내가 몰랐을 가능성이 높은 개념

| 개념 | 왜 중요한가 | 이해 포인트 |
|---|---|---|
| 이미지 | 컨테이너 실행의 재료 | 실행 전 상태의 템플릿 |
| 컨테이너 | 실제 실행 단위 | 이미지가 실행된 인스턴스 |
| Dockerfile | 이미지 빌드 자동화 | 실행 환경을 코드로 기록 |
| 레이어 | 이미지 구성 단위 | 빌드 단계별 변경분 |
| 레이어 캐시 | 빌드 최적화 | 변경 없는 레이어 재사용 |
| 포트 매핑 | 외부 접근 연결 | 호스트 포트와 컨테이너 포트 연결 |
| 환경 변수 | 실행 설정 주입 | 이미지 변경 없이 실행 설정 변경 |

## 5. 실습하면서 내 것으로 만들 부분

4주차 실습은 Docker 명령어의 기본 흐름을 손에 익히는 것이 중요하다. 여기서 명령어는 따로따로 외우면 잘 남지 않는다. 흐름으로 기억해야 한다.

내가 Docker를 쓸 때의 기본 흐름은 다음과 같다.

```text
이미지를 받는다 또는 만든다
→ 컨테이너로 실행한다
→ 실행 상태를 확인한다
→ 로그와 상세 정보를 확인한다
→ 필요하면 이미지를 다시 빌드한다
→ 실습이 끝나면 컨테이너를 정리한다
```

### 5.1 Docker 설치와 확인 흐름

Ubuntu 기준으로 Docker를 설치할 때 강의자료에 등장한 흐름은 다음과 같다.

```bash
# 기존 Docker 관련 패키지 제거
sudo apt-get remove docker docker-engine docker.io containerd runc

# Docker 저장소 키 추가
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# Docker 패키지 설치
sudo apt-get install docker-ce docker-ce-cli containerd.io

# Docker 동작 확인
docker version

# Docker Compose 동작 확인
docker-compose version
```

실제로는 운영체제 버전과 Docker 설치 방식에 따라 명령이 달라질 수 있다. 여기서는 강의자료의 흐름을 기준으로 이해하면 된다.

### 5.2 컨테이너를 실행하고 살펴보는 흐름

```bash
# 대화형 컨테이너 실행
docker container run --interactive --tty diamol/base

# 실행 중인 컨테이너 확인
docker container ls

# 특정 컨테이너에서 실행 중인 프로세스 확인
docker container top f1

# 컨테이너 로그 확인
docker container logs f1

# 컨테이너 상세 정보 확인
docker container inspect f1

# 중지된 컨테이너까지 전체 확인
docker container ls --all

# 실행 중인 컨테이너 리소스 사용량 확인
docker container stats

# 모든 컨테이너 강제 삭제
docker container rm --force $(docker container ls --all --quiet)
```

이 흐름은 Docker를 사용할 때 가장 자주 반복된다. 컨테이너를 실행하고, 살아 있는지 보고, 문제가 있으면 로그를 보고, 설정이 궁금하면 inspect를 본다.

### 5.3 이미지 내려받기, 실행, 환경 변수 변경

```bash
# Docker Hub에서 이미지 내려받기
docker image pull diamol/ch03-web-ping

# 백그라운드 컨테이너 실행
docker container run -d --name web-ping diamol/ch03-web-ping

# 로그 확인
docker container logs web-ping

# 기존 컨테이너 삭제
docker rm -f web-ping

# 환경 변수를 바꿔 컨테이너 실행
docker container run --env TARGET=google.com diamol/ch03-web-ping
```

여기서 `--env`는 이미지를 새로 만들지 않고 실행 시점의 설정을 바꾸는 방법이다. 같은 이미지라도 환경 변수를 다르게 주면 다른 대상에 요청을 보내도록 동작을 바꿀 수 있다.

### 5.4 Dockerfile로 이미지를 빌드하는 흐름

```bash
# 현재 디렉터리의 Dockerfile로 이미지 빌드
docker image build --tag web-ping .

# 이미지 목록 확인
docker image ls w*

# 환경 변수를 주고 직접 만든 이미지 실행
docker container run -e TARGET=docker.com -e INTERVAL=5000 web-ping

# 이미지 레이어 확인
docker image history web-ping

# v2 태그로 다시 빌드
docker image build -t web-ping:v2 .
```

`docker image build`는 Dockerfile을 읽어 이미지를 만드는 명령어다. Git으로 비유하면 Dockerfile은 변경 내용을 남기는 파일이고, `docker image build`는 그 파일을 기준으로 실행 가능한 결과물을 만드는 과정에 가깝다.

### 5.5 웹 애플리케이션 컨테이너 실행 흐름

강의자료 후반부에서는 여러 컨테이너를 빌드하고 네트워크에 연결하는 흐름이 등장한다.

```bash
# 애플리케이션 이미지 빌드
docker image build -t image-of-the-day .

# Docker 네트워크 생성
docker network create nat

# 컨테이너를 네트워크에 붙여 실행
docker container run --name iotd -d -p 800:80 --network nat image-of-the-day

# access-log 이미지 빌드
docker image build -t access-log .

# accesslog 컨테이너 실행
docker container run --name accesslog -d -p 801:80 --network nat access-log

# image-gallery 이미지 빌드
docker image build -t image-gallery .

# 이미지 목록 확인
docker image ls -f reference=diamol/golang -f reference=image-gallery

# gallery 컨테이너 실행
docker container run -d -p 802:80 --network nat image-gallery
```

여기서부터 Docker는 단일 컨테이너 실행 도구가 아니라, 여러 컨테이너를 연결해서 하나의 애플리케이션을 구성하는 도구로 확장된다.

### 5.6 명령어별로 정리

| 명령어 | 언제 쓰는가 | 내가 이해한 의미 |
|---|---|---|
| `docker version` | Docker 설치 확인 | Docker CLI와 엔진이 동작하는지 확인 |
| `docker container run` | 컨테이너 실행 | 이미지를 실제 실행 상태로 바꿈 |
| `docker container ls` | 실행 중 컨테이너 확인 | 현재 살아 있는 컨테이너 목록 확인 |
| `docker container logs` | 로그 확인 | 컨테이너 내부 애플리케이션 출력 확인 |
| `docker container inspect` | 상세 정보 확인 | 네트워크, 마운트, 환경 변수 등 확인 |
| `docker image pull` | 이미지 내려받기 | 레지스트리에서 로컬로 이미지 가져오기 |
| `docker image build` | 이미지 빌드 | Dockerfile을 이미지로 변환 |
| `docker image history` | 레이어 확인 | 이미지가 어떤 단계로 만들어졌는지 확인 |
| `docker network create` | 네트워크 생성 | 컨테이너끼리 통신할 공간 생성 |
| `docker container rm --force` | 컨테이너 삭제 | 실습 후 실행 흔적 정리 |

### 5.7 헷갈렸던 지점

`docker run`은 이미지를 만드는 명령어가 아니다. 이미지를 실행해서 컨테이너를 만드는 명령어다. 이미지는 `docker image build`나 `docker image pull`로 준비하고, 컨테이너는 `docker container run`으로 실행한다.

또한 포트 매핑을 하지 않으면 컨테이너 내부에서 웹 서버가 실행 중이어도 호스트 브라우저에서 접근할 수 없다.

## 6. 보충 설명

- 자료 기반 내용: 컨테이너 정의, Docker 특징, Docker 아키텍처, 컨테이너 실행, Docker Hub 이미지 사용, Dockerfile, 이미지 빌드, 이미지 레이어, 레이어 캐시
- 보충 설명: Docker는 서버에 직접 환경을 맞추는 방식에서, 실행 환경 자체를 이미지로 배포하는 방식으로 사고를 바꾸는 도구다.
- 확실하지 않음: 실제 실습 환경의 OS와 Docker 버전은 업로드된 자료만으로는 확실하지 않음.

## 7. 한 줄 요약

Docker는 애플리케이션과 실행 환경을 이미지로 패키징하고, 컨테이너로 실행해 배포 환경의 차이를 줄이는 도구다.

## 8. 추가로 공부할 키워드

- Docker Engine
- Docker CLI
- Dockerfile
- Image Layer
- Container Runtime
- Port Mapping
- Environment Variable
- Multi-stage Build
