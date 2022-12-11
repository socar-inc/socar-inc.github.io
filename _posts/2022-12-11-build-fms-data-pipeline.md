---
layout: post
title:  "빠르고 정확하게, 신사업 FMS 데이터 파이프라인 구축기"
subtitle: 빠르고 정확하게, 신사업 FMS 데이터 파이프라인 구축기
date: 2022-12-11 09:00:00 +0900
category: data
background : "/img/advanced-airflow-for-databiz/background.jpg"
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

- 개인/팀 소개
- 예상독자
- 글 전체 개요

## 1. 쏘카의 FMS 데이터 파이프라인

### 신사업 FMS 서비스 소개
### 실시간 파이프라인에서 중요한 것
- Real Time Response
- Scalability
- Data Reliablity
### IoT 시계열 데이터 특징
### 전체 파이프라인 훑어보기

## 3. 실시간 데이터 처리와 적재를 한 번에, Kafka Sink Connector 개발하기

### Kafka Connector란?
### Kafka Connector 동작 원리 및 장단점
### DynamoDB Sink Connector 요구 사항 및 구현
Exactly once
### Custom S3 Sink Connector 요구 사항 및 구현
Exactly once
### Kafka Connect 모니터링하기
Prometheus + Grafana 확인

## 4. 비정형 데이터 쿼리를 위한 미들웨어, Lambda 함수 개발하기 

### 배경
- 프로토콜이 많다 보니 토픽을 대분류로 분리해서 저장
- PoC기간에 스키마 레지스트리 도입이 힘들었음
- redshift를 활용해서 쿼리하기로 결정 (athena는 일부 window function 지원이 안됨)
    - 파티셔닝된 parquet 들
    - glue table
### 요구사항
### 람다 함수 주요 동작방식
### 람다 모니터링하기
### fallback 처리

---

## 5. 견고한 파이프라인을 위한 통합 테스트 환경 구축하기

### 테스트 환경이 필요한 이유
### Docker Compose를 통한 E2E 테스트 환경 구축
### Github Action을 통한 CI/CD 파이프라인에 테스트 자동화
### 메시지 시뮬레이터 구현
### 실 데이터 중심의 부하 테스트 진행

## 6. 데이터 신뢰성을 위한 정합성과 무결성 검증하기

### 데이터 정합성/무결성이 중요한 이유
### 검사할 대상 정의하기
- 크게 3가지…
### 정합성/무결성 검사 진행하기
### 검사 결과 모니터링하기

## 7. 하나 더 추가할까….?

## 8. 마무리
### 남은과제
### 결론