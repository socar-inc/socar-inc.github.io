---
layout: post
title: "빠르고 정확하게, 신사업 FMS 데이터 파이프라인 구축기"
subtitle: 빠르고 정확하게, 신사업 FMS 데이터 파이프라인 구축기
date: 2022-12-11 09:00:00 +0900
category: data
background: "/img/advanced-airflow-for-databiz/background.jpg"
author: grab
comments: true
tags:
    - data
    - data engineering
    - iot streaming
    - data platform
    - kafka connect
    - aws
---

0. 들어가며

안녕하세요. 데이터 플랫폼 팀의 그랩입니다.

데이터 플랫폼팀은 “쏘카 내부의 데이터 이용자가 비즈니스에 임팩트를 낼 수 있도록 소프트웨어 엔지니어링에 기반하여 문제를 해결합니다”라는 미션을 기반으로 인프라, 데이터 파이프라인 개발, 운영, 모니터링, 데이터 애플리케이션 개발, MLOps 등의 업무를 맡고 있습니다.
팀 구성원들은 모두가 소프트웨어 엔지니어라는 사명감을 가지고 개발뿐만 아니라 Ops에 대한 이해와 책임감을 가지고 업무에 임하고 있습니다.

본 글에서는 쏘카의 신사업 FMS 서비스의 IoT 데이터 파이프라인에 대해 소개하려고 합니다. 차량 IoT 디바이스에서 생성되는 서비스에서 제공되기까지 어떤 파이프라인을 거쳤는지 하나하나 설명드리려고 합니다.

다음과 같은 분들이 읽으면 좋습니다.

-   실시간 데이터 파이프라인에 관심이 있는 소프트웨어 엔지니어
-   데이터 파이프라인에 테스트를 도입하고 싶은 소프트웨어 엔지니어
-   AWS 기반의 데이터 엔지니어링 환경 구축에 관심이 있는 소프트웨어 엔지니어
-   Kafka Connect를 활용한 메시지 소비에 관심이 있는 소프트웨어 엔지니어
-   쏘카의 데이터 엔지니어가 무슨 일을 하는지 궁금한 모든 이들

목차는 아래와 같습니다.  
....

<br/>

## 1. 쏘카의 FMS 데이터 파이프라인

### 신사업 FMS 서비스 소개

-   신사업 소개

### 차량 IoT 데이터의 특징

-   주기적으로 보고하는 유형과 제어/응답하는 유형이 있음
-   주기적으로 보고하는 유형은 보통 배치로 묶어서 전송한다. e.g., kinematic 데이터
-   차량 운행 관련한 다양한 메시지 프로토콜들이 존재한다
-

### 전체 파이프라인 훑어보기

## 2. 실시간 데이터 처리와 적재를 한 번에, Kafka Sink Connector 개발하기

### 배치와 실시간

-   배치와 실시간
-   실시간
    -   Real Time Response
    -   배치와 실시간 차이
    -   실시간 파이프라인의 경우, 빠른 시간내에 데이터를 가공/적재하여 사용자에게 보여줘야 한다. 예시로...
    -   보통 기업에서는 배치로 데이터를 조회하다가, 점점 최신에 가까운 데이터를 요구하면서 실시간 환경의 니즈가 생기게 된다
    -   쏘카의 FMS 비즈니스에서 차량 관제는 주요 기능으로, 이를 위해선 실시간으로 차량에서 발생하는 데이터를 수집해서 빠르게 서비스 단에서 활용할 수 있도록 해야 합니다.

### Kafka Connector 란?

-   Kafka 란
-   Kafka Consumer 란
-   Kafka Connect 란
    -   Consumer를 한단계 추상화하여 제공한 프레임워크
-   Source/Sink Connector
-   Kafka Connector 장점

### Kafka Connector 동작 원리 및 장단점

...

### DynamoDB Sink Connector 요구 사항 및 구현

-   DynamoDB Connector는 오픈소스로 따로 없고 유료 라이센스가 있었음
-   구현 코드

### Custom S3 Sink Connector 요구 사항 및 구현

-   DynamoDB와 유사하게 measurements를 다시 풀어서 제공해줘야 함
-   Kotlin mutli module을 활용
-   S3 Sink Connector를 상속받아 구현했음
-   구현 코드

### Kafka Connect 배포 및 운영

-   Kafka Connect 이미지에 Connector를 jar 형태로 마운트시켜준다
-   Kafka Connect + k8s Helm Chart를 활요앻

### Kafka Connect 모니터링하기

-   Prometheus + Grafana 확인
-   주요하게 보면 좋은 Metric들

## 3. 비정형 데이터를 Redshift에서 쉽게 조회하기까지,

### 배치 분석 플랫폼 훑어보기

### Redshift Spectrum이란

-   redshift를 활용해서 쿼리하기로 결정 (athena는 일부 window function 지원이 안됨)
    -   파티셔닝된 parquet 들
    -   glue table

### 요구사항

-   프로토콜이 많다 보니 토픽을 대분류로 분리해서 저장
-   PoC기간에 스키마 레지스트리 도입이 힘들었음

### 람다 함수 주요 동작방식

### 람다 모니터링하기

-   Grafana에서 확인
-   문제 발생시 Alert가 오도록

### fallback 처리

-   SQS 활용
-   ***

## 4. 견고한 파이프라인을 위한 통합 테스트 환경 구축하기

### 데이터 파이프라인에 테스트가 필요한 이유

-   데이터 파이프라인의 구성요소는 각각 SPoF가 되기 쉽다.
-   Input/Output이 명확하여 테스트 결과를 정확하게 파악할 수 있다

### Docker Compose를 통한 E2E 테스트 환경 구축

-   Docker-Compose를 통해 데이터 파이프라인을 구성하는 주요 요소들을 도커 컨테이너로 전부 실행
-   소스코드
-   localhost를 통해 테스트할 컴포넌트는 접근이 가능

### Github Action을 통한 CI/CD 파이프라인에 테스트 자동화

-   Github Action이란?
-   Github Action의 Checkout을 활용해 Docker Compose 백그라운드로 실행하기
    -   Github Action은 하나의 Job이 하나의 가상환경에서 돌아감
-   Pull Request, Deploy하기 전에 E2E로 검사

## 5. 시뮬레이터를 활용한 실 데이터 기반 부하 테스트

### 실 데이터 기반의 메시지 시뮬레이터

-   MVP 서비스 런칭 전까지 데이터를 처리/저장하는 소프트웨어는 계속해서 데이터가 흘러야 했음
-   메시지 시뮬레이터를 구현해 실데이터 기반으로 메시지를 전송할 수 있도록 제공함
-   병렬 처리, 데이터 번형 전송이 가능하도록 구현

### 부하 테스트

-   부하 테스트 계획 세우기
-   주요하게 확인한 지표
-   부하테스트 진행
    -   차량 데이터 설정
    -   각 프로토콜 메시지를 병렬로 전송
-   부하테스트 결과 정리

## 6. 데이터 신뢰성을 위한 정합성과 무결성 검증하기

### 데이터 정합성/무결성이란?

-   정합성
-   무결성
-   중요한 이유

### 검사할 대상 정의하기

-   유형
    -   원본 데이터에 대한 차량 데이터 이상 현상 검사
    -   마트 테이블의 정합성 검사
    -   마트 테이블의 무결성 검사
-   검사지 만들기

### 정합성/무결성 검사 모니터링 구현하기

-   Airflow 활용
-   프로세스
    -   마트 테이블 무결성 검사는 마트 생성 Dag에서 진행
    -   원본 데이터 이상 여부 검사, 마트 테이블 정합성 검사는 하나의 Dag에서 진행
-   Dag 구현 코드

### 검사 결과 모니터링하기

    -   Grafana를 통한 대시보드에서 확인
    -   이상이 있는 경우 Slack Alert를 통해 확인

## 6. 마무리

### 남은과제

### 결론
