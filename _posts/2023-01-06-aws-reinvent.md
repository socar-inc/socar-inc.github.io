---
layout: post
title:  "AWS re:Invent 2022 참석"
subtitle: AWS re:Invent 2022 참석
date: 2023-01-06 10:00:00 +0900
category: data
background : "/img/advanced-airflow-for-databiz/background.jpg"
author: alti
comments: true
tags:
    - aws
    - dba 
    - database
---


안녕하세요! 서비스 엔지니어링 본부, Cloud DB팀의 알티입니다.

Cloud DB팀은 Data Architecture(DA)의 역할과 Database Administrator(DBA)역할을 수행합니다.
DA로서는 전사 데이터의 표준화와 데이터 거버넌스를 맡고 있으며, DBA로서는 쿼리 검수, 물리 모델링, 트러블 슈팅, 장애 대응, 신규기술, 업무 자동화 등의 다양한 업무를 맡고 있습니다.

AWS re:Invent는 글로벌 클라우드 시장에서 가장 큰 사업자인 AWS가 매년 신규 서비스와 활용사례 등을 발표하는 AWS의 최대규모 행사입니다.
데이터베이스 관리자로서 쏘카에서 업무를 수행할 때 AWS에서 제공하는 다양한 서비스를 사용하고 있는데, 좋은 기회로 이번 2022년 11월 28일부터 12월 02일까지 AWS re:Invent에 참석을 하게 되었습니다. 
이번 컨퍼런스를 통해 다음의 내용을 배울 수 있었습니다.

- AWS 서비스를 어떻게 사용해야 잘 사용할 수 있는 것인가?
- 각각의 서비스를 어떻게 유기적으로 사용하는 것이 Best Practice 인가?
- 글로벌 기업들은 대량의 트랜잭션을 다루기 위해 AWS 서비스를 어떻게 이용하였는가?
- AWS Re:Invent를 참석한 많은 사람들의 기술교류를 위한 태도는 어떠한가?

다음과 같은 분들이 읽어보시면 좋습니다.

- AWS re:Invent에 관심있는 독자
- 데이터베이스 관리자
- AWS의 New Feature에 관심있는 독자
- 데이터베이스에 관심있는 개발자

목차는 다음과 같습니다.

1. AWS Re:Invent 소개
2. SOCAR의 DBA가 듣고싶어했던 세션은 무엇일까?
    
    2.1. Purpose Built Database
    
    2.2. K8s Application With Ultra-Fast Amazon MemoryDB For Redis
    
    2.3. Modernize and gradually migrate your data model SQL to NoSQL
    
    2.4. Blue/Green Optimized Writers & Reads
    
    2.5. Access Data Lakes in S3 Using Open-source Projects
    
3. AWS Re:Invent 참석 후기

---

## 1. AWS re:Invent 2022 소개

![aws-reinvent-intro](/img/aws-reinvent/aws-reinvent-intro.jpg)*AWS re:Invent*

AWS re:Invent는 아마존 웹 서비스(Amazon Web Service)가 개최하는 최대 규모의 행사이며, 매년 AWS의 신규 서비스와 기존 서비스의 새로운 기능을 발표하는 자리입니다.

뿐만 아니라, 각 분야의 전문가와 각종 IT 관계자들이 모여 Best Pratice 등의 활용사례 등을 발표하고 공유하기도 합니다.

AWS가 제공하는 서비스 이외에도 Expo에서도 최대 규모의 행사인 만큼 한번쯤 들어봤을 만한 유명한 굴지의 솔루션 업체(GitLab, Redis, Docker, MariaDB 등등)이 모여 제공하는 서비스에 대한 소개를 들어볼 수 있는 컨퍼런스 입니다.

-----

## 2. DBA와 관련된 세션 소개

데이터베이스 관리자의 업무를 수행하면서 당연히 관심있는 분야는 데이터베이스 분야입니다.
다만, 관심있는 분야를 누군가에게 설명하고 이해시키기 위해서는 AWS Session에서만 들어볼 수 있는 다양한 이야기들이 있습니다.
글로벌 기업들의 아키텍처의 구성은 어떻게 되어있는지, 어떤 데이터베이스를 어떻게 사용하는지 확인해볼 수 있습니다.
또한 데이터베이스와 결합하여 서비스를 제공하는 사례 등을 중점적으로 듣고자 했습니다.
총 10개의 세션 중 인상깊었던 5개의 세션들을 아래에 소개합니다.

### 2.1. 목적에 맞는 데이터베이스

**세션 주제**

이 세션에서는 현대 MSA(Microservice Architecture) 환경에서 목적에 맞는 데이터베이스 선택(purpoose-built database apporach)의 중요성과 실제 기업의 데이터베이스 컨설팅 사례를 살펴봤습니다. 

MSA를 구성하는 각 서비스들은 확장성, 고가용성, 보안성, 성능 측면에서 각자 필요한 스펙이 다르기 때문에 그 목적에 맞는 데이터베이스를 선택하는게 중요하다고 설명하고 있습니다. 예를 들면 Relational / key-value / caching / time-series 등 여러 형태의 database portfolio 에서 적절한 DB를 선택할 수 있습니다. 

![AWS 가 제공하는 다양한 database](/img/aws-reinvent/database-portfolio.png)*AWS 가 제공하는 다양한 database*

구체적인 사례로는 Twilio 라는 기업이 Postflight 에서 Amazon DynamoDB 로 이관하여 Data Delay 와 비용을 줄인 케이스와, ADP가 Amazon Neptune 을 사용하여 성능을 10배 이상 개선한 케이스 등을 살펴보았습니다. 

**소감**

데이터의 성격과 목적에 맞게 데이터베이스를 선택하는 일은 중요합니다. 데이터도 사람처럼 성격과 성향이 있다고 느끼는데, 업무 상 데이터 모델링 요청이 올때도 이러한 데이터의 특성을 고려하여 알맞은 데이터베이스를 사용하는 것이 필수적입니다. 추후에 데이터가 방대하게 쌓이고 난 후에는 간단한 작업도 큰 작업이 될 수 있으며, 성능에 큰 영향을 미칠 수도 있기 때문입니다. 

구체적으로는 단순 조회작업에 오랜 시간이 소요되거나 CPU부하가 발생할 수도 있습니다.
그 예로 실시간 위치(위,경도)를 조회하고 있는 데이터의 경우 지속적으로 RDBMS에 UPDATE 작업을 진행해야 합니다.
이런 경우 RDBMS에 대량의 데이터를 적재하기보다는 DynamoDB를 활용하여 데이터를 적재하는 것이 더욱 좋은 방법이라 생각됩니다.

추가로, 실시간 데이터가 필요하다면 Redis를 활용하고 데이터 손실을 줄이기 위해서는 MemoryDB For Redis를 사용한다거나, 데이터 손실을 감수하더라도 조금 더 빠른 조회결과를 Return 받고자한다면, memcached 를 이용하여 데이터베이스를 선택하면 안정적인 서비스를 제공할 수 있을 것이라 생각합니다.

이 세션을 통해 다시금  “목적에 맞는 데이터베이스 선택”의 중요성을 깨닫고, 개발자와 모델링 협업 과정에서 해당 부분에 대해 어떻게 잘 커뮤니케이션 할 수 있을지에 대해 고민해볼 수 있었습니다.

### 2.2 MemoryDB For Redis와 쿠버네틱스를 활용한 초고속 애플리케이션 구축

**안정적이고 빠른 결과를 위한 인프라 구축**

해당 세션에서는 Kubernetes와 MemoryDB for Redis 결합을 통한 애플리케이션 구축의 Best Pratice를 소개했습니다.
이번 세션 마지막에는 소개한 구성을 실제로 구현하는 데모도 진행되었습니다. 실습 내용을 포함한 전체 발표 자료는 [링크](https://d1.awsstatic.com/events/Summits/reinvent2022/DAT313-R_Build-stateful-K8s-applications-with-ultra-fast-Amazon-MemoryDB-for-Redis.pdf)에서 보실 수 있습니다.

구체적으로는 AWS EKS 를 통해 Kubernetes 환경에서 MicroService Architecture(MSA) application 을 구성하는 방법과, MSA 에 Memory DB for Redis가 적합한 이유를 소개하였습니다.

전체적인 구성은 아래와 같습니다.

![ACK](/img/aws-reinvent/ack.png)*ACK*

세션 초반부에는 MSA의 정의와 Managed Service 로서의 AWS EKS의 장점, AWS Controller for Kuberenetes 를 통해 리소스를 관리하는 방법들을 설명합니다. 

![memorydb-durability](/img/aws-reinvent/memorydb.png)*Memory DB For Redis의 스토리지는 Aurora와 동일합니다.* 

여기서 Aurora와 동일한 스토리지 구성이라는건 Memory 기반의 데이터베이스에는 큰 의미가 있습니다.
Memory 기반의 데이터베이스인 Redis는 빠른 Return을 보장하지만, 데이터에 대한 내구성이 떨어지는 것이 사실입니다.
하지만, 관계형 데이터베이스와 동일한 스토리지 구성을 갖는다는 것은 데이터의 손실이 발생하지 않도록 보장한다는 것이며, 기존의 Redis보다 데이터에 대한 안정성을 보장하는 메모리기반의 데이터베이스라는 점입니다.
해당 세션을 들으면서 빠른 응답속도를 보장하기 위해 쿠버네틱스와 함께 데이터의 손실을 방지하고자 Memory DB For Redis를 사용하는 이유를 알아볼 수 있었습니다.

쏘카의 DB팀은 인프라팀과 긴밀하게 협업을 하고 있습니다.
예를 들면 로컬 PC에서 VPN를 통한 RDS 접근 방법이나 쏘카 서비스 이용시 Load balancer부터 데이터베이스 구성까지 DB팀과 인프라팀의 업무는 긴밀하게 붙어있는 편입니다.
그래서 인프라팀과 커뮤니케이션을 할 일이 많은데 이런 커뮤니케이션에 도움이 되는 세션이었습니다. 

### 2.3. SQL에서 NoSQL로 점진적으로 마이그레이션

**관계형 데이터베이스에서 비 관계형 데이터베이스 데이터 이관**

이번에 들었던 세션은 관계형 데이터베이스에 적재된 데이터를 In-Memory기반의 데이터베이스에 데이터를 이관하는 방법에 관련된 이야기입니다.

이기종 간의 데이터베이스 동기화 및 이관작업은 상대적으로 동일한 데이터베이스로 이관하는 것보다 신경써야하는 부분이 사실입니다. (데이터베이스 간의 버전차이, 호환성 등 고려)

관계형 데이터베이스에서 비 관계형 데이터베이스로 마이그레이션 작업은 적재되는 모든 쿼리의 결과를 메모리형 데이터베이스에 적재되어야하며, 추가로 운영중인 데이터를 동기화하는 작업이 동시에 이뤄지야하기 때문입니다.

아래는 SQL → NoSQL 마이그레이션 Pattern 입니다.

#### 마이그레이션 패턴 순서(RDBMS → NoSQL)

RDBMS에서 NoSQL로 마이그레이션을 위해서는 아래와 같은 순서로 작업이 진행되며, 세부적인 방안 Mechanism을 통해 확인할 수 있습니다.

- Lift and Shifts 방식으로 OS, 데이터, 애플리케이션을 그대로 옮기는 작업을 진행
- 요구사항에 맞는 구성으로 설계 (Ex. On-promise 환경 → Cloud 환경)
- RDBMS의 쿼리결과를 Key-Value 로 변환
- Cold / Hot Data의 적재 및 조회 방안 구성
    - Cold Data : 덜 액세스되고 빠르게 액세스 할 필요가 없는 데이터를 의미합니다.
        - SELECT/UPDATE/DELETE 가 발생되지 않는 데이터
    - Hot Data : 즉시 액세스 해야하는 데이터를 의미합니다.
        - SELECT/UPDATE/DELETE가 발생되는 데이터

#### Mechanism

**Cold/Hot 데이터를 마이그레이션 방안**

운영중인 데이터베이스를 중단하지 않고 NoSQL로 이관하기 위해서는 Hot Data와 Cold Data를 구분하여 작업을 진행합니다.

![Migration Mechanism](/img/aws-reinvent/mechanisms.png)*마이그레이션 메카니즘*

- RDBMS에서 조회되는 쿼리 결과 → Key-Value 변환
- 사전에 UPDATE 작업 또는 신규 데이터는 NoSQL 데이터베이스로 이관
- 변경이 발생된 데이터(UPDATE/DELETE/INSERT) 및 Cold Data를 DMS를 이용하여 이관
- 마이그레이션 간의 Down Time 해소

#### READ 요청 Mechanism

- NoSQL 데이터베이스로 READ 요청
    - 운영중인 데이터베이스에 READ Request 요청하여 NoSQL에서 데이터를 반환
    - NoSQL에서 데이터가 없어 Return을 못하는 경우, RDBMS에 READ Request 요청하면 RDBMS Return 과 함께 해당 데이터를 NoSQL로 Write 작업 진행

#### Write 요청의 Mechanism

- NoSQL 데이터베이스로 Write 요청
    - NoSQL로 Write 작업 요청
    - RDBMS에는 해당 Write 작업 결과가 존재하는 확인
    - NoSQL에 데이터가 적재되지 않았다면, RDBMS → NoSQL로 데이터 적재

#### 변경 요청에 대한 Mechanism

- RDBMS와 변경건에 대한 DMS(Data Migration Service)를 이용하여 진행
    - 실시간 데이터 스트림 서비스인 Kinesis로 DMS로 연결
    - Lambda를 이용하여 변경건에 대한 Trigger 진행

해당 세션에서 배운 것은 Migration 진행 시 사용되는 SQL 쿼리와 그 영향을 전수조사하는 것이 첫 단추라는 점 입니다.
마이그레이션 작업 과정보다 작업 대상에 대한 데이터의 전수조사가 약 절반을 차지하며, 파이썬 코드를 이용하여 update,insert,delete(DML) 작업을 구분하여 Dual Write 작업을 통해 이관을 합니다.

단순한 코드 몇줄, 클릭 몇번을 통한 이관이 아니라 마이그레이션에 대한 전 과정에 대해 들어볼 수 있었던 의미있는 세션이였습니다.

### 2.4. 데이터베이스의 블루/그린 최적화된 배포

**데이터베이스 버전 업그레이드 방안**

Blue/Green Optimized 세션은 Aurora의 버전 업그레이드와 관련하여 AWS 발표한 새로운 기능에 대해 소개하는 세션입니다.

데이터베이스의 버전 업그레이드와 그 과정에서 발생하는 다운타임을 고려하는 것은 DBA 입장에서 매우 중요한 부분이라 개인적으로도 가장 듣고 싶었던 세션이었습니다.

AWS re:Invent 2022 에서 발표한 Blue/Green 최적화 배포 세션은 최대 1분 이내로 업그레이드 작업을 진행할 수 있으며, 
기존의 방식보다 안전하게 작업을 완료할 수 있다고 발표하였습니다.

다만, 생성된 Aurora RDS Instance 만큼 추가로 인스턴스가 생성되기 때문에 비용은 필수불가결하게 증가될 것으로 예상됩니다.

**[Blue/Green Deployment 프로세스]**

![Blue-green](/img/aws-reinvent/blue-green.png)*Blue-Green Deployment*

- Blue → Green 간의 Logical Replication (Blue 에서 발생하는 모든 변경 사항을 Green에서 미러링할 수 있도록 하는 작업)
    - 변경 복제를 이용한 여러 수준의 Replication 지원(One-Level 복제)
    - Blue와 Green 사이의 Replication을 설정하고 메트릭을 모니터링하여 모든 것이 최신상태인지 확인
- RDS 인스턴스 버전 업그레이드 완료 및 데이터 검증 후 Blue(업그레이드 이전) 인스턴스 삭제

해당 세션을 듣고 정말 1분 이내에 업그레이드 Switch Over를 완료할 수 있을지에 대해 의문이 들었습니다.
개발 계정에서 신규 Aurora MySQL RDS Instance를 생성하여 Blue/Green Deployment 테스트를 진행했습니다.

#### 테스트 진행 절차

- 신규 RDS Instance 생성(Aurora MySQL 5.7)
- 더미데이터 Import 수행 (5GB 데이터 적재)

![Blue-green-test](/img/aws-reinvent/test-blue-green.png)*Blue-Green Deployment Test*

- Blue/Green 설정
    - Blue - Aurora MySQL 5.7
    - Green - Aurora MySQL. 8.0
- 작업 중 New Connection 연결 시 Connection Error 미발생(작업완료 약 21분 소요, db.t3.medium)
    - Blue / Green 접속 및 커넥션 확인
- Blue의 데이터 변경 시, Green의 데이터 변경 확인 가능(DML/DDL)
    - 적재된 데이터 변경작업(DML) 진행하였으나 Green에도 정상적으로 반영 확인
- Switch Over를 이용하여 Green → Blue 배포 진행
- 읽기 작업은 Blue(운영)쪽에서 진행 가능
    - Read Workflow는 Blue에서 진행할 수 있다.
- Blue/Green 배포 작업은 약 1분 이내에 수행 (Major /Minor 버전 업그레이드 포함)
    - 내부적으로 binlog bin log를 기준으로 미러링 확인
    - Switch Over 시 Timeout Setting - 1분
        - 1분으로 설정했으나 1분이 초과되어 자동 Rollback
        - 무조건 지정한 시간 내에 Switch Over가 되지 않음
- Switch Over 시 Timeout Setting - 3분 설정
    - 정상적으로 Blue/Green Deployment 완료 확인

**Switch Over 결과**

![Blue-green-test-result](/img/aws-reinvent/test-result.png)*Blue/Green Deployment 완료 후 위와 같이 분리*

실제 작업은 위와 같은 방법으로 작업이 수행되며 db.t3.medium 기준으로 약 $91의 비용이 소요됩니다.

세션에서 들었던 것과 달리 Timeout을 1분으로 설정했을 때 Switchover 작업이 실패하였는데 정확한 원인은 파악하지 못하였습니다. 혹시 1분 기준으로 테스트를 성공하신 분이 있다면 댓글 부탁드립니다.

실제로 적용하기에는 비용적인 측면에서 이슈가 발생할 수 있으며, New Feature로 운영에 적용하기에는 어떤 버그가 발생할지 모르기 때문에 적용하기에는 시기상조라고 생각합니다.
하지만, 이러한 새로운 기능에 대해 알고 있는 것은 중요하다고 생각하며, AWS의 RDS가 점점 발전하고 있다는 것을 느낄 수 있던 세션이였습니다.

 

### 2.5. 오픈소스(Trino)를 사용하여 AWS S3 데이터 조회

**S3 데이터 조회의 한가지 방법**

이번에 들었던 세션은 Open-source인 Trino를 이용한 S3 데이터 접근방법에 관련된 세션입니다.
특히, Athena를 사용하신 분들에게 관심을 가져볼 수 있었던 세션입니다.

S3의 데이터를 조회하기 위해서는 기존에는 AWS가 제공하는 서비스인 Athena를 이용하여 데이터를 조회하여 쿼리 결과를 얻을 수 있습니다.

다만, Athena를 이용하여 쿼리 결과를 얻기 위해서는 서울 리전을 기준으로 1TB당 $5의 비용이 발생합니다.

하지만 이번 세션에서는 Trino를 이용하여 Athena의 조회 비용을 제거하고 쿼리성능을 개선하는 방법을 소개하고 있습니다.
단, EMR(Elastic MapReduce)의 Computing 비용을 지불합니다.

이러한 속도를 낼 수 있는 것은 Pushdown이라는 기능 때문입니다.
그 이유는 MySQL에서도 비슷한 기능이 존재하며 Index Condition Pushdown 과 유사하기 때문입니다.
스토리지에 Index를 밀어 넣어 필요한 데이터만을 그대로 가지고 메모리에 올려놓고 결과를 반환하는 과정과 유사합니다.

마찬가지로 S3 Select는 지정된 바이트 수 만큼 CSV 파일에서 Range Scan(범위)을 전체 객체 스캐닝을 병렬로 처리하여 속도가 빠릅니다.

![trino](/img/aws-reinvent/trino.png)*Trino*

아래는 S3 내 CSV 파일 데이터를 조회하는 속도를 보여 줄 수 있는 성능 그래프 입니다.

![trino-result](/img/aws-reinvent/trino-result.png)*Trino*

- S3 내 압축되어 있지 않은 3TB의 CSV 데이터를 조회하는 속도를 확인할 수 있습니다.

**Architecture**

![data-lake-architecture](/img/aws-reinvent/data-lake-architecture.png)*데이터 레이크 구조*

아키텍처는 Amazon EMR(Elastic MapReduce)에 Trino를 설치하여 S3를 연동하여 쿼리를 실행할 수 있는 SQL 쿼리 엔진입니다.

쏘카에서는 S3에 적재된 데이터를 Athena를 이용하여 조회를 하지만, 꼭 한가지 방법만 고수하여 사용하는 것보다는 여러가지의 대안을 알고 있어야한다고 생각합니다.
어떠한 이유로 Athena를 사용 못하게 되었을 경우, 대응 방안을 가지고 해결할 수 있어야하기 때문입니다.

데이터베이스에서 더 이상 사용되지 않는 정적 데이터는 S3로 이관하는 것을 지향합니다.
불필요한 데이터는 테이블을 무겁게하고 성능에도 영향을 미치며, 대량의 테이블에서는 ALTER 작업에도 CPU 부하를 유발할 수 있기 때문입니다.

테이블의 경량화를 위해 S3로 이관하고 사용 빈도 적은 데이터를 빠르게 조회할 수 있는 방법에 대해 알아볼 수 있었던 세션이었습니다.

---

## 3. AWS re:Invent 참석 후기

AWS의 서비스를 이용하면 실제로 서비스를 만드는 팀에서 발표를 들어볼 수 있다는 것이 좋았으며, 여러 세션 중 어떤 세션은 발표한 내용을 실습할 수 있는 시간도 주어진다는 점이 매력적이었습니다.
특히, 평소에 관심있는 분야에 대해 Deep Dive 할 수 있는 기회라고 생각되며, 세션을 듣고 Insight를 가져볼 수 있는 값진 시간이었습니다.

세션은 다소 진지하게 진행되지만, 엑스포에서는 수백개의 파트너 사가 AWS와 관련된 솔루션을 홍보하면서 즐길 수 있는 장소라고 생각이 들었습니다.
re:Invent 행사가 단순히 컨퍼런스의 개념만 가지고 있는 것이아니라 클라우드 기술을 통합하는 장소라는 것을 느껴볼 수 있었습니다.

마지막으로 AWS re:Invent 를 참석할 수 있는 기회를 주신 제이든과 로원에게 감사드리며, 일주일동안 값진 시간을 보낼 수 있도록 도움을 주신 Cloud 팀원분에게 감사인사를 드립니다. 
긴 글 읽어주셔서 감사합니다.

![aws-reinvent-outro](/img/aws-reinvent/aws-reinvent-outro.jpg)
