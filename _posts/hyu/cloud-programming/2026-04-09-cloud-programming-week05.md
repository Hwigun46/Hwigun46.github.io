---
layout: post
title: "클라우드프로그래밍 5주차 학습 정리"
date: 2026-04-09
categories: [hyu]
tags: [cloud-programming, assignment]
---

# 클라우드프로그래밍 5주차: Docker 이미지 공유, 볼륨, Docker Compose

## 1. 이번 주차에서 다룬 내용

5주차는 Docker를 단순히 로컬에서 컨테이너 하나 실행하는 수준에서, 이미지를 공유하고 데이터를 보존하고 여러 컨테이너를 함께 실행하는 수준으로 확장하는 주차다.

이번 주차에서 다룬 내용은 다음과 같다.

- Docker 레지스트리, 리포지터리, 이미지 태그
- Docker Hub에 이미지 푸시
- 사설 Docker 레지스트리 운영
- 이미지 태그 전략
- 공식 이미지와 골든 이미지
- 컨테이너 파일 시스템과 데이터 소멸 문제
- Docker 볼륨과 바인드 마운트
- Docker Compose 파일 구조
- 여러 컨테이너로 구성된 분산 애플리케이션 실행
- 컨테이너 간 통신과 Docker 내장 DNS

## 2. 왜 이 내용을 배우는가

Docker를 제대로 쓰려면 컨테이너를 실행하는 것만으로는 부족하다. 내가 만든 이미지를 다른 서버에서 쓸 수 있어야 하고, 컨테이너가 삭제되어도 중요한 데이터가 남아야 하며, 웹 서버와 데이터베이스처럼 여러 컨테이너가 함께 동작해야 한다.

5주차는 Docker를 개인 실습 도구에서 배포와 운영을 위한 기본 도구로 확장하는 내용이다.

## 3. 먼저 알아야 할 배경지식

- Docker 이미지: 컨테이너 실행 템플릿
- Docker 컨테이너: 이미지로부터 실행된 인스턴스
- 레지스트리: 이미지를 저장하고 배포하는 서버
- 리포지터리: 같은 애플리케이션 이미지가 모이는 저장 공간
- 태그: 이미지 버전 또는 변형을 구분하는 이름
- 볼륨: 컨테이너와 독립적으로 존재하는 데이터 저장 단위
- YAML: Docker Compose 설정 파일에서 사용하는 데이터 표현 형식
- DNS: 이름을 IP 주소로 변환하는 시스템

### 보충 설명

컨테이너는 언제든 삭제되고 다시 만들어질 수 있는 실행 단위다. 그래서 컨테이너 안에 중요한 데이터를 그대로 저장하는 방식은 위험하다. 상태가 필요한 데이터는 컨테이너 밖에 두어야 한다.

## 4. 강의자료 기반 개념 정리

### 4.1 레지스트리, 리포지터리, 이미지 태그

Docker 레지스트리는 이미지를 저장하는 서버다. Docker Hub가 대표적인 공개 레지스트리다.

이미지 참조는 보통 다음 구조로 되어 있다.

```text
docker.io/diamol/golang:latest
```

| 부분 | 의미 |
|---|---|
| `docker.io` | 레지스트리 도메인 |
| `diamol` | 계정 또는 조직 이름 |
| `golang` | 리포지터리 이름 |
| `latest` | 이미지 태그 |

`latest`는 이름만 보면 최신처럼 보이지만, 항상 최신이라는 보장은 없다. 운영 환경에서는 `v1`, `2.1.106` 같은 명확한 버전 태그를 쓰는 것이 안전하다.

### 4.2 Docker Hub에 이미지 푸시

로컬에서 만든 이미지를 다른 환경에서 쓰려면 레지스트리에 올려야 한다. Docker Hub에 이미지를 푸시하려면 먼저 로그인하고, 이미지 이름에 내 계정명을 포함한 태그를 붙여야 한다.

이 과정은 Git과 비슷하게 이해할 수 있다.

```text
로컬 이미지 준비
→ 계정명/태그 붙이기
→ Docker Hub 로그인
→ push로 원격 레지스트리에 업로드
```

### 4.3 사설 레지스트리

사설 레지스트리는 조직 내부에서 이미지를 저장하고 배포하기 위한 레지스트리다. 실습에서는 레지스트리 자체도 컨테이너로 실행한다.

운영 환경에서 사설 레지스트리를 사용하려면 인증, TLS, 접근 제어가 필요하다. 실습용 비보안 레지스트리 설정을 그대로 운영에 쓰면 안 된다.

### 4.4 골든 이미지

골든 이미지는 조직에서 표준으로 사용하는 베이스 이미지다. 보안 패치, 공통 설정, 인증서, 기본 도구 등을 포함할 수 있다.

공식 이미지를 그대로 쓰는 것도 가능하지만, 조직 기준에 맞는 골든 이미지를 만들면 여러 팀이 일관된 기준으로 이미지를 빌드할 수 있다.

### 4.5 볼륨과 바인드 마운트

컨테이너 내부 파일 시스템은 컨테이너 생명주기와 함께 사라질 수 있다. 그래서 데이터를 보존하려면 볼륨을 사용한다.

| 방식 | 의미 | 주로 쓰는 상황 |
|---|---|---|
| Docker 볼륨 | Docker가 관리하는 저장 공간 | 애플리케이션 데이터 보존 |
| 바인드 마운트 | 호스트의 특정 경로를 컨테이너에 연결 | 개발 중 소스나 설정 파일 연결 |

### 4.6 Docker Compose

Docker Compose는 여러 컨테이너 실행 설정을 YAML 파일로 정의하고 한 번에 실행하는 도구다.

매번 `docker run` 명령을 길게 입력하면 실수하기 쉽다. Compose 파일은 여러 `docker run` 옵션을 코드로 모아둔 형태로 이해하면 된다.

### 4.7 컨테이너 간 통신

Docker 네트워크 안의 컨테이너들은 Docker 내장 DNS를 통해 컨테이너 이름 또는 서비스 이름으로 서로를 찾을 수 있다.

컨테이너 IP는 재생성될 때 바뀔 수 있으므로 IP를 직접 쓰기보다 이름 기반 통신을 쓰는 것이 좋다.

## 5. 내가 몰랐을 가능성이 높은 개념

| 개념 | 왜 중요한가 | 이해 포인트 |
|---|---|---|
| 레지스트리 | 이미지 배포의 중심 | 이미지를 저장하고 내려받는 서버 |
| 리포지터리 | 이미지 묶음 단위 | 같은 애플리케이션 이미지의 저장 공간 |
| 태그 | 버전 관리 | `latest`에 의존하면 재현성이 떨어질 수 있음 |
| 골든 이미지 | 조직 표준화 | 보안·설정 기준을 포함한 베이스 이미지 |
| 볼륨 | 데이터 보존 | 컨테이너 삭제와 독립적으로 데이터 유지 |
| 바인드 마운트 | 호스트 경로 연결 | 개발 환경에서 설정/소스 공유에 유용 |
| Docker Compose | 다중 컨테이너 실행 | 애플리케이션 구성을 YAML로 정의 |
| Docker DNS | 컨테이너 간 이름 기반 통신 | 같은 네트워크에서 서비스 이름으로 접근 |

## 6. 실습하면서 내 것으로 만들 부분

5주차 실습은 Docker 명령어가 많다. 외워서 해결할 양이 아니다. 흐름별로 나누어 기억해야 한다.

```text
이미지 공유 흐름
→ 레지스트리 로그인
→ 이미지 태그 지정
→ 이미지 푸시

데이터 보존 흐름
→ 컨테이너 내부 데이터 확인
→ 볼륨 생성
→ 컨테이너에 볼륨 연결
→ 컨테이너 삭제 후 데이터 유지 확인

다중 컨테이너 흐름
→ 네트워크 생성
→ Compose 파일 작성
→ docker-compose up
→ 로그와 상태 확인
→ docker-compose down
```

### 6.1 Docker Hub에 이미지 올리는 흐름

```bash
# Docker Hub 계정명을 환경 변수로 저장한다.
export dockerId="도커허브계정이름"

# Docker Hub에 로그인한다.
docker login --username $dockerId

# 로컬 이미지에 Docker Hub용 이름과 태그를 붙인다.
docker image tag image-gallery $dockerId/image-gallery:v1

# 이미지 참조 목록을 확인한다.
docker image ls --filter reference=image-gallery --filter reference='*/image-gallery'

# Docker Hub로 이미지를 푸시한다.
docker image push $dockerId/image-gallery:v1

# 업로드된 이미지 태그 페이지 URL을 확인한다.
echo "https://hub.docker.com/r/$dockerId/image-gallery/tags"
```

여기서 `docker image tag`는 이미지를 새로 복사하는 느낌이 아니라, 같은 이미지에 새로운 이름표를 붙이는 것에 가깝다.

### 6.2 사설 레지스트리 운영 흐름

```bash
# 로컬 레지스트리를 컨테이너로 실행한다.
docker container run -d -p 5000:5000 --restart always diamol/registry

# 리눅스/macOS에서 registry.local 이름을 로컬 주소에 연결한다.
echo $'\n127.0.0.1 registry.local' | sudo tee -a /etc/hosts

# image-gallery 이미지에 사설 레지스트리 주소를 붙인다.
docker image tag image-gallery registry.local:5000/gallery/ui:v1

# Docker 설정에서 비보안 레지스트리 허용 여부를 확인한다.
docker info

# 사설 레지스트리에 푸시한다.
docker image push registry.local:5000/gallery/ui:v1
```

강의자료에는 비보안 레지스트리 허용 설정도 등장한다.

```json
{
  "insecure-registries" : ["registry.local:5000"]
}
```

이 설정은 실습용으로 이해해야 한다. 운영 환경에서는 TLS와 인증을 적용해야 한다.

### 6.3 이미지 태그를 버전처럼 다루는 흐름

```bash
docker image tag image-gallery registry.local:5000/gallery/ui:latest
docker image tag image-gallery registry.local:5000/gallery/ui:2
docker image tag image-gallery registry.local:5000/gallery/ui:2.1
docker image tag image-gallery registry.local:5000/gallery/ui:2.1.106
```

같은 이미지에도 여러 태그를 붙일 수 있다. 중요한 것은 태그가 운영 배포의 기준이 될 수 있다는 점이다. `latest`만 쓰면 나중에 같은 환경을 재현하기 어렵다.

### 6.4 볼륨과 파일 복사 흐름

컨테이너 내부 파일이 컨테이너마다 다를 수 있고, 삭제되면 사라질 수 있다는 점을 확인하는 명령어 흐름이다.

```bash
# 서로 다른 컨테이너 실행
docker container run --name rn1 diamol/ch06-random-number
docker container run --name rn2 diamol/ch06-random-number

# 컨테이너 내부 파일을 호스트로 복사
docker container cp rn1:/random/number.txt number1.txt
docker container cp rn2:/random/number.txt number2.txt
```

컨테이너에 파일을 넣고 다시 시작하는 흐름도 등장한다.

```bash
docker container run --name f1 diamol/ch06-file-display
echo "http://eltonstoneman.com" > url.txt
docker container cp url.txt f1:/input.txt
docker container start --attach f1
```

여기서 `docker container cp`는 컨테이너와 호스트 사이에 파일을 옮길 때 사용한다.

### 6.5 Docker 볼륨을 쓰는 흐름

```bash
# 볼륨 생성
docker volume create todo-list

# 볼륨을 연결해서 컨테이너 실행
docker container run -d -p 8011:80 -v todo-list:$target --name todo-v1 diamol/ch06-todo-list

# 컨테이너 삭제
docker container rm -f todo-v1

# 같은 볼륨을 새 컨테이너에 다시 연결
docker container run -d -p 8011:80 -v todo-list:$target --name todo-v2 diamol/ch06-todo-list:v2
```

이 흐름에서 중요한 것은 컨테이너가 바뀌어도 볼륨은 남는다는 점이다. 데이터는 컨테이너에 종속시키지 말고 볼륨에 둔다.

### 6.6 바인드 마운트 흐름

```bash
# 호스트 경로를 컨테이너 내부 경로에 연결
docker container run --mount type=bind,source=$source,target=$target -d -p 8012:80 diamol/ch06-todo-list

# 웹 요청 확인
curl http://localhost:8012
```

바인드 마운트는 개발 중 로컬 파일을 컨테이너에 바로 연결할 때 유용하다. 다만 호스트 경로에 의존하므로 운영 환경에서는 이식성이 떨어질 수 있다.

### 6.7 Docker Compose 흐름

```bash
# Docker 네트워크 생성
docker network create nat

# Compose 파일 기준으로 애플리케이션 실행
docker-compose up

# 백그라운드 실행
docker-compose up --detach

# 특정 서비스 개수를 늘려 실행
docker-compose up -d --scale iotd=3

# 특정 서비스 로그 확인
docker-compose logs --tail=1 iotd

# Compose 애플리케이션 중지
docker-compose stop

# 다시 시작
docker-compose start

# 컨테이너 목록 확인
docker container ls

# Compose 애플리케이션 제거
docker-compose down
```

Compose는 여러 컨테이너를 하나의 애플리케이션 단위로 다루게 해준다. `docker run`을 여러 번 치는 대신, YAML 파일에 실행 구성을 적고 `docker-compose up`으로 한 번에 올리는 방식이다.

### 6.8 명령어별로 정리

| 명령어 | 언제 쓰는가 | 내가 이해한 의미 |
|---|---|---|
| `docker login` | 레지스트리에 로그인할 때 | 이미지 push 권한을 얻음 |
| `docker image tag` | 이미지 이름/버전을 붙일 때 | 이미지에 새 이름표를 붙임 |
| `docker image push` | 레지스트리에 업로드할 때 | 로컬 이미지를 원격 저장소로 보냄 |
| `docker container cp` | 컨테이너와 호스트 간 파일 복사 | 컨테이너 내부 파일을 꺼내거나 넣음 |
| `docker volume create` | 볼륨을 만들 때 | 컨테이너와 독립적인 저장 공간 생성 |
| `docker container run -v` | 볼륨을 붙여 실행할 때 | 컨테이너 데이터 보존 연결 |
| `docker container run --mount` | 마운트를 명시적으로 설정할 때 | 볼륨/바인드 마운트를 구조적으로 지정 |
| `docker network create` | 컨테이너 네트워크 생성 | 컨테이너 간 통신 공간 생성 |
| `docker-compose up` | Compose 앱 실행 | 여러 컨테이너를 한 번에 실행 |
| `docker-compose down` | Compose 앱 정리 | 관련 컨테이너와 네트워크 제거 |

### 6.9 헷갈렸던 지점

볼륨과 바인드 마운트는 둘 다 컨테이너 밖의 저장 공간을 연결하지만 목적이 다르다. 볼륨은 Docker가 관리하는 데이터 저장소이고, 바인드 마운트는 호스트의 특정 경로를 직접 연결한다.

Compose도 Kubernetes와 헷갈리기 쉽다. Compose는 보통 단일 Docker 호스트에서 여러 컨테이너를 편하게 실행하는 도구이고, Kubernetes는 여러 노드에서 컨테이너를 운영하는 오케스트레이션 플랫폼이다.

## 7. 보충 설명

- 자료 기반 내용: Docker Hub, 레지스트리, 리포지터리, 이미지 태그, 사설 레지스트리, 골든 이미지, 볼륨, 바인드 마운트, Docker Compose, Docker 네트워크
- 보충 설명: Docker Compose 파일은 여러 `docker run` 명령어를 코드로 정리한 실행 문서로 이해하면 쉽다.
- 확실하지 않음: 실습 소스 코드 전체와 실제 실행 결과 로그는 업로드된 강의자료만으로는 확인할 수 없음.

## 8. 한 줄 요약

Docker를 운영 관점에서 사용하려면 이미지 공유, 데이터 보존, 다중 컨테이너 실행 흐름을 함께 이해해야 한다.

## 9. 추가로 공부할 키워드

- Docker Registry
- Docker Hub
- Image Tag
- Golden Image
- Docker Volume
- Bind Mount
- Docker Compose
- Docker Network
- Docker DNS
