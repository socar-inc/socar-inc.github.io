---
layout: post
title: "싱글벙글 회원 서비스 성능 튜닝기"
subtitle: "어카운트팀이 진행하는 서비스 성능 개선 과정에 대해 소개합니다"
date: 2023-07-22 11:01:00 +0900
category: dev
background: '/img/member-service-performance-tuning-01/background.jpg'
author: choa
comments: true
tags:
  - Accounts
  - service engineering
  - performance tuning
---


더욱 활기차게 찾아온 이번 여름, 여러분의 여행지는 어디인가요? 혹시 쏘카와 함께 그곳으로 떠나볼 계획인가요? 🚘 코로나19로 어려움을 겪던 시기를 점차 이겨나가는 지금 여러분이 쏘카를 이용할 때 더 나은 경험을 제공하기 위해 열심히 노력하고 있습니다. 특히, 이동 수요가 급증하고 KTX, 숙박, B2B 등 다양한 사업 영역으로 확장하며 쏘카 시스템은 이전보다 훨씬 많은 트래픽을 처리해야 합니다. 성능을 개선하여 비즈니스 확장을 지원하는 새로운 도전이 필요한 상황이 됐습니다.

대규모 트래픽에도 서비스 안정성을 보장한 방법을 이번 테크 블로그에 자세히 소개하겠습니다. 기존 서비스에 영향을 미치지 않고 성능을 개선하는 방법에 대해 깊이 있게 다룹니다.

여러분의 서비스가 대규모 트래픽에 성공적으로 대응하는데 이 글이 조금이라도 도움이 되길 바랍니다. 함께 성장하는 IT 생태계를 위해 경험을 공유합니다.

---

## 발단

성수기를 맞아 어카운트 팀은 대대적인 개편을 진행했습니다. 하나로 구성된 거대한 시스템에서 [인증 담당 서비스를 분리하고](https://tech.socarcorp.kr/dev/2023/07/07/handling-authentication-token-traffic-02.html) [R/W 트래픽을 분산하며 DB 부하를 감소 시키고, 성능 향상을 위해 캐시를 도입하는 등](https://tech.socarcorp.kr/dev/2023/06/27/handling-authentication-token-traffic-01.html)  다가올 성수기와 신규 서비스 런칭을 안정적으로 뒷받침하기 위해 많은 노력을 기울였습니다.

그렇게 정신없는 날들을 보내고 대대적인 프로모션을 진행한 첫날, 예상치 못한 메시지를 받았습니다.

![[그림] - 느려요](/img/member-service-performance-tuning-01/slow.png)*[그림] - 느려요*


느리다?! 와우 이벤트가 대박났구나!! 🎉

저희는 환호성을 질렀습니다. 충분한 개선을 했기에 서비스가 느려진다는 것은 예상보다 많은 트래픽이 몰려오는 것으로 해석했습니다. 정말 그랬을까요?

![[그림] - 이벤트 당일 트래픽](/img/member-service-performance-tuning-01/datadog-2.png)*[그림] - 이벤트 당일 트래픽*

이벤트와 함께 트래픽이 늘어났지만 예상과는 달리 응답 시간이 더 많이 증가했습니다. 트래픽은 평소 대비 약 150% 증가했지만 응답 시간은 1000% 이상 증가했기에 서둘러 문제를 살펴봤습니다.

## 분석

간략한 시스템 구성은 아래 그림과 같습니다.

![[그림] - 서비스 구성도 ](/img/member-service-performance-tuning-01/service-3.png)*[그림] - 서비스 구성도*

쏘카 서비스는 인증된 회원 정보를 회원 서비스에 요청합니다. 회원 서비스는 인증 서비스에 인증 정보를 요청해 취합한 결과를 반환합니다.

각 쏘카 서비스 및 인증 서비스는 성수기를 대비해 대대적인 개편을 거쳤지만 회원 서비스는 문제가 없다는 판단애 개선 대상에 포함하지 않았습니다. 하지만 이 서비스에서 문제가 발생했습니다.

~~왜 슬픈 예감은 틀린적이 없나~~

서둘러 회원 서비스의 코드를 살펴봤습니다.

```kotlin
@Transactional
fun findByAccessToken(accessToken: String): MemberDto {
    val foundAccessToken = accessTokenClient
        .findByTokenIsAndExpiredAtIsNull(accessToken)
        .orElseThrow {
            InvalidTokenException("Access Token not found)")
        }

    val foundMember = memberRepository
        .findById(
            foundAccessToken.memberId!!.toInt()
        ).orElseThrow {
            MemberNotFoundException("Member Not Found")
        }

    return MemberConverter.fromMemberToMemberDto(foundMember)
}
```

[코드] - 회원 조회 코드

어떤 느낌을 받았나요? 이 코드를 처음 보고 큰 충격을 받았습니다. 손대기 싫을 만큼 읽기 어렵고 복잡한 코드를 상상했는데 코드는 너무 깔끔하고 이해하기 쉬웠습니다. 흔하게 볼 수 있는 유형의 코드인데 선언부를 포함해도 20줄 미만의 이 간단한 코드에 뭐가 숨어 있길래 성능 문제가 발생했을까요? 하나씩 살펴보며 문제를 진단했습니다.

### Write Transaction

JPA가 대중화되며 JPA가 제공하는 다양한 기능을 이용해 개발자 생산성과 애플리케이션 성능을 향상시킬 수 있었습니다. 대표적으로 영속 컨텍스트(persistent context)는 1차 캐시부터 동일성 보장, 쓰기 지연(transactional write-behind), 변경 감지, 지연 로딩이라는 멋진 기능들을 제공합니다. 하지만 이 기능은 언제나 매력적일까요? 안타깝게도 대부분의 기능은 '쓰기' 작업과 관련된 기능으로 '읽기'만 수행하는 메소드에는 불필요한 기능입니다. 지연 로딩 정도가 읽기에 도움되지만 사용하지 않고 있었습니다.

알았으니 개선해 봐야겠죠?

```kotlin
@Transactional(readOnly = true) //<- 1 차 개선
fun findByAccessToken(accessToken: String): MemberDto {
    //.. 중략..

    return MemberConverter.fromMemberToMemberDto(foundMember)
}
```

[코드] - readOnly transaction 적용

읽기만 수행하는 메소드라면 readOnly transaction이 좋은 대안입니다. 이 설정 하나로 앞서 언급한 변경 감지, 세션 플러시, 원본 스냅샷 생성은 모두 비활성화됩니다. 또한 DB 부하 분산이 필요할 때 Master/Slave DB를 선택할 수 있는 가장 적절한 기준이 되어 필요에 따라 DB 부하 분산을 쉽게 적용할 수 있습니다.

지연 읽기 기능은 readOnly transaction에도 활용할 수 있어 사이드이펙트(side effect) 걱정없이 개선을 마무리 할 수 있었습니다.

### 비효율적인 DB Connection 활용

시스템은 언제 db connection을 확보하고 반납할까요? auto-commit mode 에 따라 차이가 있지만 저희 시스템은 아래와 같이 동작했습니다.

```kotlin
@Transactional(readOnly = true) // <- DB 커넥션 획득
fun findByAccessToken(accessToken: String): MemberDto {
    val foundAccessToken = accessTokenClient // <- 외부 api 호출
        .findByTokenIsAndExpiredAtIsNull(accessToken)
        .orElseThrow {
            InvalidTokenException("Access Token not found)")
        }

    val foundMember = memberRepository // <- DB 커넥션 사용
        .findById(
        foundAccessToken.memberId!!.toInt()
    ).orElseThrow {
        MemberNotFoundException("Member Not Found")
    }

    return MemberConverter.fromMemberToMemberDto(foundMember)
} // <- DB 커넥션 반납
```

[코드] - DB Connection 활용

auto-commit 설정을 하지 않았다면 기본값은 true입니다. 이 상태로 애플리케이션을 운영하면 DB connection은 transaction 시작 시점에 획득하게 됩니다. 이것은 어떤 문제를 야기할까요? 원격지에 존재하는 시스템과 통신이 필요할 때 Connection Pool 기법을 활용합니다. Network I/O는 많은 비용이 들어가기 때문에 Connection을 미리 맺어두고 재활용합니다. 그러나 무한정 맺을 수 없기에 시스템 자원사정을 고려해 적절하게 생성하고 관리합니다. 여러 스레드(Thread)가 공유하는 자원이기에 꼭 필요할 때만 획득하고 사용 목적을 달성하면 빠르게 반납해야 합니다. 그러나 위 코드는 그런 관점에서 보면 효율적이지 않습니다.

DB 연결을 획득한 후 DB와 무관한 외부 API 호출 및 DTO 변환 작업이 포함돼 있어 빠르게 사용하고 반납할 수 없습니다. 이로 인해 사용자 요청에 대응하기 위해 많은 스레드가 연결 획득을 위해 대기하게 됩니다.

이럴 때 두 가지 선택지가 있습니다.

필요한 만큼 DB Connection을 생성하는 것과 DB Connection을 빠르게 사용하고 반납하는 것입니다. 어떤 것이 근본적인 개선일까요? 애플리케이션이 구동되는 Server 환경에 따라 다를 수 있으나 많은 경우 DB Connection을 늘리는 것은 더 심각한 문제를 유발합니다. 한정된 자원을 두고 Thread 끼리 경합하거나 많은 Thread를 적은 Core가 제어하느라 오히려 성능이 떨어지는 현상이 발생하죠. 따라서 DB Connection이 부족해 Application 성능 문제가 생겼을 때 DB Connection을 늘리는 것보다 빠르게 사용하고 반납하게 개선하는 것이 더 근본적인 개선 방향입니다.

위 코드는 회원 정보를 조회하는 memberRepository.findById를 제외한 나머지는 모두 DB와 무관한 코드입니다. 따라서 불필요하게 설정된 Transaction scope을 줄이면 DB Connection 획득 시점과 사용/반납 시점을 최대한 가깝게 유지해 효율적으로 DB Connection을 사용할 수 있어 아래와 같이 개선했습니다.

```kotlin
//<- 불필요한 Transactional 설정 제거
fun findByAccessToken(accessToken: String): MemberDto {
    //..중략..
    return MemberConverter.fromMemberToMemberDto(foundMember)
} 
```

[코드] - transaction 설정이 제거된 member service

```kotlin
@Transactional(readOnly = true) //<- Transactional 추가
    @Repository
    interface MemberRepository : JpaRepository<Member, Int> {
        fun findByUserid(email: String?): Optional<Member>
        fun findByEmail(email: String): Optional<Member>
    }
```

[코드] - transaction 이 추가된 member repository

findAccessToken 메소드는 Transaction 필요하지 않은 메소드 입니다. Transaction을 처리하는 범위를 repository로 한정해 connection을 효율적으로 활용하도록 개선했습니다.

### 무분별한 라이브러리 사용

세 번째로 살펴볼 내용은 memberEntity를 memberDto로 변환하는 코드 입니다. 내부는 다음과 같습니다.

```kotlin
fun fromMemberToMemberDto(member: Member): MemberDto {
    return getObjectMapper().readValue(
        getObjectMapper().writeValueAsString(member),
        MemberDto::class.java
    )
}
```

[코드] - memberEntity 를 memberDto 로 변환하는 코드

이 코드는 간단해 보이지만 큰 성능 문제를 갖고 있습니다. 절차를 살펴보면 다음과 같습니다:

- 객체를 JsonString으로 변환 (리플렉션을 이용)
- JsonString을 객체로 변환 (리플렉션을 이용)

여기서 ObjectMapper가 변환 역할을 담당합니다.

이 모든 과정은 리플렉션을 통해 이루어지는데 두 가지 중요한 문제가 있습니다.

첫 번째로 JIT(Just-In-Time) 컴파일러의 최적화를 제대로 활용할 수 없습니다. JIT 컴파일러는 실행 시간을 단축하기 위해 다양한 최적화 전략을 사용하는데 이 중 하나가 정적 타입 분석을 통한 메소드 인라이닝입니다. 메서드 인라이닝은 호출 비용을 줄이고 컴파일러가 더 많은 컨텍스트를 파악하여 추가적인 최적화를 합니다.

그러나 리플렉션을 사용하면 코드의 정적 구조는 런타임에만 완전히 파악할 수 있어 컴파일러가 최적화를 수행하기 어렵게 됩니다. 결국 리플렉션을 사용하는 코드가 더 느리게 실행됩니다.

두 번째로 리플렉션으로 로딩되는 추가적인 클래스 정보와 생성되는 JsonString이 모두 힙(Heap)에 할당됩니다. 이들이 생성되고 DTO가 만들어진 후에는 참조가 사라져 GC(Garbage Collection)의 대상이 됩니다. 즉, DTO를 생성하기 위해 클래스 정보가 매번 런타임에 로딩되고 JsonString이 생성되면 빈번한 GC가 성능을 저하 시킵니다.

이 코드는 ‘쉬운 문제는 쉽게 해결하라’ 는 원칙을 적용해 아래와 같이 개선했습니다.

```kotlin
fun fromMemberToMemberDto(member: Member): MemberDto {
    return MemberDto(
        id = member.id,
        createdAt = member.createdAt,
        updatedAt = member.updatedAt,
        userid = member.userid,
        //... 중략
    )
}
```

[코드] - 쉬운 문제는 쉽게 해결하자

단순한 방법이지만 효과는 큽니다. 불필요한 정보를 메모리에 로딩하지 않고 JIT 컴파일러 도움을 받아 최적화 코드를 생성합니다.

### 과하게 생성된 WAS Thread Pool

쏘카는 대부분 애플리케이션을 AWS EKS 환경에서 운영합니다. 이런 환경 덕분에 트래픽에 따라 시스템을 유동적으로 구성하고 필요에 따라 자원을 늘리거나 (scale-out) 줄이면서 (scale-in) 효율적으로 자원을 활용합니다. 하지만 안타깝게도 어카운트 애플리케이션은 자원을 효율적으로 이용하지 못했습니다.

구성을 살펴보면 다음과 같습니다.

``` yaml
# pod resource 설정
resources:
  limits:
    cpu: 1000m
    memory: 1048Mi
  requests:
    cpu: 1000m
    memory: 1048Mi
```

[코드] - pod resource 설정

회원 서비스는 멀티스레드(multi-thread)로 동작하는 Spring Boot Embedded Tomcat으로 운영하고 있으며 서버 리소스는 1 core, 1GB 메모리가 할당되어 있습니다. 작고 가벼운 애플리케이션을 지향하기 때문에 대부분의 서비스를 위 환경으로 운영합니다. 그러나 애플리케이션이 멀티스레드 방식으로 동작하면 반드시 적절한 스레드 수가 할당 됐는지 살펴봐야 합니다.

한정된 자원을 두고 경합을 벌이다 효율적으로 동작하는 임계치를 넘어서면 오히려 효율이 떨어지는 문제가 발생합니다. Embedded Tomcat의 경우 설정을 하지 않으면 활성화되는 스레드는 최대 200개입니다. 서비스 환경에 따라 다를 수 있지만 이는 적절하지 않다고 생각하여 설정을 변경하며 성능 테스트를 진행했고 아래와 같은 지표를 얻었습니다.

| 최대 톰켓 스레드 | RPS per POD | 응답편차   |
|-----------|-------------|--------|
| 200       | 500         | 불안정    |
| 100       | 800         | 불안정    |
| 50        | 1100        | 다소 불안정 |
| 30        | 1300        | 안정적    |
| 20        | 1300        | 불안정    |

스레드 수가 200개일 때 가장 낮은 처리량을 보이고 가장 빠른 응답과 가장 느린 응답 편차도 큽니다. vmstat 등을 활용해 수집한 매트릭을 분석한 결과 트래픽이 몰릴수록 많은 Context Switching이 발생했고 이것이 성능에 영향을 미친다고 판단해 tomcat thread 수를 줄여가며 테스트했습니다.

그 결과 30-50개일 때 가장 안정적으로 동작했습니다. 어떤 API를 테스트하느냐에 따라 다른 결과가 나왔지만 전반적으로 30개일 때가 가장 안정적으로 동작한다고 판단해 아래 설정을 추가했습니다.

```yaml
# tomcat thread 설정
server:
  tomcat:
    threads:
      max: 30
```
[코드] - tomcat thread 를 30개로 설정

### WarmUp

애플리케이션의 성능 테스트를 진행하다 초기 구동 시 응답 속도가 느린 문제를 확인했습니다. 이를 해결하기 위해 애플리케이션이 외부 트래픽을 받기 전에 미리 주요 API를 호출하는 코드를 추가했습니다.

JVM 애플리케이션은 클래스 로딩을 할 때 레이지(Lazy) 로딩 방식을 사용합니다. 따라서 해당 클래스가 처음 참조될 때 메모리에 로드됩니다. 이 특성으로 인해 초기 구동 속도가 느릴 수 있는데 이를 해결하기 위해 주요 API를 미리 호출하여 필요한 클래스를 미리 메모리에 로드 했습니다. 클래스를 메모리에 미리 로드해 두어 실제 트래픽에 더 빠르게 응답할 수 있습니다.

상황에 따라 웜업은 JIT(Just-In-Time) 컴파일러의 최적화를 유도할 수 있습니다. JIT 컴파일러는 코드의 실행 중에 핫스팟을 찾아 최적화합니다. 웜업을 통해 이러한 핫스팟을 미리 찾고 JIT 컴파일러가 최적화하도록 유도할 수 있습니다.

저희는 주요 API 를 호출하는 것으로도 목적을 달성할 수 있어 아래 코드를 추가했습니다.

```kotlin
@Component
class WarmupApplicationListener : ApplicationListener<ApplicationReadyEvent> {
   
    override fun onApplicationEvent(event: ApplicationReadyEvent) {
        try {
            warmup()
        } catch (e: Exception) {
            logger.error("warm up에 실패하였습니다. ${e.message}", e)
        }
    }

    private fun warmup() {
        restTemplate.getForObject("$host:$port$apiPath/members/access-token-{accessToken}", String::class.java, it.token)
        restTemplate.getForObject("$host:$port$apiPath/members/email-{email}", String::class.java, warmupMemberEmail)
        restTemplate.getForObject("$host:$port$apiPath/members/id-{id}", String::class.java, warmupMemberId)
        // ... 중략... 
    }
}
```

[코드] - 웜업 코드

## 결과

이 모든 과정을 거친 회원 서비스는 어떻게 개선됐을까요?

![[그림] - 개선 전/후](/img/member-service-performance-tuning-01/done-4.png)*[그림] - 개선 전/후*
![[그림] - 응답편차](/img/member-service-performance-tuning-01/done-5.png)*[그림] - 응답편차*

붉은 선 기준으로 개선 전과 후를 비교해보면 약 8배 정도 개선됐습니다. 응답 편차는 2~10ms로 개선 전 대비 상당한 효과가 있습니다. 이제 저희는 아무리 많은 트래픽이 몰려와도 더이상 두려움에 떨지 않는 팀이 될 수 있겠네요.

## 마치며
성능 튜닝을 진행하는 동안 많은 것을 느꼈습니다. 고수의 영역이라 생각한 성능 튜닝은 알고 보니 평소에 알고 있는 개념을 잘 조합하는 것으로도 충분했으며 사소한 실수가 모이면 성능에 악영향을 끼친다는 것도 체험했습니다. 저희가 준비한 글은 여기까지입니다. 앞으로 더 많은 숙제가 저희를 기다리고 있으며 더 높은 산을 넘어야 하지만 오늘의 경험이 내일을 준비하는 우리에게 큰 자산이 될 것입니다.

여러분의 애플리케이션에 항상 성능과 안정이 함께하기를 바라며 긴 글 마치겠습니다. 읽어주셔서 감사합니다.
