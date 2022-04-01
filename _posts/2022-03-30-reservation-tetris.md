---
layout: post
title: "모빌리티 업계의 최적화 프로젝트 - 예약 테트리스"
subtitle: "Constraint Programming을 활용한 예약 차량 배정 최적화"
date: 2022-03-30 09:00:00 +0900
category: data
background: '/assets/images/hello-i-m-nik-lUbIun4IL38-unsplash.jpg'
author: carrot, kyle
comments: true
tags:
  - data
  - optimization

---

<div class="photo-copyright">
Photo by <a href="https://unsplash.com/@helloimnik">Hello I'm Nik</a> on <a href="https://unsplash.com/photos/lUbIun4IL38">Unsplash</a>
</div>

<br>

안녕하세요, 쏘카 데이터비즈니스본부의 캐롯, 카일입니다.

쏘카에서 차량을 대여하기 위해 앱을 탐색하다보면 하나의 쏘카존 안에서도 아반떼, 레이, 코나 등 다양한 차량들이 눈에 들어옵니다. 
이 때, 한 존에 같은 종류의 차량이 한 대만 있지 않은 경우라면 어떤 차량이 어떤 과정을 거쳐 나에게 배정되는지 궁금하신 적 없으신가요?

이번 글에서는 바로 그 과정의 뒤에 있는 **예약 테트리스 프로젝트**를 다룹니다.

프로젝트를 시작하게 된 계기부터, 선형 최적화 모델과 실제 서비스에 적용하기 위해 구축한 인프라 구조까지 프로젝트의 전반적인 내용을 자세하게 다루었으니 이 글로 평소에 가지고 있던 궁금증이 풀리길 바랍니다.

<br>


## 목차

- 예약 테트리스 프로젝트 소개
- 예약 테트리스 최적화 모델링
- 전체 인프라 구조
- 예약 테트리스 적용 성과
- 마무리

<br>

---

## 예약 테트리스 프로젝트 소개

### 예약 "테트리스"?

프로젝트를 설명하기에 앞서, 갑자기 프로젝트 이름으로 테트리스라니 의아하다고 느낄 수도 있을 것 같습니다. 
먼저 간단히 테트리스 게임을 설명하자면, 테트리스는 위에서 내려오는 블록을 빈 공간에 맞춰 차곡차곡 쌓아 꽉 찬 한 줄 한 줄을 만들어 포인트를 얻는 퍼즐게임입니다. 
이번 프로젝트의 이름에 테트리스가 붙여진 이유는 **들어오는 예약을 차량의 빈 공간에 차곡차곡 정리하는 과정**이 마치 테트리스 게임과 비슷하다고 생각했기 때문입니다.

차량의 비어있는 시간에 예약을 얼마나 잘 채워넣는지는 쏘카가 차량을 효율적으로 운영하는 데 굉장히 중요한 문제입니다. 
**같은 대수의 차량으로도 예약마다 어떤 차량을 배정해주는지에 따라 얼마나 많은 고객이 쏘카를 이용할 수 있는지가 달라지기 때문**이죠.

한 쏘카존에 같은 종류의 차량이 두 대 있을 때를 예시를 들어서 부연설명을 해보겠습니다.
A와 B라는 서로 다른 두 고객이 각각 08시~12시와 20시~24시 총 두 건의 예약을 한 상태에서, 0시부터 24시까지 24시간 동안 이용하고 싶어하는 C라는 고객이 같은 존에서 같은 차량을 대여하고 싶어 하는 상황을 생각해봅시다.
이때 만약 A와 B에게 서로 다른 차량이 배정된 상태라면 C는 이용할 수 있는 차량이 없을 것입니다.
두 차량 모두 C의 예약을 받을 수 있을 만큼의 충분한 여유공간이 없기 때문이죠.

![img](/img/reservation-tetris/example-reserve-fail.png){: width="100%"}

하지만 만약 A와 B의 예약에 같은 차량을 배정했다면 두 사람의 예약에 배정되지 않은 하나의 차량이 남아 C는 수월하게 쏘카를 이용할 수 있게 됩니다. 
이처럼 예약마다 상황에 더 적절한 차량을 배정해주면 추가로 더 많은 차량을 준비하지 않아도 더 많은 고객들이 쏘카의 서비스를 이용할 수 있도록 할 수 있습니다.

![img](/img/reservation-tetris/example-reserve-success.png){: width="100%"}


### 예약 테트리스 프로젝트의 목적

예약 테트리스 프로젝트가 도입되기 이전에도 이러한 효율화 작업은 이루어지고 있었습니다. 
다만 이 모든 작업이 사람의 손으로 이루어졌다는 점이죠. 
앞서 설명한 문제 수준의 난이도라면 사람 손으로도 쉽게 처리할 수 있겠지만, 쏘카는 하루 최대 3만 건 이상의 신규 예약이 생성되고, 매일 약 2만 대의 차량이 운영되고 있는 국내 최대규모 카셰어링 서비스입니다. 
같은 종류의 차량이 두 대뿐이라고 해도 최대 90일 전부터 미리 예약을 할 수 있다는 점을 감안하면 사람의 손으로 최적의 차량 배정을 하기가 쉽지 않다는 것을 이해하기란 어렵지 않습니다.
실제로 쏘카에서 가장 큰 존인 제주 스테이션의 예약을 정리하는 것은 하루에 고정적으로 최대 4시간이 소요될 만큼 지역사업팀의 업무에 큰 부분을 차지하고 있었습니다.

**예약 테트리스 프로젝트는 이런 차량 배정 최적화 과정을 개선하기 위해 시작되었습니다.**
차량 배정 프로세스를 자동화하면 존 매니저들이 업무시간을 더 효율적으로 사용할 수 있을 뿐더러, 사람의 머리로는 도출하기에 한계가 있는 최대로 최적화된 차량 배정으로 서비스를 운영할 수 있게 됩니다. 업무 효율과 공급 효율 두 마리 토끼를 모두 잡기 위한 프로젝트인 셈이죠.

![img](/img/reservation-tetris/asis-tobe.png){: width="100%"}


## 예약 테트리스 최적화 모델링

### 제약조건 계획법

예약 테트리스 프로젝트에서는 차량 배정 최적화 모델링을 위해 제약조건 계획법을 사용합니다.
최적화 모델링을 다루기 전에 제약조건 계획법(Constraint Programming)이 무엇인지 짚고 넘어가겠습니다.
[제약조건 계획법(Constraint Programming)](https://en.wikipedia.org/wiki/Constraint_programming)이란 제약조건으로 표현된 해 공간에서 조합 최적화(Combinatorial Optimization) 문제를 푸는 최적화 기법입니다.
생소한 단어들이 많아 처음 접하는 개념처럼 느낄 수도 있지만 일차 부등식을 활용한 문제를 풀어본 경험이 있다면 높은 확률로 낯설지 않다고 느낄 것입니다.

0보다 큰 정수 x와 y가 아래와 같은 조건을 만족할 때, k값을 최대화하는 x, y를 찾는다고 해볼까요?
이런 유형의 문제가 바로 제약조건 계획법을 사용할 수 있는 간단한 예라고 할 수 있습니다.

![img](/img/reservation-tetris/cp-example.png){: width="100%"}

우리에게 익숙한 이 문제를 최적화 모델의 3요소라 할 수 있는 **제약조건, 결정변수, 목적함수**로 분해해서 설명해볼게요.

- 제약조건 (Contraints)
	- x, y의 양의 정수조건과 부등식으로 표현된 두 직선이 x와 y가 가질 수 있는 값을 한정하는 **제약조건**이 됩니다.
- 결정변수 (Decision Variable)
	- 제약조건으로 인해 가질 수 있는 값이 제한되고 우리가 알고자하는 변수인 x와 y는 k값을 결정하는 **결정변수**로 정의할 수 있습니다.
	- 모든 제약조건을 만족하는 결정변수 값의 집합을 **해 공간(solution space)**라고 합니다.
- 목적함수 (Objective Function)
	- 최대화하고자하는 값 k는 이 문제에서 궁극적으로 얻고자하는 목적이 되는 함수로 **목적함수**라고 부릅니다.

위 문제에서 각 요소들을 표시하자면 다음처럼 되겠죠.

![img](/img/reservation-tetris/cp-example-explained.png){: width="100%"}

제약조건 계획법은 이처럼 정의된 조합 최적화 문제에서 제약조건을 만족하는 해 공간을 구하고, 목적함수가 있는 경우 그 값을 최대화/최소화하는 결정변수의 조합을 탐색하는 최적화 기법이라고 할 수 있습니다.


### 어떤 상태가 더 '최적'일까?

예약 테트리스 모델링을 시작하면서 가장 고민을 많이 했던 부분은 어떤 차량 배정 상태가 더 "최적"인지를 수치적으로 판단할 수 있는 지표를 정의하는 것이었습니다.
모델 성능과 최적화 정도 사이에서 고민한 끝에, 완벽한 최적해는 아니더라도 가상의 예약건이 가장 많이 들어올 수 있도록 예약에 차량을 배정하는 모델을 짜게 되었습니다.
가상예약건의 예약 가능성이 클수록 더 최적이라고 판단한 것입니다.

앞서 들었던 예시로 설명하자면, A와 B의 예약을 서로 다른 차량에 배정한다면 예약 가능한 하루 단위의 가상예약건수는 0이 되지만, 두 예약을 같은 차량에 배정한다면 1이 되기 때문에 후자의 경우를 더 최적이라고 판단할 수 있는 것이죠.
예약이 어떤 차량에 배정되었는지만이 중요하고, 다른 예약과의 선후 관계는 변수 단위에서는 고려하지 않아도 된다는 점이 모델의 최적해 도출 속도 측면에서 큰 이점으로 작용하였습니다.

![img](/img/reservation-tetris/example-possibility-0.png){: width="100%"}

![img](/img/reservation-tetris/example-possibility-1.png){: width="100%"}


### 예약 테트리스 최적화 모델

예약 테트리스의 최적화 모델은 다음과 같이 정의됩니다.

- 결정변수: 실제 또는 가상예약에 어떤 차량의 배정 여부 (0 또는 1)
- 목적함수: 예약가능한 가상예약건수의 최대화
- 제약조건
	- 모든 예약은 단 하나의 차량에 배정되어야 한다
	- 차량이 고정되어야하는 예약은 배정된 차량이 변경되면 안된다
	- 임의의 서로 다른 두 개의 실제예약은 같은 차량에 배정된 경우 서로 겹칠 수 없다
	- 임의의 가상예약과 실제예약은 같은 차량에 배정된 경우 서로 겹칠 수 없다

이를 수식으로 표현하면 다음과 같이 쓸 수 있습니다.

![img](/img/reservation-tetris/formulation.png){: width="100%"}

### Google OR-Tools

수식화한 모델을 코드로 구현하고 최적해를 찾기 위해서는 솔버(solver)가 필요합니다.
예약 테트리스의 최적화 모델은 구글의 오픈소스 솔버인 [Google OR-Tools](https://developers.google.com/optimization)의 파이썬 패키지로 구현하였습니다.
특히 Google OR-Tools가 지원하는 솔버 중 조합 최적화 문제를 위한 [CP-SAT](https://developers.google.com/optimization/cp/cp_solver) 솔버는 무료로 사용할 수 있는 오픈소스임에도 [뛰어난 성능](https://www.minizinc.org/challenge2021/results2021.html)을 가지고 있고, 사용자의 입맛에 맞는 커스텀 설정들을 지원하기 때문에 이번 프로젝트에 적합하다고 판단했습니다.

제약조건을 수식화하는 것도 직관적이기 때문에 스크립트화 난이도도 높지 않은 편이라 솔버를 처음 접하는 분들에게도 권할만하다고 생각합니다.
앞에서 예시로 가져온 문제를 CP-SAT 솔버로 푸는 파이썬 코드를 보면 실제 수식과의 괴리감이 크게 없는 것을 확인할 수 있습니다.

![img](/img/reservation-tetris/python-code-example.png){: width="60%"}


## 전체 인프라 구조

모델링 작업을 완료했으니 이제 모델을 실제 서비스애 적용하기 위한 인프라 작업이 필요합니다.
예약 테트리스 모델을 위해서는 다음과 같은 요구사항이 있었습니다.

1. 솔버를 실행할 수 있는 환경이 준비되어야 하고  
2. 서버와 데이터를 주고받을 수 있는 데이터 파이프라인이 필요하며
3. 모니터링을 위해 실행된 결과를 데이터베이스에 저장해야 합니다.

### Pub/Sub 구조

[Pub/Sub 구조(Publish/Subscribe)](https://cloud.google.com/pubsub/docs/overview)는 마이크로서비스 아키텍쳐(MSA)에서 서비스 간의 비동기 통신을 위해 사용되는 통신 모델입니다.
Pub/Sub 구조에서 사용되는 몇 가지 중요한 개념을 짚고 넘어가면 다음과 같습니다.

* Topic
  * publisher가 발행하는 메시지를 subscriber에게 전송하는 창구 같은 개념입니다.
* Publisher/Producer
  * 메시지를 생성해 토픽으로 발행하는 서비스입니다. 
* Subscriber/Consumer
  * 토픽을 구독해 메시지를 받는 주체입니다.
* Push와 Pull
  * subscriber가 메시지를 전달받는 방식의 종류에는 Push와 Pull 두 가지가 있습니다.
  * Push: Pub/Sub 시스템이 subscriber에서 메시지를 **밀어넣는(push)** 방법
  * Pull: Subscriber가 직접 서비스로부터 메시지를 **당겨오는(pull)** 방법

(+) 예약 테트리스에서의 Pub/Sub 예제 + 설명


### 최종 인프라 구조

Google Cloud Pub/Sub과 Dataflow를 사용해 완성된 인프라 구조는 다음과 같습니다.

![img](/img/reservation-tetris/infra.png){: width="100%"}

<br>

## 예약 테트리스 프로젝트 성과

프로젝트 소개 단에서도 언급했다시피, 예약마다 어떤 차량을 배정해주는지에 따라 같은 대수의 차량에서도 얼마나 많은 고객이 쏘카를 이용할 수 있는지가 달라집니다.
그렇다면 예약 테트리스 프로젝트로 공급 효율이 얼마나 더 좋아졌는지는 말 그대로 **사용한 차량 대비 차량이 얼마나 점유되었는지**를 확인해보면 되겠죠.

* **사용한 차량** = 전체 운영 중인 차량 중 예약에 사용된 차량의 비율 = **차량사용비율**
* **차량의 점유 정도** = 점유 가능한 모든 차량의 시간 중 예약으로 점유된 시간의 비율 = **가동률**

일별로 차량사용비율 대비 가동률을 적용 전후로 확인해보면 다음과 같습니다.

![img](/img/reservation-tetris/result-graph.png){: width="90%"}

그래프에서 볼 수 있듯이 적용 후의 추세선이 전과 비교해 더 높은 것을 확인할 수 있습니다. 
이는 동일한 차량사용비율 관점에서 보면 같은 대수의 차량을 공급하여 더 많은 예약을 수용할 수 있다는 것을, 동일한 가동률 관점에서는 더 적은 비용으로 동일한 매출을 만들어낼 수 있음을 의미합니다.


단순히 매출과 가동 관점에서만 성과가 있던 것은 아니었습니다. 
매일 직접 예약을 정리하던 업무가 모두 자동화되었기 때문에 각 지역사업팀에서는 업무시간을 훨씬 더 효율적으로 쓸 수 있게 되었습니다. 
특히 팬데믹 이후 여행 수요가 몰려 성수기 수준으로 예약이 밀려들어오는 상황인 제주사업팀에서 예약 테트리스가 큰 도움이 되고 있다는 소식을 들었을 때 굉장히 뿌듯해한 기억이 있어요. 

![img](/img/reservation-tetris/reaction.png){: width="100%"}


## 마무리

실시간으로 서비스에 적용되는 최적화 프로젝트는 처음 진행해 본 거라 시행착오도 있었지만 함께 작업하며 서포트 해주신 많은 분들 덕분에 성공적으로 프로젝트를 마무리 할 수 있었다고 생각합니다.
예약 테트리스 프로젝트에서 서버 개발 쪽을 맡아주신 맷과 브루스, 프로젝트가 진행되는 동안 여러 부서를 오가며 디테일하게 챙겨주신 주디, 그리고 적극적으로 피드백 주신 제주사업팀과 기타 지역사업팀에게 감사드립니다.


<br>



---