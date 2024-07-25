---
layout: post
title:  에셋팀 레거시 개선 (2) 쏘카존 관리 시스템 - 차량재배치 리펙터링
subtitle: 객체지향과 디자인패턴
date: 2024-07-24 00:00:00 +0900
category: dev
background: '/img/how-to-organize-tech-blog/thumbnail.jpg' 
author: onestone
comments: true
tags:
    - dev
    - asset
    - developer
    - service engineering
---



# 목차
1. **소개**
2. **개선 목적**
3. **차량재배치 설명**
4. **카프카 스프링으로 변경된 아키텍처 및 코드 설명**

    4.1. **기존 코드 설명**

    4.2. **책임분리**
5. **테스트코드 작성**
6. **리펙터링과 전략패턴**
7. **마무리**
<br><br><br>


# 1. 소개
 안녕하세요. 쏘카 서비스 엔지니어링본부 에셋(Asset)팀 백엔드 개발자 원스톤입니다. 저는 쏘카 존과 차량 도메인을 개발하고 있습니다. 

 지속 성장하는 소프트웨어를 만들기 위한 코드의 품질 개선 노력들을 글로 풀어내려고 합니다. 먼저 차량재배치의 레거시 개선 작업이 필요했던 이유와 차량재배치 비즈니스의 도메인 지식과 코드 리펙터링을 순차적으로을 소개하겠습니다.

<br>

# 2. 개선 목적
차량재배치는 오래전 개발되어 방치돼 있었습니다. 코드 레벨에서의 문제로 일부 기능 수정 또는 오류 발생 시 수정해야 하는 부분이 명확하지 않았습니다. 하나의 거대 클래스로 이루어진 모든 코드를 파악한 뒤 일부분을 수정하는 방식으로 유지보수하고 있어 사이드이펙트에 대한 두려움이 존재했습니다. 기술 레벨에서의 문제로 기술조직 대부분이 사용 중인 AWS DMS와 카프카를 이용하지 않고 맥스웰과 키네시스를 이용하고 있었습니다. 맥스웰을 사용할 줄 아는 개발자는 극히 드물었고 사용하는 곳도 몇 군데 없었습니다. 문제가 발생했을 때 참고 문헌이 없어 해결하기가 쉽지 않았습니다. 맥스웰 기능에 문제가 생기면 수동으로 작업을 처리해야 했습니다. 해당 문제는 자주 발생했습니다. 결국, PM, 개발자, 사업부 모두의 동의를 얻어 일감으로 처리하게 됐습니다.

<br>

# 3. 차량재배치 설명
쏘카는 공간인 존을 갖고 있습니다. 존에는 차량이 존재하고 그 차량을 서비스합니다. 매출 지표를 참고하거나 침수 등의 사유가 발생하면 한 존에서 다른 존으로 차량 이동을 수행하고 있습니다. 그것을 차량재배치로 정의합니다. 내부 시스템인 존관리시스템에서 차량재배치를 요청하고 실제 운전자가 차량을 이동시키면 이벤트를 받아 차량재배치를 수행합니다.

<br>

# 4. 스프링 카프카로 변경된 아키텍처 및 코드 설명
### 4.1. 기존 코드 설명
기존 코드는 다음과 같이 하나의 클래스에 모든 로직이 있었습니다. 키네시스와 스프링 로직이 비즈니스 로직에 침투해 있어 분석하기 힘들었고, 키네시스를 다른 기술로 변경하면 모든 코드를 수정해야 했습니다. 게다가 테스트코드도 부재한 상황이었습니다.

```kotlin
...
@Component
class CarRelocationProcess(
   ...
) : ApplicationRunner, DisposableBean {

   // AWS SDK - 키네시스 초기화 변수
   private val STREAM_NAME = ...
   private val STREAM_ARN = ...
   private var kinesis: AmazonKinesis? = null
   private var client: KinesisAsyncClient? = null
   ...

   // 스프링 - ApplicationRunner 로직
   override fun run(args: ApplicationArguments?) {
      this.init() // AWS SDK - 키네시스 초기화 로직
      val shards = this.getShards() // AWS SDK - 키네시스 샤드를 가져오는 로직

      while (true) {
         val responseHandlers = shards.map {
            ...
            val consumerARN = // ...

            val request = SubscribeToShardRequest.builder()
                .consumerARN(consumerARN)
                .shardId(it.shardId)
                .startingPosition(StartingPosition.builder()
                    .type(ShardIteratorType.AT_TIMESTAMP)
                    .timestamp(Instant.now().minusSeconds(START_BUFFER_TIME))
                    .build()
                ).build()

           responseHandlerBuilder(...)
         }

         responseHandlers.forEach {
            it.join()
         }
      }
   }

   // 스프링 - DisposableBean 로직
   override fun destroy() {
      ...
   }
   
   private fun init() {
      ...
   }
   
   private fun getShards() {
      ...
   }
 
   private fun responseHandlerBuilder(
      ...
   ): CompletableFuture<Void> {
      val responseHandler = SubscribeToShardResponseHandler
            .builder()
            .onError { t -> logger.error("Error during stream - ${t.message}") }
            .onComplete { logger.info("All records stream successfully") }
            .subscriber { e -> process(shardId, e) }
            .build()

      return client.subscribeToShard(request, responseHandler)
   }

   // 비즈니스 로직
   private fun process(...) {
      runBlocking {
         val drivingList = drivingReservations?.map {
            // 운행중 상태의 예약에 해당하는 차량 처리 로직
         }

         val canceledList = canceledReservations?.map {
            // 취소 상태의 예약에 해당하는 차량 처리 로직
         }

         drivingList?.forEach { it.await() }
         canceledList?.forEach { it.await() }
      }
   }
}
```

<br>

## 4.2. 책임분리
### 4.2.1. 다이어그램
키네시스를 카프카로 변경하기 위해 비즈니스 로직만 재사용하고 나머지를 모두 분리하기로 했습니다.
![로직 분리 및 처리 흐름](/img/2024-07-24-legacy-car-relocation/legacy-car-relocation_img_1.png) *간단하게 도식화한 다이어그램*
    
개발 전 간단한 다이어그램을 통해 구조를 잡았습니다. 키네시스 로직은 카프카 로직으로 변경하고 클래스를 분리했습니다. 스프링 로직은 상속을 사용하지 않고 어노테이션과 DI로 처리했습니다. 비즈니스 로직은 세 부분으로 나눴습니다. `CarRelocationKafKaController`는 카프카 로직이 전달하는 파라미터에 대한 검증을 진행하고, `CarRelocationService`는 차량재배치 비즈니스 로직을 처리합니다. 하지만 실제 동작은 전달받은 파라미터의 상태에 따라 `CarRelocationDrivingService` 또는 `CarRelocationCancelService`가 담당합니다.


### 4.2.2. 패키지
패키지는 다음과 같이 구성했습니다.
    
```
carRelocation
    +--- app
    |     +---message
    |            \---converter
    |     +---service
    |     \---ui
    \---infra
          \---kafka
                +---message
                \---converter
```
    
크게 차량재배치 비즈니스 로직을 처리하는 `app`과 기술적 로직을 처리하는 `infra/kafka`로 나눴습니다. `kafka/converter`는 카프카 컨슈머가 전달받은 데이터를 자바 객체로 변환합니다. `kafka/converter`로 처리한 후 객체를 `app/ui`로 전달합니다. 자바 객체의 유효성을 검증하고 `app/converter`를 통해 비즈니스 로직에 필요한 객체를 만들어 다음 계층으로 전달합니다. 전달받은 `app/service`는 비즈니스 로직에 대한 유효성 검사와 동작을 수행합니다.


### 4.2.3. 구현체
`KafkaListener`는 카프카 토픽의 내용을 받고 컨버터를 사용해 코틀린 객체로 변환하여 다음 계층에 전달하는 책임을 갖습니다.

```kotlin
...
@Component
class KafkaListener(
    private val messageConverter: MessageConverter,
    private val carRelocationController: KafkaReservationController,
) {
    ...
    @KafkaListener(id = "reservation-kafka-listener", topics = ["\${kafka.topic.reservation}"])
    fun consumeReservationEvent(payload: ConsumerRecord<String, String>) {
            ...
        val convertedPayload: KafkaPayload = messageConverter.convert(payload)
        val result: Boolean = carRelocationController.processKafkaPayload(convertedPayload)
        ...
    }
}
```

`KafkaReservationController`는 `KafkaListener`가 전달한 파라미터의 유효성 검사 실행과 컨버터를 사용해 비즈니스 로직에서 사용할 DTO로 변환하는 책임을 갖습니다.

```kotlin
...
@Component
class KafkaReservationController(
    private val carRelocationService: CarRelocationService,
    private val reservationMessageConverter: ReservationMessageConverter,
) {
    /**
    * @see ...KafkaReservationControllerTest#테스트코드메소드명()
    * ...
    **/
    fun processKafkaPayload(payload: KafkaPayload): Boolean {
        val result: RelocationResultState = if (payload.validatePayload() && payload.validateMessageContents()) {
            val reservation: ReservationDto = reservationMessageConverter.convert(payload)
            carRelocationService.processCarRelocation(reservation)
        } else {
            RelocationResultState.SKIP
        }

        return ...
    }
}
```
    
`CarRelocationService`는 파라미터인 `ReservationDto`의 비즈니스 로직 처리 유무를 결정하고 Dto의 상태에 따라 어떤 로직을 처리할지 결정합니다.  
    
```kotlin
...
@Service
class CarRelocationService(
    private val carRelocationDao: ZoneCarRelocationDao,
    private val carRelocationCancelService: CarRelocationActionService,
    private val carRelocationDrivingService: CarRelocationActionService,
        ...
) {
    /**
    * @see ...CarRelocationServiceTest#테스트코드메소드명()
    * ...
    **/
    fun processCarRelocation(reservation: ReservationDto): RelocationResultState {
        fun validate(reservationId: Long): Boolean { ... }
        fun findCarRelocation(reservationId: Long): ICarRelocation { ... }

        val reservationId = reservation.id

        if (!validate(reservationId)) {
            return RelocationResultState.SKIP
        }

        return findCarRelocation(childOrParentReservationId)?.let {
            this.processCarRelocationHistory(it, reservation.state)
        } ?: RelocationResultState.SKIP
    }

    private fun processCarRelocationHistory(
        carRelocation: ICarRelocation,
        reservationState: ReservationState,
    ): RelocationResultState {
        return when (reservationState) {
            LegacyCodes.ReservationState.CANCEL -> {
                carRelocationCancelService.act(carRelocation)
            }
            LegacyCodes.ReservationState.DRIVING -> {
                carRelocationDrivingService.act(carRelocation)
            }
            else -> {
                RelocationResultState.SKIP
            }
        }
    }
}
```

<br>    

# 5. 테스트코드 작성
테스트코드에 대한 원칙은 각 클래스 책임에 맞고 의미 있는 테스트코드를 작성합니다. 기술검증에 필요한 학습(learning) 테스트는 별도 작성합니다.

`KafkaReservationControllerTest`는 전달한 파라미터의 유효성 검사 실행 책임을 테스트합니다.

```java
...
open class KafkaReservationControllerTest : KafkaReservationControllerTestSetup() {
    @Test
    fun `processKafkaPayload 성공 테스트`() {
        // given
        ...

        // when
        ...

        // then
        ...
    }

    @Test
    fun `processKafkaPayload 스킵 테스트 - 카프카 데이터 오류`() {
		    ...
    }

    @Test
    fun `processKafkaPayload 스킵 테스트 - 차량재배치 서비스 로직 스킵`() {
            ...
    }
}
```

`CarRelocationServiceTest`는 `CarRelocationService`는 파라미터인 `ReservationDto`의 비즈니스 로직 처리 유무를 결정하고 Dto의 상태에 따라 어떤 로직을 처리할지 결정하는 책임을 테스트합니다. 

```java
...
open class CarRelocationServiceTest : CarRelocationServiceTestSetup() {

    // 이 글에서 validate 실패의 자세한 사유는 생략합니다.
    @Test
    fun `processCarRelocation 스킵 테스트 - validate 실패`() {
        // given
        ...
				
        // when
        ...

        // then
        ...
    }

    ...

    // 이 글에서 validate 성공의 자세한 사유는 생략합니다.
    @Test
    fun `processCarRelocation 성공 테스트 - validate 성공`() {
	    ...
    }

    ...
		
    @Test
    fun `processCarRelocation 실패 테스트 - 취소 상태의 예약 처리 로직에서 스킵`() {
        ...
    }

    @Test
    fun `processCarRelocation 실패 테스트 - 운행중 상태의 예약 처리 로직에서 스킵`() {
        ...
    }

    @Test
    fun `processCarRelocationHistory 성공 테스트 - 취소 상태의 예약 처리 로직에서 성공`() {
        ...
    }

    @Test
    fun `processCarRelocationHistory 성공 테스트 - 운행중 상태의 예약 처리 로직에서 성공`() {
        ...
    }
}
```

<br>

# 6. 리펙터링과 전략패턴
기술 로직과 비즈니스 로직을 분리하여 책임이 나뉘었고 테스트 코드도 생겼기 때문에 사이드이펙트 걱정이 줄었습니다. 하지만 스프링 DI를 사용하면서 `CarRelocationService`의 실제 행동을 처리 하는 `carRelocationCancelService`과 `carRelocationDrivingService`를 직접 참조하게 됐습니다. 그렇기 때문에 처리해야 할 상태가 추가되거나 변경되면 `CarRelocationService` 또한 변경되어야 합니다.

![차량재배치 비즈니스 도메인의 UML](/img/2024-07-24-legacy-car-relocation/legacy-car-relocation_img_2.png) *차량재배치 서비스 UML*
`CarRelocationService`에서 `CarRelocationDrivingService`아 `CarRelocationCancelService`를 직접 참조하지 않게하기 위해 클래스 하나를 추가합니다. 그리고 `CarRelocationService`가 그 새로운 클래스를 사용하도록 리펙터링 합니다.

`CarRelocationActionServiceSelector`는 예약 상태에 따라 수행할 행동을 하는 클래스를 선택하여 반환하는 책임이 있습니다.

```kotlin
...
@Component
class CarRelocationActionServiceSelector(
    private val carRelocationActionServices: Map<String, CarRelocationActionService>
) {

    /**
     * @see ....CarRelocationActionServiceSelectorTest#테스트메소드명()
     */
    fun select(state: ReservationState): CarRelocationActionService? {
        return when (state) {
            ReservationState.CANCEL -> carRelocationActionServices[CANCEL_PROCESS_SERVICE_BEAN_NAME]
            ReservationState.DRIVING -> carRelocationActionServices[DRIVING_PROCESS_SERVICE_BEAN_NAME]
            else -> null
        }
    }

    companion object {
        const val CAR_RELOCATION_CANCEL_SERVICE_BEAN_NAME = "carRelocationCancelService"
        const val CAR_RELOCATION_DRIVING_SERVICE_BEAN_NAME = "carRelocationDrivingService"
    }
}
```

추가된 `CarRelocationActionServiceSelector` 클래스를 사용하도록 `CarRelocationService` 클래스를 리팩터링합니다.

```kotlin
...
@Service
class CarRelocationService(
    private val carRelocationDao: ZoneCarRelocationDao,
    private val carRelocationActionServiceSelector: CarRelocationActionServiceSelector,
	  ...
) {
    /**
    * @see ...CarRelocationServiceTest#테스트코드메소드명()
    * ...
    **/
    fun processCarRelocation(reservation: ReservationDto): RelocationResultState {
        fun validate(reservationId: Long): Boolean { ... }
        fun findCarRelocation(reservationId: Long): ICarRelocation { ... }

        val reservationId = reservation.id

        if (!validate(reservationId)) {
            return RelocationResultState.SKIP
        }

        val carRelocationActionService = carRelocationActionServiceSelector.select(reservation.state)

        if (carRelocationActionService == null) {
            logger.debug("처리할 수 있는 상태의 서비스가 존재하지 않습니다.")
            return RelocationResultState.SKIP
        }

        return carRelocationActionService.act(carRelocationHistory)
    }
}
```

`CarRelocationCancelService`와 `CarRelocationDrivingService`에 빈명을 직접 주입합니다.

```kotlin
...
@Service(CAR_RELOCATION_CANCEL_SERVICE_BEAN_NAME)
class CarRelocationCancelService(...) { // 취소 상태의 예약에 해당하는 차량 처리 로직 }
```
```kotlin
...
@Service(CAR_RELOCATION_DRIVING_SERVICE_BEAN_NAME)
class CarRelocationDrivingService(...) { // 운행중 상태의 예약에 해당하는 차량 처리 로직 }
```


![리팩터링된 차량재배치 비즈니스 도메인의 UML](/img/2024-07-24-legacy-car-relocation/legacy-car-relocation_img_3.png) *리펙터링된 차량재배치 서비스 UML*

추가된 `CarRelocationActionServiceSelector` 클래스에 대한 테스트코드를 추가해주면 작업은 마무리됩니다.

```java
...
open class CarRelocationActionServiceSelectorTest {

    @Test
    fun `취소 상태 행동 반환 테스트`() {
        ...
    }
		
    @Test
    fun `운행중 상태 행동 반환 테스트`() {
	    ...
    }
}
```

<br>

# 7. 마무리
거대 클래스를 기능 및 비즈니스 로직별로 나누어 클래스를 작성하였습니다. 덕분에 테스트코드를 작성하기 쉬워졌고 기술이 바뀌더라도 비즈니스 로직을 가진 클래스는 변경될 필요가 없어졌습니다. 게다가 책임을 나눠 어느 부분에서 오류가 발생하더라도 어떤 책임을 가진 클래스에서 문제가 발생했는지 금방 찾을 수 있게 됐습니다. 뒤에서는 클래스 확장성을 고려하며 리팩터링하다보니 전략패턴과 유사한 UML다이어그램이 만들어졌습니다.

객체지향 원칙을 적용해 코드의 품질을 적절히 관리해야만 지속 성장하는 소프트웨어를 만들 수 있습니다. 문제의 해법을 도출하기 위해 다양한 디자인패턴을 적용하고 코드의 품질을 높이기 위한 고민을 함께 하고 싶은 분들은 쏘카 애셋(Asset)팀에 지원해 주세요!