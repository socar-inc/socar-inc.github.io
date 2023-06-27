---
layout: post
title: "쏘카의 대규모 인증토큰 트래픽 대응 : 개발기"
subtitle: "어카운트팀의 대규모 트래픽 대응을 소개합니다."
date: 2023-06-27 09:00:00 +0900
category: dev
background: '/img/handling-authentication-token-traffic-01/handling-authentication-token-traffic-background.jpg'
author: oli
comments: true
tags:
    - Accounts
    - traffic
    - backend
    - developer
    - service engineering
---
  
  
  
  
안녕하세요. 어카운트 버킷의 올리입니다. 🫒

다가오는 여름, 쏘카 타고 어디론가 떠나고 싶으신가요? 🚘 맑고 한층 더워진 날씨에 쏘카와 함께 떠나고 싶은 분들을 맞이하기 위해 대규모 트래픽에 대응할 수 있도록 쏘카 서비스를 개선했습니다. 쏘카가 많은 손님 맞을 준비를 어떻게 했는지 이번 테크 블로그 글에 담았습니다.

쏘카 시스템의 모든 요청은 인증 토큰 검증이 이루어집니다. 트래픽이 낮을 때는 write DB 하나만 활용하는 게 적합한 선택이었지만 트래픽이 늘어나며 DB 부하가 급격하게 증가하는 문제가 발생했습니다. 또한, 개발 편의성을 위해 여러 시스템에서 DB에 직접 접근해 성능 문제와 함께 DB에 종속된 여러 문제가 발생했습니다. 

**쏘카의 대규모 트래픽 대응 프로젝트**는 write DB의 부하를 분산하여 응답 속도 개선을 목표로 세웠지만 달성 가능하다는 확신은 없었습니다. 기존 서비스에 영향을 주지 않고 변경을 최소화하며 목표를 달성하기는 매우 어렵기 때문입니다.

이처럼 기존 구현에 영향을 주지 않고 성능 개선을 해야 하는 상황이라면 이번 글이 도움이 될 것입니다.
  
  
  
---
  
## 목차 소개 
  
이번 글은 다음과 같은 순서로 진행됩니다.
  
1. [기존 프로젝트에서 인증토큰 관련 로직 분리](#1-기존-프로젝트에서-인증토큰-관련-로직-분리)
2. [DB 부하 분산](#2-DB-부하-분산)
3. [인증 토큰 테이블에서 유효하지 않은 토큰 분리](#3-인증-토큰-테이블에서-유효하지-않은-토큰-분리)
4. [cache layer 적용](#4-cache-layer-적용)
  
  
  
---
  
## 1. 기존 프로젝트에서 인증토큰 관련 로직 분리 
  
### **문제 상황**
  
인증 검증 로직의 현황을 파악해 보니, 인증 토큰을 처리하는 메인 서비스, php로 이루어진 레거시 서비스 3개, 그 외의 레거시 서비스가 모두 데이터 베이스에 접근하고 있었습니다. 즉, 공통된 기능이 여러 프로젝트에 분산되어 설정값 하나를 변경하려고 해도 여러 프로젝트에 변경사항을 반복 적용해야 합니다.
  
![[그림] - 기존 아키텍처 ](/img/handling-authentication-token-traffic-01/handling-authentication-token-traffic-01.png)*[그림] - 기존 아키텍처*
  
이런 환경에서 DB의 연결 설정을 변경하면 어떻게 될까요? 인증 토큰과 관련된 모든 프로젝트를 파악하여 같은 작업을 반복해야 합니다. 이러한 환경은 개발 내용과 직접 관련이 없음에도 영향받는 모든 프로젝트 파악이 필요해 리소스가 반복적으로 투입됩니다. 분산된 여러 프로젝트를 대상으로 중복 작업을 진행하다 실수로 하나라도 빼먹게 되면, 변경점이 적용되지 않아 서비스 장애가 발생할 수 있습니다. 
  
### **문제 해결 방안 찾기 - 인증 검증 전용 서비스 도입**
  
DB 접근을 담당하는 하나의 계정서비스를 두어, 인증 토큰 작업이 필요한 개별 서비스는 계정 서비스로 요청만 보내면 어떨까요? 각각의 서비스는 DB 접근 설정 변경이나 코드 변경에 관심 둘 필요 없이 필요에 따라 인증처리를 위임하고 세부사항은 계정 서비스가 관리할 수 있습니다. 
  
따라서, 계정 서비스가 인증 토큰 관련 로직을 담당하면 각각의 서비스는 변경작업이 필요 없어집니다.
![[그림] - 개선된 아키텍쳐](/img/handling-authentication-token-traffic-01/handling-authentication-token-traffic-02.svg)*[그림] - 개선된 아키텍쳐*
  
### **결과**
  
DB 부하 분산과 성능 개선을 위해 아키텍처 개선을 우선 수행했습니다. 이로 인해 분산된 인증처리를 하나의 전담 로직이 처리하도록 변경하여 불필요한 반복작업을 제거했고 개발자의 생산성을 높일 수 있었습니다.
  
  
  
---
  
## 2. DB 부하 분산  
  
write DB에 집중된 인증 토큰 관련 로직 문제를 해결하여 DB 부하를 분산시킨 방법을 설명하겠습니다.
  
### **문제 상황**
  
write DB와 read only DB가 별도로 존재했지만, 이를 호출하는 서비스에서 write DB로만 요청을 보내고 있어 단순 조회 시에도 모든 부하가 write DB로 몰리고 있었습니다. 
  
동시간 대의 write DB의 인증토큰 조회 요청 수와 read DB의 인증토큰 조회 요청 수는 다음과 같습니다.
  
![[그림] - write DB의 요청량](/img/handling-authentication-token-traffic-01/handling-authentication-token-traffic-03.png)*[그림] - write DB의 요청량* | ![[그림] - read DB의 요청량](/img/handling-authentication-token-traffic-01/handling-authentication-token-traffic-04.png)*[그림] - read DB의 요청량*
---|---|

  
모든 요청이 write DB에 집중되는 것을 확인할 수 있습니다. 
  
### **문제 해결 방안 찾기 - DB 부하 개선**
  
DB 부하 문제는 write DB를 scale up하여 해결할 수 있습니다. 하지만 이 방법은 일시적인 서비스 중단을 피할 수 없고, 이미 고사양 장비를 사용하고 있어 적합한 해결책이 아니라고 판단했습니다.
  
![[그림] - scale up](/img/handling-authentication-token-traffic-01/handling-authentication-token-traffic-05.png)*[그림] - scale up*
  
따라서 scale out을 통하여 slave DB로 read 요청에 대한 부하를 분산시키고 트래픽이 늘어나면 read DB를 늘리는 해결책을 선택했습니다. 
  
![[그림] - scale out](/img/handling-authentication-token-traffic-01/handling-authentication-token-traffic-06.png)*[그림] - scale out*
  
### **문제 해결 방안 적용 - DB 부하 분산 적용 방법**
  
write, read DB 부하 분산 방법에는 LazyConnectionDataSourceProxy가 자주 사용됩니다.
  
LazyConnectionDataSourceProxy을 사용하면 트랜잭션 진입 시점에 미리 커넥션을 결정하지 않고 사용하는 시점에 커넥션을 설정합니다. 이런 특성을 이용하여 다음의 예시 코드와 같이 코드 레벨에서 트랜잭션이 read only라면 read DB 커넥션을 사용하도록 적용할 수 있습니다.
  
```kotlin
class DynamicRoutingRoutingDataSource : LazyConnectionDataSourceProxy() {
    lateinit var writeDataSource: DataSource
    lateinit var readDataSource: DataSource

    override fun getTargetDataSource(): DataSource {
        return if (TransactionSynchronizationManager.isCurrentTransactionReadOnly()) {
            readDataSource
        } else {
            writeDataSource
        }
    }
}
```
  
maria connector 2.7 버전에서 지원하는 aurora protocol을 사용하는 방법도 있습니다. read 클러스터 endpoint 추가만으로 driver 레벨에서 write, read DB 부하 분산이 가능하고 failOver까지 지원해 코드 변경을 최소화할 수 있습니다.
  
jdbc:mariadb:aurora// **< write 클러스터 endpoint >** , **< read 클러스터 endpoint >** 의 형식으로 write 클러스터의 endpoint와 read 클러스터의 endpoint를 콤마로 구분하여 입력합니다. 
  
> 💡 단, 이 방법은 mariaConnector 3.x 이후 버전에선 지원하지 않습니다. 
  
```yaml
spring:
  datasource:
    url: jdbc:mariadb:aurora/< write 클러스터 endpoint >,< read 클러스터 endpoint >
    hikari:
      driver-class-name: org.mariadb.jdbc.Driver
```
이번 프로젝트는 단기간에 목표를 달성해야 하는 제약사항을 고려하여 코드 변경의 최소화가 필요했고, 새로 만든 계정 서비스는 aurora protocol 을 지원하는 버전을 사용하고 있어 이 방법이 가장 적합하다 판단했습니다.
  
### **결과**
  
write, read DB 부하 분산 적용 후 인증 토큰 조회 요청 수는 다음과 같습니다.
  
![[그림] - write DB의 요청수](/img/handling-authentication-token-traffic-01/handling-authentication-token-traffic-07.png)*[그림] - write DB의 요청수* | ![[그림] - read DB의 요청수](/img/handling-authentication-token-traffic-01/handling-authentication-token-traffic-08.png)*[그림] - read DB의 요청수*
---|---|
  
![[그림] - write DB와 read DB의 인증 토큰 조회 요청 수](/img/handling-authentication-token-traffic-01/handling-authentication-token-traffic-09.png)*[그림] - write DB와 read DB의 인증 토큰 조회 요청 수*
  
적용 이후 read DB로 부하가 분산된 것을 확인할 수 있습니다.
  
  
    
---
  
## 3. 인증 토큰 테이블에서 유효하지 않은 토큰 분리
**부제. access token table compaction**
  
### **문제상황**
  
MySQL과 같은 InnoDB의 인덱스는 B+tree 구조로 최신의 정렬 상태를 유지하기 때문에 log^2n의 시간 복잡도로 빠른 탐색이 가능합니다. 하지만 B+tree 구조에서 인덱스 키값을 삽입할 때는 인덱스 값 업데이트 비용 및 트리 구조 재구성 비용이 추가됩니다. 
  
쏘카의 인증 토큰 관리 테이블에서는 인증 토큰 테이블에 만료된 인증 토큰까지 보관했기 때문에 현재 인증된 회원 수에 비해 훨씬 많은 수인 약 2800만 개에 달하는 데이터를 가지고 있었습니다. 인증 토큰 테이블은 인덱스를 이용하여 빠른 탐색이 가능하도록 설계됐지만, 만료된 데이터까지 보관하고 있어 데이터 양이 많아져 데이터 삽입,삭제,수정 작업 시 인덱스 재구성 및 트리 구조 재구성 비용이 증가했습니다. 
  
데이터 쓰기 작업은 배타적인 접근을 위해 락(잠금)을 사용하는데, 작업시간이 증가하여 락 경합으로 인해 조회 작업도 락을 획득하기 위해 기다려야 하는 문제가 발생했습니다. 결과적으로 시간이 오래 걸리고, DB에 부하가 증가했습니다.
  
### **문제 해결 방안 찾기**
  
3가지의 대처 방안을 고려해 보았습니다. 
  
+ 1) 기존 방식 그대로 인증토큰 테이블에 만료 토큰을 보관  
+ 2) 만료된 인증토큰을 완전히 삭제  
+ 3) 만료된 인증토큰을 분리 보관하고 유효한 토큰만 인증토큰 테이블에 보관  
  
먼저 기존의 방식대로 하나의 테이블에서 만료된 인증토큰을 함께 보관하는 방안입니다. 처리 목적에 비해 불필요한 데이터가 많이 존재하고, 테이블의 index 또한 복잡하게 구성되어 쓰기 작업의 수행 시간의 증가 문제와 이로 인한 읽기 작업 지연 문제가 있습니다. 부하 분산을 통해 일시적으로 문제를 해결할 순 있지만 근본적인 개선이 아니라고 판단했습니다.
  
다음으로, 만료된 인증 토큰을 모두 삭제하는 방안입니다. 만료된 토큰이 고객센터에서 고객의 인증 이력을 확인하는 용도로 사용되었고, 만료된 토큰 삭제 영향도 범위를 파악하는데 많은 시간이 소요될 것으로 예상되었습니다. 프로젝트기한이 촉박해, 영향도 파악에 리소스를 사용하기 보다 원래 목적인 DB 부하 분산에 집중하기 위해서 두번째 방안은 택하지 않았습니다. 
  
따라서, 만료된 인증 토큰을 별도로 분리 보관하는 전략을 채택했습니다. 인증 토큰 테이블에는 시스템 관심 대상인 유효한 인증 토큰 데이터만 유지하고, 운영 관점에서 필요한 이력은 별도 테이블로 분리하여 기존에 발생한 성능 문제를 해결할 수 있을 것으로 예상했습니다. 이렇게 시스템에서 분리된 데이터는 추후 별도 저장소에 구성하여 마스터 DB의 용량을 절약할 수 있기에 적합한 선택지라 판단했습니다. 
  
### **문제 해결 방안 적용**
  
만료 토큰을 분리하는 작업은 많은 양의 데이터 이동이 필요하여 배치를 이용했습니다.
  
![[그림] - token compaction batch architecture](/img/handling-authentication-token-traffic-01/handling-authentication-token-traffic-10.png)*[그림] - token compaction batch architecture*
  
만료된 인증 토큰을 인증토큰 테이블에서 분리하는 작업은 2단계로 진행했습니다. 
  
| <center>단계</center> | <center>설명</center> |
| --- | --- |
| **Step 1** | 만료 인증 토큰 테이블을 신규 생성, 1회 성 배치를 통해서 현재 인증 토큰 테이블의 만료된 인증 토큰을 이동, 인증 토큰 테이블의 만료 인증 토큰 삭제  |
| **Step 2** | 한 시간 단위의 배치를 통해 만료된 인증 토큰을 인증토큰 테이블에서 만료 인증 토큰 테이블로 이동, 인증 토큰 테이블의 만료 인증 토큰 삭제  |
  
Step1은 1회 성 배치로 진행했습니다. 인증토큰 테이블에 보관한 만료된 인증 토큰을 만료 인증 토큰 테이블로 이동시키고 기존 인증 토큰 테이블에서 삭제했습니다. 이때, 한 번에 모든 토큰을 새로운 테이블에 저장하고 인증 토큰 테이블에서 삭제하는 작업이 이루어지면 데이터 베이스의 리소스를 많이 사용하여 cpu가 증가하고 부하가 발생할 수 있습니다. 이를 방지하기 위해서 5일에 걸쳐 배치 1회마다 인증 토큰의 개수를 450개로 제한하여 작업을 진행했습니다. 
  
Step1 이후부터는 1시간 단위의 만료 토큰 이동 배치를 설정하여 지속적으로 만료된 토큰을 분리보관했습니다. 이를 통해 인증 토큰 테이블에 불필요한 데이터가 적재되지 않도록 유지했습니다. 
  
### **결과**
  
Step1 배치가 종료된 이후, 인증 토큰의 테이블 row 개수가 약 86% 감소한 2백만 개로 줄었습니다. 이후에도 만료된 토큰을 1시간마다 이동시켜 유효한 토큰만 유지하여 문제를 해결했습니다.
  
![[그림] - 만료된 인증 토큰 분리 후 인증 토큰 테이블 데이터 개수](/img/handling-authentication-token-traffic-01/handling-authentication-token-traffic-11.png)*[그림] - 만료된 인증 토큰 분리 후 인증 토큰 테이블 데이터 개수*
  
  
  
---
  
## 4. redis cache layer 적용  
  
### **문제상황**
  
쏘카 시스템의 모든 요청은 인증 토큰 검증이 필요하여 인증 토큰의 생성, 수정, 만료 요청 보다 조회 요청이 압도적으로 많았습니다. 1단계 작업을 통해 쓰기 작업과 읽기 작업을 분리하여 read DB의 부하는 분산시켰지만, 전체 요청 수는 동일하여 트래픽 증가함에 따라 인증 토큰 검증 요청으로 인한 slave DB ****성능 이슈가 발생할 수 있습니다. 여기서 slave DB로 들어오는 read 요청 수까지 감소시킬 수 있다면 어떨까요? 
  
### **문제 해결 방안 찾기 - redis cache layer**
  
매 요청마다 DB가 직접 토큰 검증 요청을 처리하지 않도록 캐시를 이용해서 DB의 부하를 줄일 수 있습니다. 
  
여기서 캐시란 무엇일까요?
  
![[그림] - 캐시 설명](/img/handling-authentication-token-traffic-01/handling-authentication-token-traffic-12.png)*[그림] - 캐시 설명*
  
캐시는 반복적인 데이터를 불러오는 경우에 자주 사용하는 데이터를 미리 복사해 저장하는 임시 장소로서 DB 접근을 줄여 부하를 줄일 수 있습니다. 또한, 메모리에 데이터를 저장하고, key-value 저장 구조라 시간 복잡도가 o(1)으로 빠른 탐색이 가능해 데이터 조회 성능이 개선됩니다.
  
### **문제 해결 방안 적용 - redis cache layer 캐시 전략**
  
**redis cache layer 캐시 전략** 
  
캐시는 key-value의 저장 방식으로 변동성 여부, 데이터 적합성 등에 따라 캐시 전략을 선택할 수 있습니다. 
  
**look aside 전략** 
look aside 전략은 사용자의 조회 요청을 받으면 1) 캐시 스토어에 데이터 존재 여부를 확인하고 존재하면 데이터를 반환(cache hit) 합니다. 2) 캐시 스토어에 데이터가 없다면 DB에서 조회하여 반환(cache miss) 합니다. 3) DB에서 조회해온 데이터를 캐시 스토어에 업데이트하는 방식으로 반복적인 호출이 많은 경우 적합한 전략입니다.
![[그림] - look aside 캐시 전략](/img/handling-authentication-token-traffic-01/handling-authentication-token-traffic-13.png)*[그림] - look aside 캐시 전략*
  
**read through 패턴** 
read through 패턴은 1) 캐시 스토어에 데이터 존재 여부를 확인하고 존재하면 데이터를 반환(cache hit) 합니다. 2) 캐시 스토어에 데이터가 없으면 DB에서 조회하여 반환(cache miss) 합니다. 3) DB에서 직접 캐시 스토어에 업데이트하는 방식으로 look aside와 비슷하지만 캐시 스토어에 저장하는 주체가 서버가 아닌 DB에 있다는 차이가 있습니다.
![[그림] - read through 캐시 전략](/img/handling-authentication-token-traffic-01/handling-authentication-token-traffic-14.png)*[그림] - read through 캐시 전략*
  
**write back 패턴** 
write back 패턴은 1) 캐시가 큐의 역할을 수행하며 모든 데이터를 캐시 스토어에 저장하고 2) 일정 시간이 지난 후 DB에 반영하는 방식입니다. 데이터 적합성이 높고 DB 요청 수를 줄일 수 있지만 캐시에 장애가 발생할 경우 데이터가 유실되며 사용하지 않는 정보까지 캐시에 저장하는 단점이 있습니다.
![[그림] - write back 캐시 전략](/img/handling-authentication-token-traffic-01/handling-authentication-token-traffic-15.png)*[그림] - write back 캐시 전략*
  
**write through 패턴** 
write through 패턴은 DB에 요청되는 모든 데이터가 캐시 스토어를 통해서 수행하는 방식으로 캐시가 주 DB의 역할을 수행합니다. 모든 요청이 캐시를 통해 이루어지기 때문에 데이터 적합성이 높고 캐시와 DB에 모든 변경사항이 반영되어 데이터 유실 가능성이 낮지만, 캐시와 DB에 모두 작업이 이루어져야 하기 때문에 상대적으로 느리고 사용하지 않는 정보도 캐시에 저장하는 단점이 있습니다.
![[그림] - wrtie through 캐시 전략](/img/handling-authentication-token-traffic-01/handling-authentication-token-traffic-16.png)*[그림] - wrtie through 캐시 전략*
  
**write around 패턴** 
write around 패턴은 쓰기 작업은 캐시를 거치지 않고 DB에만 진행하는 방식으로 cache miss가 발생하는 데이터만 캐시에 저장합니다. 데이터 적합성의 문제가 발생할 수 있지만, 빠르며 쓰기 작업이 많지 않은 경우에 사용됩니다. 
  
**캐시 전략 선택** 
인증 토큰 조회 요청은 token 값을 이용해서 인증 정보를 요청하는 경우와 멤버의 id를 이용해  인증토큰 정보들을 요청하는 경우 2가지로 구분됩니다. 모든 요청에 캐시를 적용하면 캐시 저장 key의 값이 2개로 읽기를 제외한 쓰기, 수정, 삭제 작업 시에 캐시 정보를 수정하는 작업을 병행하면 데이터 적합성에 문제가 발생할 수 있습니다. 이에 더해 쓰기 요청 보다 조회 요청이 압도적으로 높은 인증토큰 로직의 특성을 고려하여 쓰기 작업 DB 요청 수를 줄이는데 집중하기 보다 데이터의 불일치 정보를 해소하는 것이 더 중요하다 판단했습니다. 
  
발생 가능한 문제를 최소화하기 위해 계정 api의 인증 토큰 캐시 전략은 Look Aside + Write Around 조합을 선택했습니다. write around 방식에서 발생할 수 있는 쓰기 작업 데이터 적합성 문제 해결을 위해 데이터 쓰기 작업 시 캐시 정보를 수정하는 대신 삭제하는 방식을 도입했습니다.
  
이를 통해 조회 요청 시에는 캐시를 먼저 탐색하여 DB 데이터 조회 요청 수는 감소시키고 데이터 쓰기 작업 시 캐시를 삭제하고 이후 조회할 때 다시 캐시에 적재해 데이터 적합성 문제를 최소화했습니다. 
  
### **결과**
  
redis cache 적용해 slave DB의 부하를 줄였습니다.
  
![[그림] -  redis cache 적용 후 DB 요청 수](/img/handling-authentication-token-traffic-01/handling-authentication-token-traffic-17.png)*[그림] -  redis cache 적용 후 DB 요청 수*
  
![[그림] -  redis cache 적용 후 write DB 와 read DB 요청 수](/img/handling-authentication-token-traffic-01/handling-authentication-token-traffic-18.png)*[그림] -  redis cache 적용 후 write DB 와 read DB 요청 수*
  
또한, 캐시를 이용해 요청에 빠르게 응답하여 결과적으로 latency를 줄였습니다. 
  
![[그림] -  redis cache 적용 후 계정 서비스 latency](/img/handling-authentication-token-traffic-01/handling-authentication-token-traffic-19.png)*[그림] -  redis cache 적용 후 계정 서비스 latency*
  
  
  
---
  
## 마무리
  
쏘카의 대규모 트래픽 대응 프로젝트는 인증 토큰 검증으로 인한 부하 최소화라는 큰 목표를 가지고 진행한 프로젝트로 이번 프로젝트를 통해 다음의 개선을 이뤘습니다. 
  
- 여러 서버로 분산된 중복 기능을 하나의 시스템이 담당하여 변경에 용이하도록 개선
- write DB와 read DB의 부하 분산을 적용을 통해 특정 DB의 과부하 문제 해결
- 데이터 테이블의 불필요한 데이터 제거를 통한 조회 성능 개선
- 캐시 적용을 통한 read DB의 부하 감소 및 응답 속도 개선
  
프로젝트를 진행하면서 여러 이벤트로 인해 초반 예정되어 있던 2개월의 개발 기간을 1개월 남짓한 기간으로 줄이며 짧은 기간 안에 여러 단계의 작업과 단계별 QA까지 완료해야 했습니다. 힘든 일정이었지만 배포 후 눈에 띄는 결과를 보며 더 힘을 내어 끝까지 마무리할 수 있었습니다. 또한, 성수기 트래픽 대응이라는 큰 목표에 도달하기 위해 단계별로 접근하여 기존 서비스 영향은 최소화하며 작은 단위의 문제에 집중하고 정복하는 과정을 배울 수 있었습니다. 이를 통해 앞으로 다가올 문제 또한, 분할 정복하여 목표를 달성할 수 있을 것이라 기대합니다. 
  
어카운트 팀은 마지막 단계 이후에도 성능 테스트를 계속하며 개선하기 위해 치열한 고민을 이어가고 있습니다. 이런 저희의 비전에 흥미를 느낀다면 쏘카 개발자로 지원해 주세요!