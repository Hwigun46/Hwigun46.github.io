---
layout: post
title: "클라우드활용 2주차 학습 정리"
date: 2026-03-19
categories: [hyu]
tags: [cloud-utilization, assignment]
---

# 클라우드활용 2주차: AWS Management Console과 AWS CLI 기초

## 1. 이번 주차에서 다룬 내용

이번 주차에서는 AWS를 실제로 조작하는 운영 환경을 학습했다. 1주차가 클라우드 개념과 인프라 구조였다면, 2주차는 AWS Management Console과 AWS CLI를 통해 리소스를 확인하고 조작하는 입구를 익히는 주차다.

이번 주차에서 다룬 내용은 다음과 같다.

- AWS Management Console의 역할
- Root User와 IAM User 로그인 방식
- 콘솔 기본 UI 탐색
- 리전 변경과 대시보드 확인
- EC2, S3, IAM 서비스 탐색 흐름
- AWS CLI와 CloudShell의 차이
- S3, EC2, IAM 조회 명령어
- AWS CLI 설정 시 Access Key 관리 주의사항

## 2. Background

- GUI:
  그래픽 기반 사용자 인터페이스다. AWS Management Console은 GUI 방식으로 AWS 리소스를 조작한다.

- CLI:
  명령어 기반 인터페이스다. AWS CLI는 터미널 명령어로 AWS 리소스를 조회하고 제어한다.

- Root User:
  AWS 계정 생성 시 만들어지는 최상위 계정이다. 강력한 권한을 가지므로 일상 작업에 계속 사용하는 것은 위험하다.

- IAM User:
  AWS 계정 내부에서 생성되는 개별 사용자다. 필요한 권한만 부여해서 사용하는 것이 원칙이다.

- CloudShell:
  AWS 콘솔 안에서 바로 사용할 수 있는 브라우저 기반 CLI 환경이다.

### 보충 설명

콘솔과 CLI는 둘 중 하나만 쓰는 관계가 아니다. 콘솔은 구조를 눈으로 확인하기 좋고, CLI는 반복 작업과 자동화에 적합하다. 처음에는 콘솔로 메뉴와 리소스 위치를 익히고, 이후 CLI로 같은 작업을 조회하거나 자동화하는 방식이 현실적이다.

## 3. 개념 정리

### 3.1 AWS Management Console

AWS Management Console은 웹 브라우저에서 AWS 리소스를 시각적으로 관리할 수 있는 도구다. EC2, S3, RDS, IAM 같은 서비스를 검색하고, 리소스를 생성·삭제·설정·모니터링할 수 있다.

콘솔의 장점은 처음 AWS를 배우는 사람이 구조를 눈으로 확인하기 좋다는 점이다. 예를 들어 EC2 대시보드에서는 인스턴스 상태를 확인하고, S3 대시보드에서는 버킷 목록과 접근 설정을 볼 수 있다.

### 3.2 Root User와 IAM User

| 구분 | Root User | IAM User |
|---|---|---|
| 생성 시점 | AWS 계정 생성 시 자동 생성 | AWS 계정 내부에서 별도 생성 |
| 로그인 정보 | 이메일 주소와 비밀번호 | Account ID, IAM Username, Password |
| 권한 수준 | 최상위 권한 | 부여받은 권한만 사용 |
| 사용 용도 | 계정 수준의 특수 작업 | 일반적인 운영 작업 |
| 보안 관점 | 계속 사용하면 위험 | 최소 권한으로 운영 가능 |

강의자료에서는 Root User와 IAM User의 로그인 방식 차이를 다룬다. 이 차이는 단순 로그인 절차가 아니라 보안 운영 방식과 연결된다. Root User는 권한이 너무 크기 때문에 실무에서는 IAM User나 Role 중심으로 운영하는 것이 더 안전하다.

### 3.3 리전 변경과 대시보드

AWS 리소스는 리전에 종속되는 경우가 많다. 서울 리전에서 만든 EC2 인스턴스는 다른 리전에서 조회되지 않는다. 따라서 실습할 때 리전이 다르면 “분명 만들었는데 안 보이는” 문제가 생길 수 있다.

| 대시보드 | 확인할 수 있는 내용 |
|---|---|
| EC2 대시보드 | 인스턴스 상태, 시작/중지, 모니터링 지표 |
| S3 대시보드 | 버킷 목록, 버전 관리, 접근 설정 |
| IAM 대시보드 | 사용자, 그룹, 역할, 정책 현황 |
| Billing 대시보드 | 월별 사용량, 서비스별 비용, 비용 추세 |

처음 AWS를 사용할 때는 리전 표시 위치를 먼저 확인하는 습관이 필요하다.

### 3.4 AWS CLI와 CloudShell

AWS CLI는 AWS 리소스를 명령어로 제어하는 도구다. 콘솔이 클릭 기반이라면 CLI는 명령어 기반이다.

CloudShell은 별도 설치 없이 콘솔에서 사용할 수 있는 CLI 전용 터미널이다. 로컬 PC에 Access Key를 저장하지 않아도 되기 때문에, 초반 실습에서는 CloudShell을 사용하는 것이 더 안전하다.

| 구분 | AWS CLI 로컬 설치 | AWS CloudShell |
|---|---|---|
| 실행 위치 | 개인 PC 터미널 | AWS 콘솔 브라우저 안 |
| 설치 필요 | 필요 | 불필요 |
| 키 관리 | Access Key 설정 필요 | 콘솔 인증 기반 사용 |
| 장점 | 로컬 자동화에 적합 | 초기 실습에 안전하고 간편 |
| 주의점 | 키 유출 위험 | 콘솔 접속 권한 필요 |

## 4. 내가 몰랐을 가능성이 높은 개념

| 개념 | 왜 중요한가 | 이해 포인트 |
|---|---|---|
| 리전별 리소스 조회 | 실습 중 리소스가 안 보이는 원인 | 만든 리전과 보는 리전이 같아야 함 |
| Root User | 계정 전체 보안과 직결 | 일반 작업에 계속 쓰면 위험 |
| IAM User | 권한 분리의 기본 단위 | 필요한 권한만 부여해야 함 |
| CloudShell | CLI 실습의 안전한 출발점 | 로컬 키 저장 없이 명령어 사용 가능 |
| Access Key | CLI 인증 정보 | 유출되면 계정 비용과 보안 사고로 이어짐 |
| 대시보드 | 리소스 상태를 빠르게 파악 | 서비스별 운영 현황을 요약해서 보여줌 |

## 5. 실습하면서 내 것으로 만들 부분

이번 주차의 실습 흐름은 콘솔로 구조를 확인하고, CLI로 같은 자원을 조회하는 방식으로 이해하면 된다.

```text
콘솔 로그인
→ 리전 확인
→ EC2/S3/IAM 서비스 탐색
→ 리소스 상태 확인
→ CloudShell 실행
→ CLI 명령어로 같은 리소스 조회
→ 필요 없는 리소스 삭제 또는 종료
```

### 5.1 파이프라인

1. AWS Management Console에 로그인한다.
2. 현재 리전을 확인하고 필요한 리전으로 변경한다.
3. EC2 대시보드에서 인스턴스 상태와 정보를 확인한다.
4. S3 대시보드에서 버킷 목록과 설정을 확인한다.
5. IAM 대시보드에서 사용자, 그룹, 역할, 정책을 확인한다.
6. CloudShell을 열고 CLI 명령어로 리소스를 조회한다.
7. 실습 후 생성한 리소스가 있다면 삭제 또는 종료한다.

### 5.2 명령어 또는 코드 흐름

```bash
# S3 버킷 목록 조회
aws s3 ls

# 특정 버킷 내부 객체 목록 조회
aws s3 ls s3://버킷이름

# 특정 버킷의 하위 경로 조회
aws s3 ls s3://버킷이름/folder-name/

# 버킷의 리전 조회
aws s3api get-bucket-location --bucket 버킷이름

# EC2 인스턴스 정보 조회
aws ec2 describe-instances

# EC2 인스턴스 ID만 조회
aws ec2 describe-instances --query "Reservations[*].Instances[*].InstanceId" --output text

# 특정 리전의 EC2 인스턴스 조회
aws ec2 describe-instances --region ap-northeast-2

# IAM 사용자 목록 조회
aws iam list-users

# IAM 그룹 목록 조회
aws iam list-groups

# IAM 역할 목록 조회
aws iam list-roles

# IAM 정책 목록 조회
aws iam list-policies
```

### 5.3 명령어별로 정리

| 명령어 | 언제 쓰는가 | 내가 이해한 의미 |
|---|---|---|
| `aws s3 ls` | S3 버킷 목록을 볼 때 | 현재 계정에서 접근 가능한 S3 버킷 확인 |
| `aws s3 ls s3://버킷이름` | 특정 버킷 안의 객체를 볼 때 | 버킷 내부 파일 목록 확인 |
| `aws s3api get-bucket-location --bucket 버킷이름` | 버킷이 어느 리전에 있는지 확인할 때 | S3 버킷 위치 확인 |
| `aws ec2 describe-instances` | EC2 인스턴스 정보를 볼 때 | 인스턴스 상태와 설정 전체 조회 |
| `aws ec2 describe-instances --region ap-northeast-2` | 서울 리전 EC2를 확인할 때 | 리전을 명시해서 조회 |
| `aws iam list-users` | IAM 사용자 목록을 볼 때 | 계정 내 사용자 확인 |
| `aws iam list-groups` | IAM 그룹을 볼 때 | 권한 묶음 단위 확인 |
| `aws iam list-roles` | IAM 역할을 볼 때 | 임시 권한을 맡을 수 있는 Role 확인 |
| `aws iam list-policies` | IAM 정책을 볼 때 | 권한 정책 목록 확인 |

### 5.4 헷갈렸던 지점

- 콘솔에서 안 보이는 리소스는 삭제된 것이 아니라 다른 리전에 있을 수 있다.
- Root User와 IAM User는 로그인 방식뿐 아니라 권한 범위가 다르다.
- CloudShell은 AWS CLI를 브라우저에서 쓰는 것이며, 콘솔과 별개 서비스처럼 생각하면 헷갈린다.
- CLI 명령어는 기본 리전을 기준으로 동작하므로, 필요한 경우 `--region`을 명시해야 한다.
- Access Key를 로컬이나 공용 PC에 저장하면 보안 위험이 크다.

| 개념 A | 개념 B | 차이점 |
|---|---|---|
| Console | CLI | Console은 시각적 조작, CLI는 명령어 기반 조작 |
| Root User | IAM User | Root는 계정 최상위 권한, IAM User는 부여받은 권한만 사용 |
| AWS CLI | CloudShell | CLI는 도구, CloudShell은 브라우저에서 실행되는 CLI 환경 |
| 대시보드 | 리소스 상세 화면 | 대시보드는 요약, 상세 화면은 개별 설정 확인 |

### 5.5 나중에 다시 볼 포인트

- 현재 콘솔에서 선택된 리전을 확인할 수 있는가?
- EC2, S3, IAM 대시보드에서 각각 무엇을 확인해야 하는가?
- CloudShell에서 S3, EC2, IAM 조회 명령어를 실행할 수 있는가?
- Access Key를 로컬에 저장할 때 어떤 위험이 있는지 설명할 수 있는가?

## 6. 보충 설명

* 자료 기반 내용:
  * AWS Management Console은 웹 기반 GUI로 AWS 리소스를 시각적으로 관리하는 도구다.
  * 리전 선택에 따라 생성·조회되는 리소스가 달라진다.
  * AWS CLI는 명령어 기반으로 AWS 리소스를 제어할 수 있는 도구다.
  * CloudShell은 브라우저에서 실행할 수 있는 AWS CLI 전용 터미널 환경이다.

* 보충 설명:
  * 실습 초반에는 CloudShell을 사용하는 편이 키 관리 측면에서 더 안전하다.
  * 콘솔에서 구조를 익힌 뒤 CLI로 같은 리소스를 조회하면 AWS 구조를 더 빨리 이해할 수 있다.

* 확실하지 않음:
  * 2주차 자료만으로는 실제 계정 생성 과정 전체나 MFA 설정 절차는 확인되지 않는다.

## 7. 한 줄 요약

AWS 운영의 출발점은 콘솔로 리소스 구조를 눈으로 익히고, CloudShell과 CLI로 같은 리소스를 명령어 기반으로 조회하는 것이다.

## 8. 추가로 공부할 키워드

* AWS Management Console
* AWS CLI
* AWS CloudShell
* Root User
* IAM User
* Access Key
* Region
* Billing Dashboard
