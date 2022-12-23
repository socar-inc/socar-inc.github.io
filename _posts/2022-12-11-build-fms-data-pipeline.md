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

## 0. 들어가며 (heading 삭제 예정)

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

분량 관계상 생략하는 부분은 다음과 같습니다.

-   Kafka에 대한 기본적인 설명
-   주요 컴포넌트들의 상세한 구현 설명 및 코드

글을 읽으시면서 궁금한 점들이 있다면 편하게 질문 남겨주시면, 확인 후 답변드리겠습니다.

목차는 아래와 같습니다.  
....

<br/>

## 1. FMS 데이터 파이프라인 소개

### FMS 서비스 소개

FMS는 Fleet Management System의 약자로, 차량에 통신이 가능한 단말을 부착해 구동되는 시스템입니다.  
...내용추가 필요...  
FMS 서비스를 통해 고객사는 수집/분석된 차량 데이터를 실시간으로 확인하고 대응이 가능합니다.

올해 2022년 7월 부터 PoC 개발을 시작으로, 현재 주요 고객사들을 대상으로 PoC 서비스를 성공적으로 런칭하여 운영중에 있습니다.

### 배치와 스트리밍

![batch-stream.png](/img/build-fms-data-pipeline/batch-stream.png)_이미지 저작권 확인 후 교체 필요(직접 만들까?)_

보통 비즈니스/분석 요구사항에 맞게 데이터 파이프라인을 구축하다 보면 배치와 스트리밍에 대한 고민을 자연스럽게 하게 됩니다.

**배치**는 특정 시간 범위의 데이터를 일괄로 처리하는 기술을 뜻합니다. 보통 데이터베이스의 데이터를 시간대에 맞춰 가져와서 처리하면 ‘배치’라고 많이 이야기 합니다. 배치로 처리하기 위해선 보통 이를 자동화하기 위한 툴(Airflow, Spring batch 등)을 도입하게 됩니다. 배치는 실시간으로 집계를 하지 않는 경우에 일반적으로 데이터를 처리하는 방식을 의미하며, 많은 비즈니스의 경우 배치 환경을 기본으로 도입하게 됩니다.

[참고]
현재 데이터 플랫폼 팀에서도 쏘카의 데이터 배치 환경을 구축하기 위해 GCP 환경 내에서 Airflow, Bigquery 등을 운영하고 있습니다 (자세한 내용은 [여기](https://tech.socarcorp.kr/data/2022/11/09/advanced-airflow-for-databiz.html)를 참고해주세요)

**스트리밍**은 생성되는 데이터를 실시간으로 처리하는 기술을 뜻합니다. 실시간으로 생성되는 데이터(메시지)를 일시적으로 저장하는 데이터베이스로 메시지 큐를 많이 활용합니다. 이는 대표적으로 Kafka, Redis, RabitMQ등의 오픈소스와 클라우드 서비스인 Kinesis, SQS, PubSub등이 있습니다.

보통 기업에서는 배치로 데이터를 조회하다가, 점점 최신에 가까운 데이터를 요구하면서 실시간 환경의 니즈가 생깁니다.
쏘카의 FMS 서비스에서 차량 관제는 핵심 기능으로, 이를 위해선 실시간으로 차량에서 발생하는 데이터를 수집해서 빠르게 시각화할 수 있도록 스트리밍은 필수입니다.

### 데이터 파이프라인의 주요 컴포넌트 소개

![overall-architecture.png](/img/build-fms-data-pipeline/overall-architecture.png)_FMS 데이터 파이프라인 아키텍처_

이번 챕터에서는 FMS 데이터 파이프라인을 구성하는 주요 컴포넌트들을 가볍게 소개드리고 아래 챕터에서 더 자세하게 다루도록 하겠습니다. 이미 알고 있는 내용이라면 아래 "데이터가 흐르는 순서"로 바로 읽으셔도 좋습니다. (참고: 본 글에서는 실시간 조회에 사용되는 Redis와 관련 서비스(Consumer, Backend API 등)은 따로 다루지 않습니다)

처음으로 실시간 파이프라인을 담당하는 주요 컴포넌트입니다.

**IoT Core**  
AWS IoT Core는 IoT 디바이스의 메시지를 송/수신하는 메시지 브로커로 내부는 MQTT 프로토콜로 구현되어 있습니다. Fully Managed Service로 인프라 관리가 필요 없고 보안이나 디바이스 관리 등의 이점이 있어 현재 쏘카 서비스와 FMS 모두 IoT Core를 차량 데이터의 매개체로 사용하고 있습니다.

**MSK(Managed Streaming for Kafka Service)**  
보통 실시간 데이터 파이프라인 아키텍처를 설계할 떄 메시지 브로커인 Kafka를 많이 선택합니다. Kafka는 분산 스트리밍 플랫폼으로 실시간으로 들어오는 데이터를 확장성있게 처리할 수 있어 많은 기업들이 메시지 브로커로 사용하고 있습니다 (본 글에서는 Kafka에 대해 자세하게 다루지 않곘습니다)

Kafka는 메시지 브로커로 많은 장점이 있지만 관리하기 위해서 많은 지식과 노하우를 필요로 합니다. AWS에서 제공하는 MSK(Managed Streaming for Kafka Service)는 Kafka를 완전관리형으로 제공해주며 사용자 측에서 관리 비용을 아끼고 애플리케이션 개발에 집중할 수 있습니다. 실제로 Kafka를 운영하기 위한 컴포넌트들(Broker, Zookeeper 등)을 자동으로 프로비저닝해주고 보안, 모니터링 등을 폭넓게 지원해줘서 쏘카에서도 많이 사용하고 있는 서비스입니다.

**Kafka Connect**  
Kafka Topic에 저장된 메시지들을 데이터베이스/스토리지에 적재하기 위해서는 이를 처리하는 애플리케이션이 필요합니다. 일반적으로 각 프로그래밍 언어에서 이를 구현할 수 있도록 Consumer 라이브러리가 있습니다.  
Kafka Connect는 Consumer를 메시지 추출(Source)과 적재(Sink)에 적절하게 추상화한 프레임워크입니다. 실제로 작업을 수행하는 Kafka Connector들을 Kafka Connect에 등록하여 관리하게 되는 구조라고 보시면 됩니다. 사용자는 이를 이용해 손쉽게 Kafka의 메시지를 추출/적재할 수 있습니다.  
현재 MongoDB, MongoDB, S3, ElasticSearch 등을 대상으로 메시지를 처리하는 Kafka Connector가 오픈소스로 많이 존재합니다 ([Confluent Hub](https://www.confluent.io/hub)에서 확인이 가능합니다)

다음은 FMS 서비스에서 주요 사용되는 데이터베이스입니다.

**Redis**  
Redis는 실시간 데이터 조회에 활용되는 Key-Value 기반 데이터베이스입니다.
현재 서비스에서 차량 관제를 할 때 실시간으로 차량의 이동 정보나 전압 등의 상태 등을 확인하기 위해서 Redis에서 데이터를 조회합니다. Kafka의 메시지를 Consumer가 실시간으로 적재하고 있습니다.

**DynamoDB**  
DynamoDB는 AWS에서 운영하는 완전관리형 NoSQL 데이터베이스입니다. 사용자는 손쉽게 DynamoDB 대한 성능을 설정할 수 있고 주요 지표들로 모니터링을 하기 용이합니다. 5분단위 집계된 평균 화물칸의 온도와 같은 실시간으로 집계하는 용도나 준 실시간 데이터 조회 목적으로 사용되고 있습니다.

**S3**  
S3는 AWS에서 제공하는 객체 스토리지로 다양한 형태(정형/비정형)의 파일들을 저장할 수 있습니다. 현재 배치 분석에 필요한 json 원본 데이터와 Parquet 형식의 데이터를 저장하는 데이터 레이크 형태로 사용하고 있습니다.

다음은 배치 분석 집계를 위한 컴포넌트입니다.

**Lambda**  
Lambda는 AWS의 서버리스 컴퓨팅 플랫폼입니다. 개발자는 서버에 대한 존재를 모르고 코드만 작성하면 손쉽게 서버를 운영할 수 있게 됩니다. 람다는 API 서버 형태로 활용이 가능할 뿐만 아니라 AWS 리소스(S3, Kafka, Kinesis 등)의 이벤트 기반으로 동작시킬 수 있어 활용 범위가 굉장히 넓습니다.  
현재 배치 분석 집계를 할 때 S3의 적재된 Raw 파일을 타입에 맞게 분리하여 parquet로 변환하는 용도로 Lambda를 사용하고 있습니다.

**Redshift**  
Redshift는 AWS의 데이터 웨어하우스 서비스입니다. 표준 SQL(ANSI)를 따르며 대용량 데이터 처리를 빠르게 처리할 수 있어 데이터 웨어하우스 솔루션으로 AWS Athena(Presto 기반)와 함께 많이 사용되고 있습니다.  
저희는 Redshift 운영 비용을 낮추기 위해 Serverless를 사용하고 S3에 적재된 Parquet를 읽을 수 있도록 Redshift Spectrum 기능을 도입하였습니다.

**Glue Data Catalog**  
Glue는 AWS의 완전 관리형 ETL 도구입니다. Glue는 크게 `Data Catalog`와 `Data Integration and ETL`로 나뉘는데, 이번에 저희가 주로 사용한 서비스는 Data Catalog입니다. Data Catalog는 비정형/반정형 데이터들을 SQL 형태로 조회할 수 있도록 메타 정보들(테이블, 파티션 등)을 저장하는 메타스토어로 활용됩니다.  
현재 S3에 Parquet로 데이터를 저장할 때 스키마를 추론하는 용도와 Redshift Spectrum에서 외부 테이블 형태로 사용할 때 Glue Catalog를 활용하고 있습니다.

**Airflow**  
Airflow는 Airbnb에서 개발한 워크플로우 관리 오픈소스로 현재 많은 기업에서 데이터 파이프라인을 자동화할 때 사용하는 툴입니다. 스케줄링, 재처리 기능, 외부와 연동해주는 다양한 3rd party 라이브러리, 직관적인 UI 등을 제공해줘서 많은 기업들이 배치 집계를 할 때 사용하고 있습니다.  
현재 [RedshiftSQLOperator](https://airflow.apache.org/docs/apache-airflow-providers-amazon/2.4.0/operators/redshift.html)를 활용해서 데이터 마트 데이터 집계를 스케줄링하는데 사용하고 있습니다.

여기까지 주요 컴포넌트들에 대해 가볍게 소개드렸습니다. 더 자세한 내용은 글에서 더 다루도록 하겠습니다.

### 데이터가 흐르는 순서

처음에 차량에서 수집되는 여러 상태 데이터들은 최종적으로 데이터베이스/스토리지에 적재됩니다. 아래와 같은 흐름을 거쳐 FMS 서비스에 필요한 형태로 데이터가 저장됩니다.

1. 차량에서 다양한 상태의 데이터를 수집합니다
2. 수집되는 데이터는 발송 주기에 맞춰 IoT Core로 전송합니다.
3. IoT Core 메시지 브로커에 저장된 메시지는 라우팅 규칙에 따라 상위 주제별로 Kafka Topic으로 라우팅됩니다.
4. Kafka Topic의 각 파티션에 저장된 메시지는 Kafka Connect를 통해 데이터 싱크(DynamoDB, S3)로 적재됩니다 (Redis는 필터링을 하는 Kafka Consumer를 통해 적재됩니다)
5. S3에 적재된 Json 파일은 람다를 통해 분류/변형 후 S3에 적재됩니다 (Redshift, Athena 쿼리에 적합한 형태로 적재됩니다)
6. Airflow로 스케줄링된 Redshift 쿼리를 통해 데이터를 집계하여 RDS(데이터 마트)에 저장합니다.

## 2. 차량 데이터가 Kafka로 오기까지

### 차량 IoT 데이터의 특징

쏘카 서비스와 동일하게 FMS 서비스도 관리하는 차량들은 IoT 디바이스 내에서 차량의 상태 정보를 수집 서버(AWS IoT Core)로 전송합니다. 해당 메시지는 가공/적재 과정을 거쳐 서비스에서 활용됩니다.

쏘카의 차량에서 수집되는 상태 메시지는 다음과 같은 특징들이 있습니다.

1. 보고하는 유형과 제어 응답 유형이 있습니다  
   일반적으로 차량을 관제하기 위해선 차량 디바이스에서 차량의 상태를 주기적으로 수집해서 보고하는 것이 필요합니다. 실제로 특정 프로토콜은 차량의 위치(위도, 경도)와 속도 같은 이동 정보를 주기적으로 보고하는 역할을 합니다.  
   또한 차량 디바이스를 제어하기 위해 클라이언트에서 명령을 보낼 수 있습니다. 이떄 디바이스는 명령은 수행한 후 결과를 응답하게 됩니다. 예를 들어 블랙박스에 녹화되고 있는 영상을 업로드 하라는 명령이 있습니다. 디바이스는 이를 수행한 후 결과를 메시지로 수집 서버에 전송합니다.

2. 차량의 상태를 표현하기 위한 다양한 프로토콜이 존재합니다.  
   FMS 서비스에서 차량 관제, 운전 효율화 등의 기능을 제공하기 위해선 다양한 수집 데이터를 필요로 합니다.
   차량 운행 상태, 화물 차량의 온도 상태, 블랙박스 상태 등 각 역할 별로 프로토콜을 나눠서 수집해야 합니다. 실제로 현재 수집되는 메시지 프로토콜의 유형은 OO개가 넘습니다. 이들은 특성과 목적에 맞게 필드 값에 따라 분류되어 있습니다.

    위처럼 메시지 프로토콜이 다양하다 보니 쏘카에서는 프로토콜별로 스키마를 설계할 때 프로젝트에 참여하는 주요 팀들과 함께 논의를 진행했습니다. 덕분에 스키마간의 통일성과 규칙이 생겼으며 데이터를 처리하는 쪽에서는 예측 가능하게 소프트웨어 개발이 가능했습니다. 개인적으로 스키마 설계를 할 때 이해관계자들이 함께 참여하는 것이 정말 중요하다고 느껴졌습니다.

3. 주기적으로 보고하는 유형은 보통 배치로 묶어서 전송합니다
   디바이스에서 수집하는 상태 정보들은 프로토콜 별로 설정된 Hz 수집 주기에 따라 수집됩니다. 이때 메시지를 수집 서버로 전송한다면 통신비나 클라우드 리소스(IoT Core, Kafka 등)의 비용이 더 비싸집니다. 따라서 비용 효울화를 위해 상태 정보를 배치 형태로 묶어서 전송 주기에 따라 전송됩니다. 그래서 주기 보고의 메시지는 아래와 같은 형태로 구성됩니다.

    ```json
    {
        "object": "vehicle",
        "type": "kinematic",
        "messaged_at": "2023-01-01T12:00:00+09:00",
        "measurements": [
            {
                "timestamp_iso": "2023-01-01T11:59:00+09:00",
                "speed": 30,
                ...
            },
            {
                "timestamp_iso": "2022-01-01T11:59:01+09:00",
                "speed": 30,
                ...
            },
            ...
        ],
        ...
    }
    ```

### IoT Core에서 Kafka로

차량에서 수집된 데이터는 IoT Core의 `메시지 라우팅` 규칙에 따라 kafka로 전송됩니다. 위 메시지에서 분류 목적으로 사용하고 있는 `object` 필드에 대응되는 각 kafka topic으로 라우팅됩니다.

여기서 한가지 중요한 점은 하나의 topic에 여러 프로토콜의 메시지가 들어올 수 있다는 점입니다. 메시지 프로토콜에 1:1 대응되도록 topic을 만든다면 수많은 토픽을 관리해야 하며 관리에 대한 부담이 늘어나게 됩니다.  
보통 Kafka를 운영하면 메시지 스키마에 대한 검증을 하기 위해 사용하는 Schema Registry를 사용하는 경우가 많습니다. 쏘카의 유즈케이스에서는 하나의 토픽에 여러 메시지가 들어오다 보니 Schema Registry를 도입하기가 쉽지 않았기에 우선 장애 없이 메시지를 적재하고 이후에 스키마를 검증하는 `fail safe` 방식으로 아키텍처를 설계하였습니다.

### Kafka 관리

현재 MSK를 통해 kafka를 운영하고 있습니다. 현재 모니터링은 MSK의 [향상된 파티션 수준 모니터링](https://docs.aws.amazon.com/msk/latest/developerguide/metrics-details.html#topic-partition-metrics) 설정으로 kafka 운영에 필요한 주요 메트릭을 cloudwatch에서 확인하고 있습니다. 이를 통해 기본 메트릭 뿐만 아니라 Consumer Group 별로 Lag 확인이 가능해서 모니터링하는데 큰 도움이 되고 있습니다.

토픽의 경우 메시지의 분류 필드인 `object` 별로 생성하여 관리하고 있으며, 실패한 메시지들을 저장하는 deadletter 전용 토픽이나 일부 유즈케이스에 사용되는 토픽 등이 있습니다. 주요 topic들은 partition과 replication factor를 설정해서 처리 성능과 가용성을 높게 유지하고 있습니다.

저장되는 메시지는 실시간으로 **UI for Apache Kafka**를 통해 확인하고 있습니다. UI for Apache Kafka는 직관적인 UI로 kafka 관리를 위한 많은 기능들을 제공해줍니다. 특히 토픽에 쌓이는 메시지를 실시간으로 조회가 가능하며 여러 검색 방식을 지원해줘서 초기에 Kafka 관리 툴로 사용하기에 적합합니다.

## 3. 실시간 데이터 처리와 적재를 한 번에, Kafka Sink Connector 개발하기

본 장에서는 Kafka 토픽에 저장된 메시지를 외부 데이터 싱크(DynamoDB, S3)로 적재하는 Kafka Sink Connector를 개발하게 된 배경과 구현 사항, 장단점 등에 대해 알아보도록 하겠습니다.

### Kafka Connect란?

![kafka-connect.jpeg](/img/build-fms-data-pipeline/kafka-connect.jpeg)_kafka connect의 역할 (출처: https://developer.confluent.io/learn-kafka/kafka-connect/intro)_

보통 Kafka 토픽의 메시지를 적재하기 위해선 크게 2가지 방식을 활용합니다(클라우드, SaaS에서 제공해주는 기능은 제외하였습니다) 첫 번째는 프로그래밍 언어 별 존재하는 kafka sdk를 사용해서 Kafka Consumer를 구현하는 것이며 두 번째는 Kafka Connect를 활용하여 적재하는 방법입니다.

Kafka Consumer는 높은 자유도로 개발이 가능하며, 다양한 프로그래밍 언어(JVM 계열 언어, Python, Javascript 등)로 개발할 수 있도록 SDK를 지원합니다. 일반적으로 Kafka 토픽의 메시지를 처리할 때 광범위하게 사용됩니다.

Kafka Connect는 Consumer를 한단계 추상화하여 제공하는 Confluent에서 개발한 프레임워크입니다. 데이터 소스에서 Kafka로 데이터를 옮기거나 Kafka에서 데이터 싱크로 적재하는 목적으로 주로 사용됩니다. 대중적인 데이터 소스/싱크에 대한 Connector(Mysql, MongoDB, S3, ElasticSearch 등)는 이미 오픈소스로 나와있어 손쉽게 사용이 가능합니다 ([Confluent Hub](https://www.confluent.io/hub)에서 확인이 가능합니다)

Kafka Connect를 썼을 때 장점은 아래와 같습니다

-   다양한 데이터 소스,싱크에 대한 오픈소스를 활용하면 손쉽게 데이터 이동이 가능합니다.
-   프레임워크의 가이드를 따라 Connector를 손쉽게 개발하여 사용할 수 있습니다.
-   REST API를 통해 Kafka Connect의 운영이 가능합니다.
-   Worker와 Task 갯수 조정을 통해 손쉽게 스케일 아웃이 가능합니다.
-   Property 기반으로 Kafka Connector 설정을 할 수 있어 선언적인(declarative) 소프트웨어 운영이 가능해집니다. 아래 예시는 S3 Sink Connector를 배포할 때 사용하는 프로퍼티입니다.

    ```json
    {
        "connector.class": "kr.socar.fms.connector.s3.S3SinkConnector",
        "topics": "OOO",
        "tasks.max": "1",
        "key.converter.schemas.enable": "false",
        "value.converter.schemas.enable": "false",
        "value.converter": "org.apache.kafka.connect.json.JsonConverter",
        "key.converter": "org.apache.kafka.connect.json.JsonConverter",
        "flush.size": "60000",
        "rotate.interval.ms": "300000",
        "s3.part.size": "60000000",
        "partition.duration.ms": "3600000",
        "s3.region": "ap-northeast-2",
        ...
    }
    ```

    하지만 꼭 장점만 있는 것은 아닙니다.

-   Kafka Connect 프레임워크의 동작방식을 기본적으로 이해하고 있어야 합니다.
-   Kafka Consumer에 비해 상대적으로 테스트하기나 디버깅하기가 불편합니다.
-   Java로 개발되어 있어 JVM 계열 언어로만 개발이 가능합니다.
-   간단한 변형 후 적재가 아닌 비즈니스 요구사항이나 복잡한 처리가 포함된 작업을 구현할 경우 Kafka Consumer로 구현하는 것이 수월합니다.

Kafka Connect에 대한 더 자세한 설명은 [여기](https://docs.confluent.io/platform/current/connect/index.html)를 참고해주세요.

### Kafka Connect의 동작 방식

Kafka Connect는 Kafka와 외부 데이터 소스/싱크를 연결해주는 프레임워크입니다. Kafka Connector는 Kafka Conneect에서 실제로 동작하는 구현체이며 Kafka Connect에 의해 관리됩니다. Kafka Connect를 사용하기 위해선 Kafka Connector를 jar 형태로 Kafka Connect 내부(보통 Docker Image)에 포함시킨 후, Kafka Conenct를 실행한 후 제공되는 API로 Kafka Connector를 등록하는 과정을 거치게 됩니다.

Kafka Connector는 Source와 Sink 2가지 방식을 제공합니다. Source Connector는 데이터 소스에서 Kafka 토픽으로 메시지를 전달하고 Sink Connector는 Kafka 토픽에서 데이터 싱크로 메시지를 전달합니다. 실제로 Kafka Connector를 구현하기 위해서 Source, Sink에 따라 나뉘어진 인터페이스를 따르게 됩니다.

Kafka Connect를 관리하기 위해선 `Worker`와 `Task`의 개념을 알고 있어야 합니다. Task는 Kafka Connector의 논리적인 실행 단위이며 JVM으로 실행되는 Process라고 보시면 됩니다. 보통 하나의 Task는 Kafka 토픽의 한 개 이상의 파티션을 담당합니다 (보통 1개의 Task가 1개의 파티션을 맡도록 운영합니다).

Worker는 Task를 운영하는 물리적인 프로세스로 Task의 라이프사이클을 담당합니다. 즉 하나의 Worker에는 여러 개의 Task를 실행하고 있으며, Worker끼리 서로 통신하면서 Task를 재할당해주기도 합니다. FMS 프로젝트에서는 Kubernetes 환경에서 Kafka Connect를 운영하며 이때 Worker는 `Pod`이 됩니다.

Kafka Connect는 `Standalone Mode`와 `Distributed Mode`가 있는데, Standalone은 Worker를 1개, Distributed Mode는 Worker를 여러 개 사용할 수 있습니다. 주로 운영 환경에서 Kafka Connect는 Distributed Mode로 사용하며 여러 개의 Worker와 Task를 상황에 맞게 조정하며 변화되는 트래픽에 유연하게 대응합니다.

Kafka Connect의 동작 방식에 대한 더 자세한 내용은 [여기](https://docs.confluent.io/platform/current/connect/concepts.html)를 참고해주세요.

### 요구 사항 및 결정 이유

Kakfa 토픽의 메시지를 처리하기 위해 Kafka Consumer와 kafka Connect 중 선택할 때는 현재 비즈니스 요구 사항에 맞춰 장/단점을 잘 비교하여 선택하는 것이 중요합니다. 사실 Kafka 토픽의 메시지를 단순하게 적재하는 경우라면 오픈소스 Kafka Connector를 사용하는 게 낫습니다. 하지만 FMS 프로젝트에는 아래와 같은 요구사항들이 있었고, 충분히 기술적 검토를 한 후 Kafka Connector를 직접 개발하여 하나의 Kafka Connect로 메시지 적재를 관리하자는 결정을 내렸습니다.

1.  **Kafka 토픽 별 메시지들이 S3와 DynamoDB에 적재되어야 합니다.**  
    Kafka에서 S3로 데이터를 적재하는 S3 Sink Connector는 오픈소스로 존재하여 많은 곳에서 사용하고 있습니다. 하지만 DynamoDB의 경우 별도의 Sink Connector가 존재하지 않아 직접 구현이 필요한 상황이었습니다. 이미 제공되는 S3 Sink Connector를 사용하면서 DynamoDB도 Connector 형태로 개발한다면 하나의 플랫폼으로 빌드, 운영할 수 있게 되어 이점이 있을 것이라고 판단하였습니다.

2.  **일부 요구사항에 맞게 가벼운 변형 작업이 필요합니다.**  
     DynamoDB는 레코드를 추가할 때 Partition Key를 필수적으로 입력해야 합니다. FMS 프로젝트에서 DynamoDB 테이블은 비용/성능 효율화를 위해 [Single Table Design](https://aws.amazon.com/ko/blogs/compute/creating-a-single-table-design-with-amazon-dynamodb/) 기법으로 디자인했고 이에 맞는 Partition key가 적재되기 전에 메시지에 추가되어야 합니다. 이외에도 비용 절감을 위해 불필요한 컬럼을 삭제하는 것도 고려가 필요합니다.

    여기서 Kafka Connect에는 [SMT(Single Message Transformation)](https://docs.confluent.io/platform/current/connect/transforms/overview.html)가 있어 Property 기반으로 손쉽게 메시지의 변형이 가능합니다. 물론 SMT 특성상 제약 사항이 존재하지만 필요하면 직접 Transfrom 을 구현하여 사용이 가능합니다.

3.  **스트리밍 환경에서 신뢰성과 확장성이 보장되어야 합니다.**
    스트리밍 환경에서는 메시지를 빠르게 처리 후 적재하는 것이 중요합니다. 따라서 kafka의 메시지가 빠르게 쌓여도 Transformation & Load 레이어에서는 일관성있게 처리할 수 있어야 합니다.  
    Kafka Connect는 `Distributed Mode`를 통해 Worker 갯수를 조정하여 Scale Out/In을 쉽게 할 수 있으며, Worker가 만약 실패하더라도 기존 Worker들에 Task들을 리밸런싱 해줘서 안전하게 운영이 가능합니다.

**결과적으로 DynamoDB Sink Connector를 직접 개발하였으며, S3 Sink Connector도 추가 요구사항을 위해 Class를 Override하여 커스마이징하였습니다.**

### Kafka Connector 레포 구성

참고 : Kafka Connector를 직접 개발하고 싶다면 [여기](https://docs.confluent.io/platform/current/connect/devguide.html#developing-a-simple-connector)에서 더 자세한 정보를 확인해보세요.

Kafka Connector를 구현하는 레포지토리는 Kotlin으로 작성되었으며 아래와 같은 구성으로 이뤄집니다. S3, DyanmoDB Sink Connector가 멀티 모듈 형태로 구성되어 있으며 공통 기능(변형)을 하는 모듈을 별도로 의존하고 있습니다.

```
...
build.gradle.kts
Dockerfile
e2e
subprojects
├── core
│   └── src
│       ├── main/...
│       |   ├── converters
│       │   │   ├── SplitListConverter.kt
│       │   │   └── UpdateFieldsConverter.kt
│       │   └── transforms
│       │       ├── InsertFieldInStringTemplate.kt
│       │       └── SplitArrayField.kt
│       └── test/...
├── dynamodb
│   └── src
│       ├── main
│       │   ├── DynamoDbDao.kt
│       │   ├── DynamoDbSinkConnector.kt
│       │   ├── DynamoDbSinkConnectorConfig.kt
│       │   └── DynamoDbSinkTask.kt
│       └── test/...
└── s3
    └── src
        ├── main
        │   ├── S3SinkConnector.kt
        │   ├── S3SinkConnectorConfig.kt
        │   └── S3SinkTask.kt
        └── test/...

```

S3 Sink Connector의 경우

build.gradle.kts에는 Kafka Connector 관련 의존성(org.apache.kafka:connect-api)을 추가하고 S3 Sink Connector의 구현체

```gradle
implementation("io.confluent:kafka-connect-s3:10.0.11")
implementation("org.apache.kafka:connect-api:$kafkaVersion")
implementation("org.apache.kafka:connect-transforms:$kafkaConnectTransformVersion")
```

DynamoDB는 레코드를 추가할 때 Partition Key를 필수적으로 입력해야 합니다. FMS 프로젝트에서 DynamoDB 테이블은 비용/성능 효율화를 위해 [Single Table Design](https://aws.amazon.com/ko/blogs/compute/creating-a-single-table-design-with-amazon-dynamodb/) 기법으로 디자인했고 이에 맞는 Partition key를 추가해줘야 했습니다. Kafka Connect에는 [SMT(Single Message Transformation)](https://docs.confluent.io/platform/current/connect/transforms/overview.html)가 있어 Property 기반으로 손쉽게 메시지의 변형이 가능합니다. 물론 SMT 특성상 제약 사항이 존재하지만 필요하면 직접 Transfrom 을 구현하여 사용이 가능합니다.  
아래는 메시지를 템플릿 언어 기반으로 변경할 수 있도록 구현한 Transform입니다.

```json
"transforms": "RenameField",
"transforms.InsertFieldInStringTemplate.field": "pk",
"transforms.InsertFieldInStringTemplate.value" : "${id}#${object}#${type}"
```

-   as-is

```json
{
    "object": "vehicle",
    "type": "kinematic",
    "id": 350
}
```

-   to-be

```json
{
    "object": "vehicle",
    "type": "kinematic",
    "id": 350,
    "pk": "350#vehicle#kinematic"
}
```

또한 위에서 언급했듯이 FMS 프로젝트의 차량 IoT 데이터는 보통 배치로 묶여서 메시지들이 들어옵니다. 만약 클라이언트가 이렇게 nested된 형태의 데이터를 쿼리하는 경우 전처리를 진행해야 하기 때문에 적재하기 전에 데이터를 풀어서 적재해주는 것이 좋습니다.  
보통 메시지를 전처리하기 위해서 적재 전에 별도의 Consumer를 두곤 하지만, PoC 단계에서 관리 포인트를 높이고 싶지 않았습니다. 그래서 Kafka Connector에서 Property 기반으로 배치 메시지를 풀어줄 수 있도록 기능을 추상화하여 제공하였습니다.

```json
"converter.split.list.key": "measurements"
```

-   as-is

```json
{
    "object": "vehicle",
    "type": "kinematic",
    "messaged_at": "2023-01-01T12:00:00+09:00",
    "measurements": [
        {
            "timestamp_iso": "2023-01-01T11:59:00+09:00",
            "speed": 30
        },
        {
            "timestamp_iso": "2022-01-01T11:59:01+09:00",
            "speed": 40
        }
    ]
}
```

-   to-be

```json
[
    {
        "object": "vehicle",
        "type": "kinematic",
        "messaged_at": "2023-01-01T12:00:00+09:00",
        "timestamp_iso": "2023-01-01T11:59:00+09:00",
        "speed": 30
    },
    {
        "object": "vehicle",
        "type": "kinematic",
        "messaged_at": "2023-01-01T12:00:00+09:00",
        "timestamp_iso": "2023-01-01T11:59:01+09:00",
        "speed": 45
    }
]
```

### DynamoDB Sink Connector 요구 사항 및 구현

-   요구사항

-   DynamoDB Connector는 오픈소스로 따로 없고 유료 라이센스가 있었음
-   구현 코드

### Custom S3 Sink Connector 요구 사항 및 구현

-   요구사항

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

## 4. 비정형 데이터를 Redshift에서 조회하기 까지

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

## 5. 견고한 파이프라인을 위한 통합 테스트 환경 구축하기

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

## 6. 시뮬레이터를 활용한 실 데이터 기반 부하 테스트

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

## 7. 데이터 신뢰성을 위한 정합성과 무결성 검증하기

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

## 8. 마무리

### 남은과제

### 결론
