---
layout: post
title: "클라우드프로그래밍 6주차 학습 정리"
date: 2026-04-16
categories: [hyu]
tags: [cloud-programming, assignment]
---

# 클라우드프로그래밍 6주차: Docker 기반 웹 서비스 검증과 복원력

## 1. 이번 주차에서 다룬 내용

6주차는 Docker로 실행한 웹 서비스가 단순히 켜져 있는지 확인하는 수준을 넘어서, 실제로 정상 동작하는지 검증하는 내용을 다룬다.

이번 주차에서 다룬 내용은 다음과 같다.

- Docker 헬스 체크
- 애플리케이션 비정상 동작 감지
- Dockerfile의 `HEALTHCHECK`
- 디펜던시 체크
- 커스텀 체크 유틸리티
- Docker Compose에서 헬스 체크와 의존성 정의
- 여러 환경을 위한 Compose 구성
- 환경 변수와 비밀 값 주입
- Compose 확장 필드로 중복 제거
- Docker와 Docker Compose를 이용한 빌드 및 테스트
- 지속적 통합 절차

## 2. 왜 이 내용을 배우는가

컨테이너가 실행 중이라고 해서 애플리케이션이 정상이라는 뜻은 아니다. 프로세스는 살아 있지만 API가 실패할 수 있고, 웹 컨테이너는 떠 있지만 의존하는 API나 DB가 준비되지 않았을 수 있다.

운영에서 중요한 것은 “컨테이너가 켜졌는가”가 아니라 “사용자 요청을 정상적으로 처리할 수 있는가”다. 6주차의 핵심은 컨테이너 실행 상태와 서비스 정상 상태를 구분하는 것이다.

## 3. 먼저 알아야 할 배경지식

- 프로세스 상태: 프로그램이 실행 중인지 여부
- 애플리케이션 상태: 실제 기능이 정상 동작하는지 여부
- REST API: HTTP 기반으로 자원을 요청하고 응답받는 인터페이스
- 의존성: 애플리케이션이 정상 동작하기 위해 필요한 외부 서비스
- 환경 변수: 실행 시점에 설정 값을 주입하는 방식
- CI: 코드 변경 후 빌드와 테스트를 자동으로 수행하는 절차

### 보충 설명

헬스 체크는 단순히 프로세스가 살아 있는지 보는 것이 아니다. 지금 요청을 처리할 수 있는 상태인지 확인해야 의미가 있다.

## 4. 강의자료 기반 개념 정리

### 4.1 Docker 헬스 체크

Docker 헬스 체크는 컨테이너 내부 애플리케이션이 실제로 정상 동작하는지 확인하는 상태 점검 로직이다.

기본적으로 Docker는 컨테이너의 주 프로세스가 살아 있는지 확인한다. 하지만 프로세스가 살아 있어도 API가 실패할 수 있다. 이때 Dockerfile에 `HEALTHCHECK`를 추가하면 컨테이너에 `healthy` 또는 `unhealthy` 상태를 부여할 수 있다.

```dockerfile
HEALTHCHECK CMD curl --fail http://localhost/health
```

내가 이해한 방식으로는 헬스 체크는 “서버가 켜져 있는가”가 아니라 “지금 요청을 처리할 수 있는가”를 묻는 검사다.

### 4.2 디펜던시 체크

디펜던시 체크는 애플리케이션이 의존하는 다른 서비스가 준비되었는지 확인하는 과정이다.

웹 애플리케이션이 API 서버를 필요로 한다면, 웹 컨테이너가 먼저 떠도 API가 준비되지 않으면 정상 동작하지 않는다. 따라서 시작 시점에 의존 서비스를 확인하는 로직이 필요하다.

### 4.3 커스텀 체크 유틸리티

간단한 상태 확인은 `curl`로 가능하지만, 복잡한 검증에는 별도 유틸리티가 필요할 수 있다. 강의자료에서는 .NET 기반의 체크 유틸리티를 사용하는 흐름이 등장한다.

### 4.4 Compose와 환경별 구성

Docker Compose는 여러 컨테이너를 함께 실행할 수 있지만, 개발/테스트/운영 환경마다 설정이 달라질 수 있다. 이때 여러 Compose 파일을 조합하거나 override 파일을 사용한다.

```bash
docker-compose -f docker-compose.yml -f docker-compose-test.yml up -d
```

### 4.5 CI와 Docker

CI는 코드 변경 후 빌드와 테스트를 자동으로 수행하는 흐름이다. Docker를 사용하면 빌드 환경을 이미지로 고정할 수 있어 재현성을 높일 수 있다.

## 5. 내가 몰랐을 가능성이 높은 개념

| 개념 | 왜 중요한가 | 이해 포인트 |
|---|---|---|
| 헬스 체크 | 운영 상태 판단 | 프로세스가 아니라 서비스 기능 정상 여부 확인 |
| 디펜던시 체크 | 시작 순서 문제 해결 | 의존 서비스 준비 여부 확인 |
| 커스텀 유틸리티 | 복잡한 검증 로직 | 셸 스크립트 한계를 보완 |
| 환경 변수 | 환경별 설정 관리 | 이미지 재빌드 없이 설정 주입 |
| 비밀 값 | 민감 정보 보호 | 이미지에 직접 포함하면 안 됨 |
| Compose override | 환경별 차이 분리 | 개발/테스트/운영 설정 구분 |
| 확장 필드 | YAML 중복 제거 | 반복 설정을 재사용 |
| CI 파이프라인 | 배포 품질 관리 | 빌드, 테스트, 산출물 생성 자동화 |

## 6. 실습하면서 내 것으로 만들 부분

6주차 실습은 명령어가 많지만 흐름은 분명하다.

```text
문제 상황 만들기
→ 컨테이너는 Up인데 API가 실패하는 상황 확인
→ Dockerfile에 HEALTHCHECK 추가
→ 새 이미지 빌드
→ 컨테이너 health 상태 확인
→ 의존성 체크 추가
→ Compose로 여러 컨테이너를 함께 실행
→ 환경별 Compose 설정 관리
```

### 6.1 헬스 체크 없는 API 컨테이너 확인

```bash
# API 컨테이너 실행
docker container run -d -p 8080:80 diamol/ch08-numbers-api

# API를 여러 번 호출한다.
curl http://localhost:8080/rng
curl http://localhost:8080/rng
curl http://localhost:8080/rng
curl http://localhost:8080/rng

# 컨테이너 상태 확인
docker container ls
```

이 실습의 포인트는 네 번째 호출에서 API가 실패해도 컨테이너 상태는 여전히 `Up`으로 보일 수 있다는 점이다. 즉, Docker의 기본 상태 확인만으로는 애플리케이션 장애를 잡지 못한다.

### 6.2 Dockerfile에 HEALTHCHECK 추가

```dockerfile
FROM diamol/dotnet-aspnet

ENTRYPOINT ["dotnet", "/app/Numbers.Api.dll"]
HEALTHCHECK CMD curl --fail http://localhost/health

WORKDIR /app
COPY --from=builder /out/ .
```

`HEALTHCHECK`는 컨테이너 내부에서 주기적으로 실행할 상태 점검 명령을 정의한다. 여기서는 `/health` 엔드포인트가 실패하면 컨테이너가 비정상 상태로 표시될 수 있다.

### 6.3 헬스 체크가 들어간 이미지 빌드와 확인

```bash
# 예제 코드 디렉터리로 이동
cd ./ch08/exercises/numbers

# v2 Dockerfile로 이미지 빌드
docker image build -t diamol/ch08-numbers-api:v2 -f ./numbersapi/Dockerfile.v2 .

# 가장 최근 컨테이너 상세 상태 확인
docker container inspect $(docker container ls --last 1 --format '{{.ID}}')

# v2 이미지로 컨테이너 실행
docker container run -d -p 8081:80 diamol/ch08-numbers-api:v2

# 상태 확인
docker container ls

# API 호출
curl http://localhost:8081/rng
curl http://localhost:8081/rng
curl http://localhost:8081/rng
curl http://localhost:8081/rng

# 시간이 지난 뒤 health 상태 확인
docker container ls
```

여기서 `inspect`는 단순 목록보다 훨씬 많은 정보를 보여준다. 헬스 체크 결과, 환경 변수, 네트워크, 마운트 등을 확인할 때 사용한다.

### 6.4 디펜던시 체크 흐름

```bash
# 실행 중인 모든 컨테이너 제거
docker container rm -f $(docker container ls -aq)

# 웹 컨테이너 실행
docker container run -d -p 8082:80 diamol/ch08-numbers-web

# 상태 확인
docker container ls
```

웹 컨테이너가 실행되어도 의존하는 API가 없으면 실제 서비스는 정상 동작하지 않을 수 있다. 그래서 강의자료에서는 의존성 체크가 추가된 Dockerfile 흐름으로 넘어간다.

```dockerfile
ENV RngApi:Url=http://numbers-api/rng

CMD curl --fail http://numbers-api/rng && \
    dotnet Numbers.Web.dll
```

이 구조는 애플리케이션 실행 전에 필요한 API가 응답하는지 먼저 확인한다.

### 6.5 커스텀 유틸리티와 health interval

```bash
# 기존 컨테이너 정리
docker container rm -f $(docker container ls -aq)

# 헬스 체크 주기를 짧게 지정해 실행
docker container run -d -p 8080:80 --health-interval 5s diamol/ch08-numbers-api:v3

# 상태 확인
docker container ls

# API 호출
curl http://localhost:8080/rng
curl http://localhost:8080/rng
curl http://localhost:8080/rng
curl http://localhost:8080/rng

# health 상태 확인
docker container ls
```

`--health-interval`은 헬스 체크를 얼마나 자주 수행할지 정한다. 실습에서는 빠르게 결과를 보기 위해 짧게 설정할 수 있다.

### 6.6 Compose로 여러 컨테이너 실행

```bash
# 예제 디렉터리로 이동
cd ./ch08/exercises/numbers

# 기존 컨테이너 정리
docker container rm -f $(docker container ls -aq)

# Compose로 애플리케이션 실행
docker-compose up -d

# 컨테이너 상태 확인
docker container ls

# 웹 컨테이너 로그 확인
docker container logs numbers-numbers-web-1
```

Compose를 사용하면 API, 웹, 네트워크, 환경 변수 등을 하나의 구성으로 묶어서 실행할 수 있다.

### 6.7 Compose 파일 조합과 환경별 실행

```bash
# 기본 Compose 파일과 override 파일을 합쳐 최종 설정 확인
docker-compose -f ./todo-list/docker-compose.yml -f ./todo-list/docker-compose-v2.yml config

# 테스트 프로젝트 이름으로 실행
docker-compose -f ./todo-list/docker-compose.yml -p todo-test up -d

# 실행 컨테이너 확인
docker container ls

# 특정 컨테이너의 포트 매핑 확인
docker container port todo-test_todo-web_1 80

# 환경별 Compose 파일 조합으로 실행
docker-compose -f ./numbers/docker-compose.yml -f ./numbers/docker-compose-test.yml up -d

# 정리
docker-compose down
```

`-f` 옵션은 Compose 파일을 여러 개 조합할 때 사용한다. 개발, 테스트, 운영 설정을 분리할 때 유용하다.

### 6.8 CI 빌드 흐름

```bash
# 인프라 컨테이너 실행
cd ./ch11/exercises/infrastructure
docker-compose -f docker-compose.yml -f docker-compose-linux.yml up -d

# 빌드용 Compose 실행
cd ./ch11/exercises
docker-compose -f docker-compose.yml -f docker-compose-build.yml build

# 이미지 라벨 확인
docker image inspect -f '{{.Config.Labels}}' diamol/ch11-numbers-api:v3-build-local

# pull까지 포함한 빌드
docker-compose -f docker-compose.yml -f docker-compose-build.yml build --pull

# 로컬 레지스트리 확인
curl http://registry.local:5000/v2/_catalog
curl http://registry.local:5000/v2/diamol/ch11-numbers-api/tags/list
curl http://registry.local:5000/v2/diamol/ch11-numbers-web/tags/list
```

이 흐름은 Docker가 단순 실행 도구가 아니라 빌드와 테스트, 배포 산출물 생성에도 쓰인다는 점을 보여준다.

### 6.9 명령어별로 정리

| 명령어 | 언제 쓰는가 | 내가 이해한 의미 |
|---|---|---|
| `curl http://localhost:8080/rng` | API 응답을 직접 확인할 때 | 애플리케이션 기능이 실제로 동작하는지 확인 |
| `HEALTHCHECK CMD ...` | Dockerfile에 상태 점검을 넣을 때 | 컨테이너에 health 상태를 부여 |
| `docker container inspect` | 컨테이너 상세 상태를 볼 때 | health 로그, 설정, 네트워크 확인 |
| `docker image build -f` | 특정 Dockerfile로 빌드할 때 | 여러 Dockerfile 중 원하는 파일 지정 |
| `docker container rm -f $(docker container ls -aq)` | 실습 환경을 초기화할 때 | 모든 컨테이너 강제 삭제 |
| `docker-compose up -d` | Compose 앱을 백그라운드 실행할 때 | 여러 컨테이너를 한 번에 실행 |
| `docker-compose -f ... config` | Compose 파일 병합 결과를 확인할 때 | 최종 설정이 어떻게 해석되는지 확인 |
| `docker container port` | 포트 매핑을 확인할 때 | 컨테이너 포트가 호스트 어디에 연결됐는지 확인 |
| `docker-compose build --pull` | 최신 베이스 이미지를 당겨 빌드할 때 | 빌드 전 의존 이미지를 갱신 |

### 6.10 헷갈렸던 지점

`docker-compose depends_on` 같은 설정은 컨테이너 시작 순서를 어느 정도 제어할 수 있지만, 서비스가 실제로 준비되었는지까지 완벽하게 보장하는 것은 아니다. 그래서 readiness나 dependency check가 따로 필요하다.

또한 헬스 체크는 “컨테이너를 자동으로 고쳐주는 기능”이 아니다. 상태를 드러내는 기능이고, 그 상태를 보고 재시작하거나 교체하는 정책은 별도로 연결되어야 한다.

## 7. 보충 설명

- 자료 기반 내용: Docker 헬스 체크, 디펜던시 체크, 커스텀 유틸리티, Compose 환경 구성, 환경 변수와 비밀 값, 확장 필드, CI 절차
- 보충 설명: 헬스 체크는 Kubernetes의 liveness/readiness probe와도 연결된다. 다음 주차 Kubernetes를 이해하기 위한 중요한 선행 개념이다.
- 확실하지 않음: 실제 실습에서 사용한 Compose 파일 전체와 테스트 결과 로그는 업로드된 강의자료만으로는 확실하지 않음.

## 8. 한 줄 요약

운영 가능한 컨테이너 서비스는 단순히 실행되는 것이 아니라, 상태를 검증하고 의존성을 확인하며 실패를 드러낼 수 있어야 한다.

## 9. 추가로 공부할 키워드

- Docker HEALTHCHECK
- Dependency Check
- Readiness
- Liveness
- Docker Compose Override
- Environment Variable
- Secret Management
- CI Pipeline
