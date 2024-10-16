# CI란?

## 정의

CI : Continuous Integration, 지속적인 통합

## 필요한 이유/원하는 것

자동으로 빌드 및 테스트
- 하나의 동작이 발생했을 때, 수동으로 빌드하지 않고 자동으로 빌드 및 테스트를 실행해준다.

## Backend CI 스크립트

> [backend_ci.yml](https://github.com/woowacourse-teams/2024-code-zap/blob/main/.github/workflows/backend_ci.yml)


> 해당 스크립트는 Pull request base 브랜치를  dev/be, main 브랜치로 할 때 실행된다.

### 1. 체크아웃

- 레포지토리를 CI 환경으로 가져온다. 
- PR의 대상이 되는 브랜치를 체크아웃하여 작업을 진행할 수 있도록 설정한다.

### 2. MySQL 실행

- MySQL을 설정된 포트 및 설정값으로 실행한다. 
- MySQL 버전, 데이터베이스 이름, 포트, 루트 비밀번호 등은 사전에 보안상의 이유로 `secrets`에 저장된 값을 활용한다.

### 3. DB 설정 파일 가져오기

- 보안을 위해 DB 설정 정보는 `secrets`에 숨겨두었으며, 이를 활용해 `application-db.yml` 파일을 생성한다. 
- 이 파일은 애플리케이션이 데이터베이스에 접근할 때 필요한 설정을 담고 있다.

### 4. Gradle 캐싱

- 동일한 CI 동작이 반복될 때 이전에 다운로드한 Gradle 종속성 및 빌드 결과물을 캐싱하여 빌드 시간을 단축한다. 
- 캐싱된 결과물이 있으면 이를 활용하여 빠르게 빌드가 진행된다.

### 5. JDK 17 설치

- 프로젝트가 Java 17을 사용하도록 JDK 17을 설치한다. 
- 이를 통해 애플리케이션이 Java 17 환경에서 정상적으로 빌드 및 실행될 수 있도록 보장한다.

### 6. 테스트 실행

- Gradle을 사용해 프로젝트의 테스트 코드를 실행한다. 
- 이 과정에서 MySQL 데이터베이스에 대한 통합 테스트가 포함될 수 있으며, 프로젝트 내에서 정의된 모든 테스트가 수행된다.

---

<br>

# CD란?

## 정의

CD : Continuous Deployment, 지속적 배포

## 필요한 이유/원하는 것

개발자의 변경 사항을 레포지토리에서 프로덕션으로 릴리스하는 것을 자동화하여 고객이 사용할 수 있도록 한다.

## Backend CD 스크립트

> [backend_cd.yml](https://github.com/woowacourse-teams/2024-code-zap/blob/main/.github/workflows/backend_cd.yml)

> 해당 스크립트는 dev/be, main 브랜치가 업데이트 될 때 실행된다.

> 스크립트는 총 3개의 jab을 가지고 있다.
> 1. build : 배포를 위해 현재 데이터를 빌드해 jar 파일로 저장한다.
> 2. deploy_develop:  개발 서버에 배포한다.
> 3. deploy_production: 브랜치가 main이면 운영 서버에 배포한다.

<br>

### jab 1 : build

#### 1. 체크아웃

- 레포지토리를 CI 환경으로 가져온다. 
- PR의 대상이 되는 브랜치를 체크아웃하여 작업을 진행할 수 있도록 설정한다.

#### 2. JDK 17 설치

- 프로젝트가 Java 17을 사용하도록 JDK 17을 설치한다. 
- 이를 통해 애플리케이션이 Java 17 환경에서 정상적으로 빌드 및 실행될 수 있도록 보장한다.

#### 3. Gradle 캐싱

- 동일한 CI 동작이 반복될 때 이전에 다운로드한 Gradle 종속성 및 빌드 결과물을 캐싱하여 빌드 시간을 단축한다. 
- 캐싱된 결과물이 있으면 이를 활용하여 빠르게 빌드가 진행된다.

#### 4. bootJar로 jar 파일 생성

- Gradle을 사용해 `bootJar` 작업을 수행하여 애플리케이션의 실행 가능한 JAR 파일을 생성한다.

#### 5. Artifact 업로드

- 빌드 결과물(JAR 파일)을 저장해 이후 배포 단계에서 사용하기 위한 Artifact로 업로드한다.

<br>

### jab 2 : deploy_develop

- `build` 작업이 완료되어야 실행된다.
- 브랜치가 `dev/be`인 경우에만 실행되며, 이 작업은 개발 서버로 배포하는 용도이다.
- self-hosted runner 중 `spring`과 `develop` 라벨이 있는 runner를 사용한다.

#### 1. Artifact 다운로드

- 이전 `build` 작업에서 저장된 Artifact를 동일한 CI 환경에서 다운로드하여 배포에 사용한다.

#### 2. 배포 스크립트 실행

- 서버 내부에 미리 정의된 배포 스크립트를 실행하여 애플리케이션을 개발 환경에 배포한다.

<br>

### jab 2 : deploy_production

- `build` 작업이 완료되어야 실행된다.
- 브랜치가 `main`인 경우에만 실행되며, 이는 프로덕션 서버로의 배포를 담당한다.
- self-hosted runner 중 `prod_a`와 `prod_b` 라벨이 있는 runner를 사용한다.

#### 1. Artifact 다운로드

- `build` 작업에서 저장된 Artifact를 다운로드하여 배포에 사용한다.

#### 2. 배포 스크립트 실행

- 서버에 정의된 배포 스크립트를 실행하여 애플리케이션을 프로덕션 환경에 배포한다.

<br>

--- 

# 추가 설명

### Secrets 추가 방법

![제목 없음](https://private-user-images.githubusercontent.com/109158497/350897006-2a897363-3b36-48b7-b007-fbe08dc3f7bb.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3MjkxMDE2NjMsIm5iZiI6MTcyOTEwMTM2MywicGF0aCI6Ii8xMDkxNTg0OTcvMzUwODk3MDA2LTJhODk3MzYzLTNiMzYtNDhiNy1iMDA3LWZiZTA4ZGMzZjdiYi5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBVkNPRFlMU0E1M1BRSzRaQSUyRjIwMjQxMDE2JTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDI0MTAxNlQxNzU2MDNaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT01MmY3YjgxNjc1OWVkNDM4MGNhZmFmNWJjYzc1ZjY5YWYzZDM2NmVmOTM3MDM5NzE4YjA1ZWIxMDMzNDA5MTdlJlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCJ9.5WiKpvmhxyIj_E1nHvCykc4-OWkZTxR1X9T2VkA9xuI)

<br>

### Artifact

Artifact는 빌드 프로세스나 CI/CD 파이프라인에서 생성되는 결과물로, 주로 배포 또는 테스트에 필요한 파일들을 말합니다. 
이는 애플리케이션의 실행 파일, JAR 파일, Docker 이미지, 로그 파일, 테스트 결과 등이 포함될 수 있습니다.

GitHub Actions에서는 `Artifact`를 통해 빌드된 파일을 저장하고, 이후 작업(job)에서 다시 사용할 수 있습니다. 

<br>

### Self-hosted runner

Self-hosted runner는 GitHub Actions에서 제공하는 CI/CD 기능을 사용하기 위해 사용자가 직접 관리하는 서버 또는 가상 머신입니다. 
GitHub에서 제공하는 기본적인 호스팅 환경이 아닌, 사용자가 자신의 환경에서 직접 실행할 수 있도록 설정한 것입니다. 이를 통해 더 많은 제어권과 유연성을 가질 수 있습니다.

#### 사용 예시

- 특정 언어 또는 프레임워크에 최적화된 소프트웨어 환경을 필요로 하는 경우 (예: Java, .NET 등).
- 내부 보안 정책에 따라 외부 서버에 접근할 수 없는 경우.
- 고유한 하드웨어 요구 사항이 있는 빌드나 테스트 작업을 수행해야 하는 경우.

<br>

### Github Actions에 Self-hosted Runner 등록

#### 1) Settings - Actions - Runners 이동 및 `New self-hosted runner` 클릭

![runner_step1](https://github.com/user-attachments/assets/dd532dd4-e2f0-4dd8-a13d-d498966b42b7)

<br>

#### 2) runner로 등록한 자원에 맞게 환경 설정

![runner_step2](https://github.com/user-attachments/assets/9368d7c5-7822-4811-b67f-cd86dd57f833)


> 우리는 Linux의 armx64 을 사용한다

<br>

#### 3) runner로 등록한 자원으로 이동 후 명령어 실행

##### Download

- 폴더 생성
```
mkdir actions-runner && cd actions-runner
```
- 가장 최신의 Runner 패키지 다운로드
```
curl -o actions-runner-linux-arm64-2.317.0.tar.gz -L https://github.com/actions/runner/releases/download/v2.317.0/actions-runner-linux-arm64-2.317.0.tar.gz
```
- 다운로드 받은 압축 파일 압축 해제
```
tar xzf ./actions-runner-linux-arm64-2.317.0.tar.gz
```

##### Configure

- 저장소 연결
```
./config.sh --url [repository url] --token [github token]
```
- runner 프로그램 실행
```
nohup ./run.sh &
```

##### Using your self-hosted runner

- job 실행 환경 설정
```
runs-on: self-hosted
```

