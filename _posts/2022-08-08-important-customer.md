---
layout: post
title: "쏘카 주니어 PM의 장기적으로 ‘가장' 중요한 고객 찾기"
subtitle: "부름 신규 UX A/B TEST"
date: 2022-08-08 09:00:00 +0900
category: data
background: '/img/develop-model-classifying-washed-car/0.png' 
author: bucky
comments: true
tags:
  - pm
  - data
---

안녕하세요. 쏘카에서 고객용 제품을 담당하는 PM1팀의 PM(Product Manager) 버키입니다. 저는 쏘카 입사 후 이제 막 수습 기간을 종료한 주니어 PM인데요. 이 글은 서비스 기획자였던 제가 쏘카의 PM으로서 실무에 투입하기 전 수습 과제를 진행하며 알게된 방법론과 풀어내는 과정에서 배운 사고과정, 쏘카 프로덕트 본부는 어떻게 함께 자라는가?를 공유하기 위한 글입니다. 

다음 내용이 궁금하시다면 끝까지 함께해 주세요. 😃

- 서비스 기획자 경험을 기반으로 쏘카 PM으로 함께하고자 하시는 분
- 쏘카 PM은 어떤 관점으로 일하는지 궁금하신 분
- 장기적으로 중요한 고객을 찾고자 하시는 분
- 고객 ↔ 비즈니스 가치의 조율점을 고민하시는 분

## 목차

글의 목차는 다음과 같습니다.

- 왜 이 글을 쓰게 되었나요?
- 장기적인 관점에서 중요한 고객
    - 장기적인 관점에서 중요한 고객을 왜 찾아야 하는가?
    - 중요한 고객은 누구인가?
- 고객 관점에서 장기적으로 중요한 고객 찾기
- 비즈니스 관점에서 장기적으로 중요한 고객 찾기
    - 행동목표, 5 why로 파고들기
    - 액션 포인트, 수치 목표 연결하기
- 고객 ↔ 비즈니스 관점 싱크 하기
    - 교차점 찾기
    - 실효성 있는 액션 플랜을 위한 방법 (feat. 코호트 분석)
- 느낀 점

## 1. 왜 이 글을 쓰게 되었나요?

![혼란스러운 주니어 PM](/img/important-customer/pm-avator.png)

약 2년 동안 서비스 기획자로 커리어를 쌓아오다 쏘카의 PM이 된 지 어엿 4개월 차, 입사 직후 PM의 R&R에 대해 많은 궁금증과 혼란이 있었습니다. 분명 서비스 기획자와 사고하는 큰 방향성은 다르지 않은 것 같은데 미묘하게 다른 느낌에 “앞으로 실무, 커리어를 위한 PM적 사고로 어떻게 전환하지?”가 가장 큰 고민거리였습니다. 

우선 서비스 기획자와 쏘카의 PM은 어떤 점에서 차이를 보이는지 데스크 리서치와 프로덕트 본부의 온보딩을 통해 다음과 같은 다양한 부분에서 차이점을 발견할 수 있었습니다. 


> 👉🏽 서비스 기획자 (이전 경험) 
> * `커뮤니케이션 형태` 기능 조직 형태로 기획, 디자인, 개발 팀이 분리되어 있음. 프로젝트가 생성됨에 따라 협업하게 됨
> * `조직문화 형태` 수직적인 의사결정 체계, 워터폴 운영 방식
> * `의사결정 형태` 디렉터급에서 비즈니스 의사결정이 이뤄지고 여기에 의견을 덧붙일 수 있음. 이후 화면 기획서, 요구사항 정의서 등과 같은 산출물로 디자인, 개발팀과 커뮤니케이션을 진행함
>
> 👉🏽 쏘카 PM (현재)
> * `커뮤니케이션 형태` 목적 조직 형태로 ‘프로덕트'를 기준으로 팀이 형성됨. 기획, 디자인, 개발 구성원이 자발적이고 적극적인 태도로 최종 산출물(결과)를 만들게 됨 
> * `조직문화 형태` 수평적인 의사결정 체계, 애자일 방법론 운영 방식 (작은 구성 요소를 신속하고 빠른 주기로, 반복적으로 개선/제공하여 고객 만족도를 높이는 것)
> * `의사결정 형태` 프로덕트를 기준으로 생성된 팀에서 전사&프로젝트 목표 맥락 안에서 자유롭게 의견을 발재하고 의사결정을 이룰 수 있음. 이 과정에서 로드맵과 방향성 설정의 리더십을 기반으로 자유도가 있음


쏘카의 PM이라고 자신있게 말하기 위해서 ‘PM의 사고방식’을 함양하는 것이 가장 중요한 목표라고 생각하고 있던 찰나 평가가 아닌, 적응을 돕기 위한 수단으로 수습 과제가 있다는 것을 알게 되었습니다. 

이에 일주에서 이주에 한번 리더와 함께 고민과 근황을 허심탄회하게 나눌 수 있는 1on1 미팅을 진행하며, ‘고객'에 대해 진지하게 고민해 볼 수 있는 주제로 과제를 진행하는 것이 어떤지 제안받고 직접 경험해볼 수 있는 기회가 주어졌습니다.

‘고객'이라는 큰 주제로 쏘카 PM, 더나아가 고객을 위해 프로덕트를 만드는 PM으로서 사고하는 방법과 쏘카에 적응하는 시간을 가질 수 있을 것이라 기대하였는데요.

그럼 이제부터 본격적으로 과제로 선택한 주제인 ‘고객', 그 중에서도 장기적인 관점에서 중요한 고객을 선별하기 위해 고민했던 부분들을 함께 나누고자 합니다. 

## 2. 장기적인 관점에서 중요한 고객

### 2.1 장기적인 관점에서 중요한 고객을 왜 찾아야 하는가?

먼저 장기적으로 중요한 고객(이하 중요한 고객)은 누구일까요? 

우선 ‘장기적'의 의미는 제품 라이프사이클(Product Life Cycle) 측면에서 쇠퇴기 전까지의 기간, 즉 제품의 도입기부터 성장/성숙기라고 정의내려 볼 수 있습니다. 쇠퇴기 전까지로 한정지은 이유는 제품이 쇠퇴기에 진입하면 이후 시장에서 제품의 기회를 기대할 수 없기 때문입니다. 

![그림1) 제품 라이프 사이클](/img/important-customer/product-life-cycle.png)*그림1) 제품 라이프 사이클 / 출처 : [https://haltian.com/resource/new-product-development-cycle-infographic/](https://haltian.com/resource/new-product-development-cycle-infographic/)*

제품의 쇠퇴기를 늦추려면 성장/성숙기 기간을 길게 만들어야 합니다. 제품, 비즈니스가 존재하기 위한 타임라인인 성장/성숙기의 ‘장기성’ 안에서 목표 해야 하는 사용자를 ‘중요한 고객'이라고 말해볼 수 있습니다.

하지만 우리에게 시간과 리소스는 언제나 한정되어 있기 때문에 이 모든 것을 가능케 할 확률이 가장 높은 ‘중요한 고객'을 우선순위로 액션 플랜을 수립해야 합니다.  

![그림2) 북극성 지표를 통해 제품의 올바른 성공을 측정할 수 있습니다.](/img/important-customer/polaris-metric.png)*그림2) 북극성 지표를 통해 제품의 올바른 성공을 측정할 수 있습니다*

그림2 ) 북극성 지표를 통해 제품의 올바른 성공을 측정할 수 있습니다.

중요한 고객을 선별하기 위해선 우선 PM의 전문 영역이라고 할 수 있는 제품 측면에서 궁극적인 목표부터 설정해야 합니다. 

“목표? 그냥 매출로 설정하자! 그게 제일 직관적이고 제품(혹은 비즈니스)의 가치는 언제나 매출로 산정하지 않냐!”고 질문하실 수도 있을 것 같습니다. 하지만 매출은 오직 비즈니스 관점만 반영한 지표이며 우리가 제품에 신규 기능이나 개선 사항을 반영한 후에만 확인할 수 있는 수동적인 지표인 후행 지표라고 답변드려 볼 수 있을 것 같습니다. 

더불어 후행 지표는 판단의 근거로 활용할 수 없기 때문에 좋은 지표라고 보긴 어렵습니다. 그렇다면 어떤 지표가 ‘목표'로 지정할 만하며 액션을 통해 조정할 수 있는 능동적인 성격을 띈 지표일까요?

이에 대해선 제품, 고객, 비즈니스 등 다양한 관점의 조건을 포함하며 제품이 ‘진짜로' 성장하고 있는지 판단할 수 있는 ‘북극성 지표'라는 개념으로 설명드리겠습니다. 북극성 지표는 **제품이 고객에게 전달하고자 하는 가장 핵심적인 가치가 반영된** 단 하나의 핵심 지표이며 관점입니다. 

북극성 지표의 특성을 좀 더 살펴보자면 다음과 같습니다.

- 실제 고객에게 실현된 가치를 반영해야 하며 액션을 통해 정량적인 임팩트로 산출할 수 있어야 합니다.
- 우리가 행하는 각 액션의 최종 목표로 귀결될 수 있어야 합니다.
- ‘매출'과 같이 통제 불가한 후행지표가 아닌 액션을 통해 판단하고 조정할 수 있는 선행 지표여야 합니다.

조금 더 빠른 이해를 위해 다른 기업의 북극성 지표 예시를 함께 살펴보겠습니다. 

아래 그림3과 같이 Airbnb의 경우 ‘n박 예약’, Spotify의 경우 ‘음악 재생 시간’을 북극성 지표로 설정한 것을 볼 수 있습니다. 
아래 예시에서 볼 수 있 듯 각 기업이 설정한 북극성 지표의 공통점은 통제 불가한 후행지표 보다는 액션을 통해 통제 가능한 선행 지표의 특성을 띈다는 것입니다. 

![그림3) 여러 기업의 북극성 지표](/img/important-customer/polaris-metric-of-companies.png)*그림3) 여러 기업의 북극성 지표 / 출처 : [https://growwithward.com/north-star-metric/](https://growwithward.com/north-star-metric/)*

북극성 지표를 증분시키기 위해 해볼 수 있는 액션은 너무도 다양할 것입니다. 우리가 행할 액션을 조금 더 구체적으로 설정하기 위해 북극성 지표와 상관 관계에 있는 투입 지표를 추가로 고려해볼 수 있습니다. 

이 투입 지표가 북극성 지표에 어떤 영향을 주는지, 북극성 지표가 영향 미칠 매출엔 어떤 영향을 주는지 살펴봄과 동시 액션을 조정해 가며 목표하는 방향으로 나아갈 수 있을 것입니다.

아래는 음악 스트리밍 서비스인 Spotify의 북극성 지표와 상관 관계에 있는 투입 지표 예시입니다. 

![그림4) Spotify의 북극성 지표, 투입 지표](/img/important-customer/polaris-metric-of-spotify.png)*그림4) Spotify의 북극성 지표, 투입 지표 / 출처 : [https://growwithward.com/north-star-metric/](https://growwithward.com/north-star-metric/)*

Spotify가 최종 목표를 ‘매출'로 바라보고 있는지 정확히 알 수 없지만 ‘음악 재생 시간'으로 북극성 지표를 설정하였으며 투입 지표로 ‘더 자주 사용자가 앱을 방문하도록 하는 것', ‘사용자가 앱에서 보내는 시간을 더 길게 하는것'에 집중하고 있음을 알 수 있습니다.

이렇게 되면 방문률, 사용자가 앱에 머문 시간을 뜻하는 세션 시간에 집중하여 이 두 지표를 증가 시키기 위한 액션을 고려할 수 있게 되는 것이죠.

그럼 Spotify의 예시를 살펴봤으니 제품, 고객, 비즈니스 등 다양한 관점을 포함하여 제품 측면에서 쏘카의 북극성 지표를 위해 다음의 기준에 의해 함께 설정해 보도록 하겠습니다. 
> 설정된 지표는 쏘카의 공식적인 북극성 지표가 아닌 개인 과제로 도출된 지표입니다.

- 고객의 성공 순간을 반영했는가? 
  - 쏘카에서 고객들의 ‘성공' 순간은 쏘카를 통해 편리한 경험을 얻고, 만족하는 순간임. 그러기 위해선 단순 ‘예약'에만 그치면 안되고 차량 반납을 통해 ‘경험을 완료했다.’는 근거가 있어야함 = `‘반납'`
- 모든 고객이 ‘가치'를 얻는 내용인가? 
  - 쏘카에서 ‘가치'란 쏘카에서 제공되는 제품인 앱과 차를 통한 이동 경험에서 얻을 수 있는 것임. 더불어 “가치를 얻었다.”란 앱과 차를 통한 경험이 완수된 시점 기준이 포함되어야 함. 경험이 완료되지 않았다면 정확히 가치를 100% 경험했다, 얻었다고 보기는 힘듬. = `‘반납’`
- 측정 가능한가? 
  - 일,주,월별로 측정 가능함. = `‘반납 건 수’`
- 투입 지표를 통해 통제 가능한 요인인가? 
  - 앱과 차를 통한 경험 과정[예약 전(앱) > 이용 중 및 반납(앱&차량)]에서 과정을 세분화해 관련된 다양한 투입 지표를 통해 북극성 지표를 통제할 수 있음. = 다양한 투입 지표에 따른 `반납 건 수의 증감을 확인할 수 있음`
- 비즈니스 관점에서 성장을 직접 반영하는가? 
  - 예약이 발생한 시점에선 고객이 언제든지 예약 취소를 할 수 있으므로 비즈니스 관점에서 성장이 발생하지 않음. 하지만 ‘반납' 시 `고객은 이용료를 지불해야 하므로 비즈니스의 성장을 반영`한다고 볼 수 있음.
- 빠르게 판단할 수 있는가? 
  - ‘반납 건 수' 조회 기준을 `시간/일자별로 설정하여 빠르게 판단`할 수 있음.

위 기준에 따라 최종적으로 도출한 제품 측면에서 쏘카의 북극성 지표를 `‘월 반납 건수 증분'`이라고 결론 지어 볼 수 있었습니다. 

![그림5) 제품 측면에서 쏘카의 북극성 지표](/img/important-customer/polaris-metric-of-socar.png)*그림5) 제품 측면에서 쏘카의 북극성 지표*

> **`쏘카 프로덕트 본부는 이렇게 일해요 1 : 자발적인 스터디`** <br > 
북극성 지표, 선행&후행지표, 투입 지표..모두 똑같은 ‘지표’이지만 위계가 다른데요. 
저 역시 쏘카 프로덕트 본부의 일원이 되기 전엔 이러한 개념에 대해 알고 있지 못했습니다. 😉
쏘카 프로덕트 본부에선 실무를 진행하며 부족한 부분이 있다고 느낄 경우 누구나, 자발적으로 
스터디를 만들 수 있는데요. 지금도 그로스 해킹, 가격 전략, 심리, 데이터 분석 등 다양한 주제로 활발히 
진행되고 있습니다!
참고로 다양한 지표에 대한 개념은 그로스 해킹 스터디에서 동료들과 위클리 스터디를 통해 개념을 익히고 진행 중인 프로젝트에 대입하며 고민하는 시간을 통해 숙지하게 되었습니다.


### 2.2 중요한 고객은 누구인가?

그럼 중요한 고객은 누구일까요?  장기적으로 중요한 고객의 배경을 ‘제품'으로 설정하였기 때문에 우리에게는 ‘제품 시각'으로 사고하고 고민하여 그 안에 존재할 수 있는 중요한 고객을 찾는 과정이 필요합니다. 

제품 시각은 곧 제품을 이용하는 고객 중심으로 사고하는 ‘고객 관점'과 이를 지속가능하게 제공할 수 있는 사업 측면에서의 ‘비즈니스 관점'을 함께 고려하는 것을 의미합니다. 

획기적인 기술, 말도 안 되게 편리하고 아름다운 UI/UX를 제공한다 하더라도 그것이 우리 제품을 사용하는 고객의 니즈가 아니라면 그들에게 우리 제품은 혁신적인 것도, 사용할만한 것도 아니게 될 가능성이 큽니다.

하지만 여기서 중요한 것은 고객 관점으로만 제품을 제작하면 안 된다는 것입니다. 고객 관점을 초석으로 발견한 고객의 니즈(문제)는 비즈니스 관점에선 곧 기회입니다. 이 기회를 활용해 사업적으로 얻고자 하는 것은 무엇인지,  “어디로, 어떻게 고객을 드라이브 할 것인가?” 라는 목표점이 있어야 시장 존속 가능한 제품 제작이 가능하며 이를  통해 얻은 가치를 고객에게 환원하여 시장에서 오랫동안 살아남는 제품을 만들 수 있습니다. 

사용자 측면의 고객 관점과 공급자 측면의 비즈니스 관점을 모두 고려한 ‘제품 시각', 더 나아가 제품 시각에서 가장 중요한 고객을 찾았을 때 비로소 제품을 통해 가치를 얻는 고객 수의 증분과 이를 통한 매출, 브랜드 이미지 확산을 통한 투자로 성장/성숙기는 길어질 수 있으며 쇠퇴기의 도래를 막을 수 있을 것입니다. 

>  **`쏘카 프로덕트 본부는 이렇게 일해요 2 : 함께 자랍니다.`** <br > 
처음 과제를 진행할 때 생각했던 포인트는 [북극성 지표에 따른 중요한 고객 찾기]였습니다. 
하지만 스스로 생각하기에도 “비약이 있는 것 같은데?”란 느낌을 받았는데요.
이에 리더, 동료들에게 *“지금 중요한 고객을 찾기 위해 과제를 진행하고 있는데 비약이 있는 것 같아요! 왜 저 스스로도 비약이 있는 것 같다고 느낄까요?”*라고 솔직한 고민으로 대화를 나누며 힌트를 얻을 수 있었습니다.
> - 안톤 :  버키! 고객 관점을 세분화하기 위해 고객이 제품을 이용하면서 겪는 단계인 퍼널(Funnel)로 쪼개어서 고민해 보는 건 어떨까요? 과거에 관련한 내용이 있었는데 이거 공유드릴게요 참고해 보세요!
> - 타미 : 고객 관점만으로 비약이 있다고 느껴진다면 상반되는 개념인 비즈니스 관점도 생각해 보는건 어떨까요? 우리가 고객에게 뭘 원하는지, 고객은 뭘 원하는지 함께 살펴보면 그 접점에서 우리가 집중해야 하는 부분을 찾을 수 있을 것 같아요!


![그림6) 제품 시각에서 중요한 고객을 왜 찾아야 하는가?](/img/important-customer/customer-diagram.png)*그림6) 제품 시각에서 중요한 고객을 왜 찾아야 하는가?*

그림6의 벤다이어 그램을 통해 비즈니스 관점과 고객 관점의 교차점에 있는 중요한 고객을 왜 찾아야 하는지 조금 더 살펴보겠습니다. 

- a\b  (오직 비즈니스 관점에서만 고려한 중요한 고객)
    * `정의` 
      - 신규 사용자 유치, 매출 증분 등 오직 비즈니스 관점의 성공을 위해서만 제품을 개선할 가능성이 높아짐
    * `결과 예시`
      - 오직 ‘많은 수의' 신규 사용자 유치만을 위해 이벤트, 프로모션을 상시 진행하고 이후 사용자 행동 패턴 분석 및 후속조치는 고려하지 않는다.
      - 오직 ‘매출'만을 위해 지역별 수요는 고려하지 않고 모든 차량에 대한 가격 인상을 단행한다.
- b\a (오직 고객 관점에서만 고려한 중요한 고객)
    * `정의` 
      - 앱리뷰, 인터뷰, 사용성 등을 통해 ‘고객’이 불편을 겪는 부분으로만 제품을 개선할 가능성이 높아짐
    * `결과 예시`
      - 특정 지역에 ‘쏘카존이 부족하다.’는 의견만 반영하여 해당 지역에만 쏘카존을 증설한다.
      - ‘특정 차량을 타고 싶다.’는 의견만 반영하여 해당 차량을 구매 및 제공한다.
- a**∩b (비즈니스 관점과 고객관점 모두를 고려한 교집합에 있는 고객군 = 중요한 고객)**
    * `정의` 
      - 비즈니스 관점 및 고객 관점을 모두 고려하여 제품을 개선하고 북극성 지표를 통해 이를 검증하며 전략을 수정할 수 있음
    * `결과 예시`
      - 고객 관점에서 **주중에 쏘카를 사용하는 고객들이 ‘쿠폰'사용에 어려움**을 ****겪고 있다는 것을 발견, 비즈니스 관점에서 ‘쿠폰' 사용이 가장 저조한 지역, 차종 등을 고려하여 새로운 쿠폰 기획을 통해 ‘쿠폰 사용성' 측면에서 효과를 검증해봄 → 사용성 증대 시 이후 전지역, 전차종으로 영향 범위 확대
      - 고객관점에서 ‘여행지에선 조금 더 많은 인원이 탑승 가능한 차량'을 원한다는 니즈를 발견, 비즈니스 관점에서 가장 대여가 활발한 여행 지역을 선정하여 차종을 다양화 한 후 ‘대여 기준'에서 효과를 검증해 봄 → 효과를 입증한 후 ‘여행지'라는 지역의 특수성을 고려하여 [여름맞이 캠핑카 프로모션] 등 차종 다양화, 마케팅 전략으로 영향 범위 확대

## 3. 고객 관점에서 장기적으로 중요한 고객 찾기

그럼 이제부터 고객 관점에서 장기적으로 중요한 고객이 누구인지 한번 찾아보도록 하겠습니다. 

아래 그림7은 고객 관점에서 중요한 고객을 찾기 위해 진행했던 단계입니다.

![그림7) 고객 관점에서 장기적으로 중요한 고객 찾기 진행 단계](/img/important-customer/customer-view-stage.png)*그림7) 고객 관점에서 장기적으로 중요한 고객 찾기 진행 단계*

중요한 고객을 마케팅, 사업, 운영 관점에서 고민해 볼 수도 있지만 이번 글에선 사고의 주체가 PM이기 때문에 제품 관점에 초점을 맞춰 고객이 제품을 이용하면서 겪는 단계인 퍼널(Funnel)을 기준으로 세분화하여 찾아보겠습니다.

고객이 앱을 이용하면서 겪는 단계를 세분화하기 위해선 ‘고객 여정 지도'라는 개념을 이용해 볼 수 있습니다. 
고객 여정 지도란 고객이 서비스 또는 제품과 관련된 모든 접점에서 고객의 경험하는 여정을 시각화한 단계를 말합니다. 고객 여정 지도를 활용함으로써 사고 기준을 고객에 둘 수 있으며 이를 통해 고객 중심 가치로 사고할 수 있습니다. 

이를 통해 쏘카의 고객 여정 지도를 크게 다음은 부분으로 나눠볼 수 있었습니다. 더불어 퍼널을 고려할 때 고객 관점 외에 해적 지표, 비즈니스 관점을 추가함으로써 어디로 고객의 니즈를 드라이브할 수 있을지 대략적으로 고려할 수 있었습니다. 

![그림8) 쏘카를 이용하기 위한 고객 여정 지도](/img/important-customer/funnel-stages.png)*그림8) 쏘카를 이용하기 위한 고객 여정 지도*

- 쏘카 이용을 위한 가입/로그인
- 차량 예약을 위한 탐색/조회
- 결정을 위한 대여 정보 확인
- 결제를 위한 결제 정보 확인
- 차량을 찾기 위한 운행 전 단계
- 차량을 찾은 후의 운행 중 단계
- 차량 반납을 위한 운행 후/반납 단계

> 💡 **`막간 지식타임`**  <br > 
해적 지표란? 기업이 사업의 성장을 평가하기 위해 추적해야 할 5가지 사용자 행동 지표인 획득, 활성화, 유지, 추천, 수익의 이니셜을 딴 약자입니다.
> 
> **해적 지표의 목적은 다음과 같이 크게 2가지가 있습니다.** 
> 1. 기업이 비즈니스의 건전성에 직접적인 영향을 미칠 수 있는 지표에만 집중할 수 있도록 합니다. 
> 2. 올바른 데이터를 사용하여 제품 관리 및 마케팅 노력의 성공을 가늠할 수 있도록 하고, 효과가 없는 마케팅 계획을 개선하는 데 도움이 됩니다. <br > 
> 출처 | [https://mixpanel.com/ko/blog/aarrr-pirate-metrics/](https://mixpanel.com/ko/blog/aarrr-pirate-metrics/)

여러 퍼널 중 한 단계인 ‘탐색/조회'에 집중하여 과정을 설명드리겠습니다.

1. 고객여정지도에 의해 퍼널을 분리한 뒤, 각 퍼널에서 고객이 해야 하거나 할 수 있는 행동들 및 니즈를 나열했습니다. 
    - 고객이 할 수 있는 행동
        
        탐색/조회 단계에서 고객이 최종적으로 내릴 수 있는 결정을 위해 ‘할 수 있는' 행동들을 아이데이션 하였습니다.
        
    - 해당 단계에서 고객의 니즈
        
        탐색/조회 단계에서 고객이 최종적으로 내릴 수 있는 결정을 위한 과정에서 가질 수 있는 니즈를 앱 리뷰, CS 내용, 동료 인터뷰 내용 중 일부를 참고함으로서 갈음하였습니다. 
        
        (실무에선 고객 니즈를 발굴할 때 설문조사, 인터뷰, 소수의 응답자 집단을 대상으로 비체계적이고 자연스러운 분위기로 조사목적과 관련된 토론으로 의견을 받는 FGI, 데이터 분석 등을 활용하곤 합니다.)
        

![그림9) 탐색/조회 단계에서 고객이 할 수 있는 행동 및 고객의 니즈](/img/important-customer/searching-stage.png)*그림9) 탐색/조회 단계에서 고객이 할 수 있는 행동 및 고객의 니즈*

1. 탐색/조회 퍼널 내 존재할 수 있으며, 위에서 나열한 니즈를 가질 수 있는 세그먼트를 우선 카테고리 별로 나열하고 카테고리가 가질 수 있는 세그먼트로 세분화 하였습니다. 
    
    e.g. 멤버십 여부에 따라 → 멤버십이 있는(패스포트 구독자) / 없는(비구독자)
    
    ![그림10) 탐색/조회 단계에서 니즈를 가질 수 있는 다양한 세그먼트](/img/important-customer/searching-stage-segments.png)*그림10) 탐색/조회 단계에서 니즈를 가질 수 있는 다양한 세그먼트*
    

1. 그 후 북극성 지표인 ‘월 반납 건 수'를 위해 액션을 전개 했을 때 작은 액션에도 큰 영향을 받을 수 있는 세그먼트를 고려하기 위해 모수(규모) 기준으로 세그먼트를 다음과 같이 분류하였습니다. 
    - 일반적인 경우 = Mass 고객 (임팩트가 큰)
      - `정의` : 특정한 니즈, 페인포인트 없이 일반적인 상황에 있는 고객군을 의미합니다. 
      ex) 개인회원, 신규가입한 회원, 준회원(카드/면허 등록 여부에 따라)
    
    - 특이한 상황 속에 있는 경우 = Specific 고객 (임팩트가 적은)
      - `정의` Mass 고객과는 반대로 모수 규모가 크지 않은, 즉 특이 상황에 있는 고객군을 의미합니다. 
      ex) 사고가 발생한 경우, 등록한 카드로 결제가 불가한 경우, 면허 등록 시 다양한 이유로 보류 처리된 회원
    
    - 분류 과정 중 고객이 가질 수 있는 ‘성격'으로 볼 수 있는 경우 = 고객의 요소
      - `정의` 위 두개의 고객군을 나눈 기준과 별개로 고객이 가질 수 있는 성격(특성)을 의미합니다. 
      ex) 경험 빈도 (예약경험이 전무한, 회원 평균 대비 많은/적은, 예약경험은 있으나 반납경험이 없는) 등
    

![그림11) 다양한 세그먼트를 일반적인 경우와 특이한 경우로 분리](/img/important-customer/mass-and-specific.png)*그림11) 다양한 세그먼트를 일반적인 경우와 특이한 경우로 분리*

1. 그다음 고려하지 못한 세그먼트 케이스가 없도록 최대한 다양하게 분류하기 위해 일반적인 경우(Mass 고객, 임팩트가 큰)와 고객의 요소를 조합해 다양한 세그먼트를 고려해볼 수 있었습니다. 
    
    ![그림13) 일반적인 경우와 조합될 고객의 요소](/img/important-customer/various-segments.png)*그림13) 일반적인 경우와 조합될 고객의 요소*
    

| 조합 결과 No. | Mass 고객 + 고객의 요소 조합 내용                                                                                       | 조합을 통해 새롭게 지정한 세그먼트유형 |
| --- |--------------------------------------------------------------------------------------------------------------| --- |
| 1 | - 개인 <br > - 신규가입한 - 카드/면허 등록이 완료되지 않았으며 <br > - 예약/반납 횟수가 없으며 <br > - 방문 횟수 = 탐색/조회 횟수가 비슷한                 | 둘러보기만 하고 있는 초기 탐색자 유형 |
| 2 | - 개인 <br > - 가입한 지 n일이 지났으며 <br > - 준회원 (카드 등록만 완료한) <br > - 예약/반납 횟수가 없으며  <br > - 방문 횟수 = 탐색/조회 횟수가 비슷한    | 가입한 지 시간이 지났음에도 뚜렷한 이용의지가 보이지 않는 방관자 유형 |
| 3 | - 개인 <br > - 일반회원 (카드/면허 등록 완료한) <br > - 방문/탐색 조회 횟수 대비 예약 횟수가 많고  <br > - 이용 텀이 평균 대비 짧은                    | 앱 방문 후 예약하는 비율이 높고, 자주 쏘카를 이용하는 쓸 때가 됐어 유형 |
| 4 | - 개인 <br > - 일반회원 (카드/면허 등록 완료한) <br > - 방문/탐색 조회 횟수 대비 예약 횟수가 적고  <br > - 이용 텀이 평균 대비 긴                     | 앱 방문 후 예약하는 비율이 적고, 쏘카를 다른 사용자 대비 가끔 이용하는 내가 원할 때 쓰는 유형 |
| 5 | - 개인 <br > - 일반회원 (카드/면허 등록 완료한) <br > - 방문/탐색 조회 횟수가 평균 대비 많으며  <br > - 방문/탐색 조회 횟수 대비 예약 횟수는 적은            | 다른 사용자 대비 방문과 탐색/조회는 자주 하지만 예약까지는 이어지지 않는 뭘 망설일까 유형 |
| 6 | - 개인 <br > - 일반회원 (카드/면허 등록 완료한) <br > - 방문 횟수가 평균 대비 많지만 <br > - 방문/탐색 조회 횟수 대비 예약 횟수는 적은  <br > - 이용 텀이 평균 대비 짧은 | 다른 사용자 대비 방문과 탐색/조회 자주 하지만 예약까지는 쉽게 이어지지 않는, 그러나 뭘 망설일까 유형보단 이용 텀이 짧은 쓸 때가 됐나 유형 |
| 7 | - 개인 <br > - 일반회원 (카드/면허 등록 완료한) <br > - 방문 횟수가 평균 대비 많지만  <br > - 탐색 조회 및 예약 횟수는 평균 대비 적은                   | 다른 사용자 대비 방문만 많이 하고 탐색/조회, 예약은 적은 들어는 와본다 유형 |
| 8 | - 개인 <br > - 일반회원 (카드/면허 등록 완료한) <br > - 방문/탐색 조회 횟수 평균 대비 적은  <br > - 이용 텀이 평균 대비 긴                         | 가입한 지 시간도 어느 정도 됐고, 카드/면허 등록도 완료했지만 쏘카를 가끔씩 이용하는 필요할 때가 좀 적구나 유형 |

표1) 일반적인 경우와 고객의 요소로 조합된 세그먼트 

그 후 북극성 지표인 ‘월 반납 건수’에 큰 영향을 미칠 수 있는 세그먼트(=중요한 고객)에서 집중해야 하는 고객을 직관적으로 찾아낼 수 있도록 x축은 예약 시도 횟수, y축은 운행 완료 횟수인 사분면을 구성했으며, 
발견한 세그먼트들을 위치시켜 어느 분면에 위치한 세그먼트가 제품의 성장/성숙기를 길게 만들어 줄 수 있는 장기적으로 중요한 고객인지 살펴보았습니다.

*실제 업무 시 데이터 사이언티스트 분들과 함께 협업하며 가설을 세우고, 데이터로 검증하며 임팩트 산출까지 하는 것이 일반적이나, 이번 글은 ‘사고하는 법'에 초점이 맞춰져 있으므로 데이터 검증 전, 사고 과정에서 도출된 중요 고객으로 갈음하였습니다.

![그림14) 조합으로 새롭게 발견한 세그먼트를 사분면 위에 위치 시킨 모습](/img/important-customer/quadrant.png)*그림14) 조합으로 새롭게 발견한 세그먼트를 사분면 위에 위치 시킨 모습*

| No. | 사분면 이름 | 정의  | 중요도 고려사항      |
| --- | --- |-------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------|
| 1 | 자연 증분하는 유형 | - 방문/탐색 조회를 통해 예약 시도하는 횟수가 많으며  <br > - 그렇게 시도한 경험이 반납 횟수까지 영향을 미치는 유형 | - 공급자 관점에서 큰 개선을 제공하지 않아도, 쏘카의 가치가 일상생활에서 필요한 유형  <br > - Loyalty에 대한 보상, 할인 등을 통해 끈끈한 관계를 이어나갈 수 있는 유형    |
| 2 | 액션이 필요한 유형 | - 방문/탐색 조회를 통해 예약 시도하는 횟수가 많지만  <br > - 그렇게 시도한 경험이 반납 횟수로는 이어지지 않는 유형 | - 방문/탐색은 활발히 하나 만족되지 않는 ‘페인 포인트’가 있어 예약/반납까진 이어지지 않는 유형 <br > - 공급자 관점에서 해당 세그먼트를 분석해 제품 안에서 개선책을 제공해야 하는 유형  <br > - 쏘카 제품에서 가장 많은 비율을 차지할 것으로 고려되는 유형 |
| 3 | 사용자 유지 전략이 필요한 유형 | - 방문/탐색 조회를 자주 하진 않지만,  <br > - 평균 대비 조금 더 긴 텀으로 쏘카를 이용하는 유형 | - 공급자 관점에서 제품을 개선하기 보단 마케팅 전략이 조금 더 필요할 것으로 고려되는 유형  <br > - 제품 관점에서만 개선 사항을 고려하긴 힘들 것으로 추측되는 유형   |
| 4 | 임팩트가 적은 유형 | - 방문/탐색 조회를 자주 하지 않고  <br > - 예약/반납 횟수도 거의 없는 유형 | - 공급자 관점에서 제품을 개선하고, 마케팅 전략을 펼치기 위해 많은 리소스가 투입될 것으로 고려되는 유형 <br > - 개선 보다 신사업 등을 통한 새로운 가치를 제공해야 할 것으로 추측되는 유형  |

표2) 조합으로 새롭게 발견한 세그먼트에 대한 설명

## 4. 비즈니스 관점에서 중요한 고객 찾기

고객 관점으로 중요한 고객을 선별했지만 이들이 “우리(비즈니스) 관점에서도 정말 장기적으로 중요한 고객일까?”는 추가로 고민이 필요합니다. 이에 비즈니스 관점에서 중요한 고객을 한 번 더 찾아보는 과정을 진행했습니다. (고객 관점과 마찬가지로 ‘탐색/조회'단계 내용을 나눠보고자 합니다.)

![그림15) 비즈니스 관점에서 중요한 고객 찾기 진행 단계 ](/img/important-customer/business-view-stage.png)*그림15) 비즈니스 관점에서 중요한 고객 찾기 진행 단계*

1. 첫 번째로 진행한 사항은 비즈니스 관점에서 퍼널 별 목표로 하는 성과 지표가 무엇인지 설정하는 것입니다. 
    
    성과 지표 먼저 설정하는 이유는 뚜렷한 목표 지점을 통해 우리가 고객에게 원하는 것이 무엇인지 명확하게 생각할 수 있으며, 개선하기 위한 액션 플랜도 명확한 **성과 지표를 기반**으로 설정할 수 있기 때문입니다. 
    
    단, 여기서 염두 해야 하는 것은 우리는 ‘월 반납 건수 증분'이라는 북극성 지표를 설정했기 때문에 퍼널 별 설정하는 성과 지표가 북극성 지표로 귀결될 수 있도록 해야 한다는 것입니다. (성과 지표는 앞에서 설명했던 투입지표와 동일한 맥락을 가집니다.)
    
    하지만 성과지표만을 목표로 지정하면 이를 위해 우리가 할 수 있는 액션은 무궁무진 할 것입니다. 이에 구체적인 액션 플랜 수립이 가능하며 성과 지표와 상관 관계에 있는 선행지표를 통해 액션의 방향성이 옳은지 빠르게 판단할 수 있습니다.
    아래는 ‘월 반납 건 수 증분'이라는 북극성 지표 증분을 위해 상관 관계에 있는 탐색/조회 단계에서의 성과지표, 성과지표 증분을 위해 세분화 해볼 수 있는 선행 지표 예시입니다. 
    
    ![그림16) 북극성 지표, 퍼널 별 성과지표 이와 상관 관계에 있는 선행지표](/img/important-customer/leading-indicators.png)*그림16) 북극성 지표, 퍼널 별 성과지표 이와 상관 관계에 있는 선행지표*
    

1. 탐색/조회 단계에서의 선행지표를 정했으므로 선행지표와 영향 받을 성과지표를 고려해 고객에게 바라는 행동 목표는 무엇인지 편하게 적은 뒤, 왜 이것을 바라는지 **근본적인 원인을 찾기 위해 5 whys 도구를 이용**하였습니다.

    > 💡 **`막간 지식타임`**
    > 5 whys는 도요타의 taiichi ohno가 체계적인 문제 해결을 위해 개발한 도구입니다. 기본적인 형태는 문제에 대해 ‘왜’라고 질문하고, 거기서 나온 답에 대해 다시‘왜’라는 질문을 5번 던짐으로서 문제의 근원을 찾아나가는 것입니다.
    
    ![그림17) 비즈니스 관점에서 고객에게 바라는 것 5 whys로 파고들기](/img/important-customer/5-whys.png)*그림17) 비즈니스 관점에서 고객에게 바라는 것 5 whys로 파고들기*
    

1. 마지막으로 5 whys로 구체화한 원인에 집중하여 이 문제를 해결할 수 있으며 & 해결 시 설정한 선행지표와 성과 지표에 긍정적인 영향을 줄 수 있는 액션 플랜을 아이데이션해 볼 수 있었습니다. 
    
    ![그림18) 5 whys로 구체화한 원인과 이를 해결하기 위한 액션 플랜 아이데이션, 결과 목표인 선행 지표 연결](/img/important-customer/action-plan-ideation.png)*그림18) 5 whys로 구체화한 원인과 이를 해결하기 위한 액션 플랜 아이데이션, 결과 목표인 선행 지표 연결*
    
    이때 아이데이션한 아이템이 문제를 직접적으로 해결할 수 있는 것인지, 간접적으로 해결할 수 있는 것인지 고민해 보며 액션 플랜을 전개했을 때 선행&성과 지표가 얼마나 영향 받을지도 대략 추측해 볼 수도 있습니다.
    
    ![그림19) 비즈니스 관점에서 도출한 전략 Map](/img/important-customer/strategy-map.png)*그림19) 비즈니스 관점에서 도출한 전략 Map*
    
    비즈니스 관점에서 선행, 성과 지표 설정 및 행동 목표 구체화와 이를 통한 액션 플랜 아이데이션까지 해보며  위와 같은 전략 Map을 그려볼 수 있었습니다. 
    
    하지만 아직까지 장기적인 관점에서 중요한 고객 군은 찾지 못했는데요. 액션 실행의 대상이 되는, 액션을 실행한 후 선행/성과/북극성 지표의 변화와 함께 살펴봐야 하는 중요한 고객은 대체 어떻게 찾을 수 있을까요?
    
    마지막 챕터에서는 앞에서 그려본 고객 관점 ↔ 비즈니스 관점의 교차점을 이용하여 장기적인 관점에서 중요한 고객을 찾는 과정을 살펴보겠습니다. 
    

## 5. 고객 ↔ 비즈니스 관점 싱크하기

### 5.1 교차점 찾기

고객 관점에서 장기적으로 중요한 고객과 비즈니스 관점에서 고객에게 바라는 행동 목표를 다시 정리해 보면 다음과 같습니다. 이를 조금 더 풀어서 고객이 원하는 것과 우리가 원하는 것의 맥락이 같은 지를 살펴보았습니다. 

![그림20) 두가지 관점이 왜 중요한지 복습](/img/important-customer/finding-crossing-point.png)*그림20) 두가지 관점이 왜 중요한지 복습*

장기적으로 중요한 고객을 찾아보는 단계에서 고객 관점에서 찾은 ‘뭘 망설일까' 유형의 특성인 [방문, 탐색/조회를 많이 하지만 예약 횟수가 적다]를 기반으로 세그먼트에 맞춘 니즈를 다시 고민하고, 고객에게 원하는 행동 목표와 대조해 보며 “해당 세그먼트를 정말 장기적으로 중요하다고 볼 수 있을까?”에 대한 질문에 답을 내려볼 수 있었습니다. 

예컨데 비즈니스 관점에서는 ‘빠르게’, ‘원하는'에 방점을 찍어볼 수 있었으며 해당 키워드와 맥락을 같이 하는 고객 관점에서의 니즈가 있음을 확인할 수 있었습니다. 비즈니스 관점에서 ‘빠르게'에 해당하는 고객 니즈는 “예약을 하기 위해 다시/여러번 쏘카존, 차량, 시간 등을 설정하는 일이 없었으면 좋겠다", “예약을 위해 빠르게 조회하고 판단하여 선택하고 싶다.”가 있음을 알 수 있었으며, ‘원하는'에 해당하는 고객 니즈는 “내가 원하는 차(신차, 특가차량, 주유/충전량이 보장된 또는 사고 이력이 많지 않은)를 빌리고 싶다"가 있음을 확인하며 [뭘 망설일까] 유형이 장기적으로 가장 중요한 고객이라고 볼 수 있습니다. 

| 비즈니스 관점 | 고객 관점 = ‘뭘 망설일까’ 유형의 니즈                                                           |
| --- |-----------------------------------------------------------------------------------|
| 빠르게 | - 다시/여러번 ‘조건 설정’ 하는 일이 없었으면 좋겠다. <br > - 예약을 위해 빠르게 조회/선택 하고 싶다.                  |
| 원하는 | - 내가 원하는 차량을 빌리고 싶다.  <br > ㄴ 원하는 : 가격이 저렴한, 경험을 통해 지식이 있는 차량, 청결한, 주유/충전이 되어있는 등 |

표3) 비즈니스 관점 x 고객 관점 싱크해보기

### 5.2 실효성 있는 액션 플랜을 위한 방법 (feat. 코호트 분석)

이제 남은 마지막 단계는 장기적으로 중요한 고객을 세분화하여 해당 고객 군이 정말로 세분화된 기준에 문제를 겪고 있는지 데이터로 살펴보는 과정이 남았습니다.

이 과정을 통해 뾰족한 액션 플랜과 당위성을 함께 얻을 수 있는데요. 데이터로 살펴보기 위해 ‘코호트 분석'을 활용하고자 합니다. 

코호트(동질 집단)이란 동일 기간 대, 공통된 경험을 지닌 고객 집단을 말하며, 코호트 분석이란 이러한 코호트를 특정 주기(일/주/월별)가 경과함에 따라 어떻게 달라지는지 파악하는 것을 의미합니다. 

![그림21) 코호트 분석 예시 (예약전환율)](/img/important-customer/cohort-graph.png)*그림21) 코호트 분석 예시 (예약전환율)*

예를 들어 위의 예약전환율 코호트 차트를 예시로 다음과 같이 코호트 분석을 진행해 볼 수 있습니다.

- 코호트 : 특정 월에 예약을 한 동질 집단
- 코호트 분석 기준 :  예약 전환율 (쏘카앱 방문 대비 결제까지 완료한 비율)
- 분석을 통해 알 수 있는 것
    - A. 2022년 08월에 예약한 사용자들은 시간이 흐름에 따라 어떻게 변할까? 
      - 위 예시 표를 통해선 2022년 8월에 예약을 한 사용자들이 1개월이 경과될 때마다 더욱 많은 비율로 예약을 지속해 나가고 있음을 알 수 있습니다. (8.9% → 9.0% → 10.1%…) 
      - 하지만 예시표와 다르게 시간이 경과됨에 따라 분석 기준인 예약 전환율이 하락한다면 다른 월에 예약한 사용자들과 어떤 차이가 있었는지 살펴보며 대응할 수 있을 것입니다.
    - B. 2022년 08월 및 이후 예약한 사용자들의 예약 전환율에는 어떤 차이가 있을까?
      - 위 예시 표를 기준으로 본다면 시간이 흐름에 따라 예약하는 서로 다른 사용자들이 점점 증가하고 있음을 알 수 있습니다. (8월 : 8.9%, 9월 9.1%, 10월 : 9.9%…) 
      - 하지만 만일 예약을 한 사용자들이 점점 줄어들고 있는 경향을 파악한다면 우리가 행한 액션 플랜을 수정하는 등 의사결정을 할 수 있습니다.
    - C. 기존 사용자(A)의 예약 전환율과 신규 사용자(C)의 예약 전환율에는 어떤 차이가 있을까?
      - 위 예시 표에 의하면 2022년 8월, 9월, 10월 시간이 흐름에 따라 예약 전환율이 점차 증가하고 있음을 확인할 수 있습니다. 
      - 예를 들어 8월에 예약을 한 사용자들(8.9%)이 이용한 앱버전은 14.0.0이라 가정했을 때 9월에 사용자들(9.4%)이 예약을 위해 이용한 앱버전은 14.0.1일 경우, 우리가 수행한 액션 플랜이 예약 전환율에 약 0.5%p정도 증분을 발생시켰다고 해석해볼 수 있을 것입니다. 
      - 하지만 쏘카에선 앱 업데이트 외에도 대여가격&주행요금 인하, 신차 도입, 프로모션, 시즈널리티(휴가철)등 외부 변수가 예약 전환율에 영향을 미쳤을 수도 있기 때문에 이를 주도 면밀히 살펴 해석의 비약을 방지해야 합니다.

탐색/조회 단계에서 특정 기능에 대한 사용성을 측정할 수도 있음에도 코호트 분석을 선택한 이유는 장기적으로 중요한 고객인 ‘뭘 망설일까 유형'을 우리가 행하는 액션에 따른 다양한 코호트로 세분화하고, 경과에 따라 성과 지표로 설정했던 예약 전환율 현황을 살펴보며 코호트 별 차이를 파악하고 개선이 필요한 타이밍에 즉시 액션을 실행할 수 있기 때문입니다. 

이에 다음과 같이 ‘뭘 망설일까 유형’ 세그먼트를 다양한 코호트로 나누고 특정 코호트의 예약 전환율이 하락한다면 적절한 타이밍에 A/B Test로 액션을 실행할 수 있으며 이후 선행/성과지표를 통해 런칭 논의를 할 수 있습니다. 

[탐색/조회 퍼널 안에서 ’뭘 망설일까 유형’을 세분화 한 코호트 예시]

- `경과로 살펴볼 데이터`  성과 지표 중 예약 전환율
- `코호트(예시)`
    1. Map에서 시간설정을 가장 먼저 설정한 집단의 예약 전환율 경과
    2. 차종 필터를 설정한 집단의 예약 전환율 경과
    3. 쏘카존 2개 이상을 클릭한 집단의 예약 전환율 경과
    4. 대여 이력이 있는 차종을 대여 정보 확인 페이지까지 유지한 집단의 예약 전환율 경과

예를 들어 위 코호트 예시 중 [1. Map에서 시간설정를 가장 먼저 설정한 집단의 예약 전환율 경과]를 살펴봤을 때 각 월 별 코호트의 예약전환율이 하락한다면 “시간설정을 가장 먼저 설정하는 것"에 고객들이 문제를 겪거나 어려움이 있음을 추측해볼 수 있습니다. 

현황을 파악한 뒤에는 시간설정과 예약 전환율과의 상관관계에 대한 추가 분석을 데이터 분석팀에 요청하는 등 문제점을 뚜렷하게 확인한 뒤 디자인, 개발 동료와 논의하여 A/B Test로 개선 액션을 실행해 볼 수 있습니다. 
A/B Test를 통해선 개선 액션을 적용한 실험군과 그렇지 않은 대조군의 예약 전환율 경향을 파악하여 액션의 영향도를 확인할 수 있습니다. 

> **`쏘카 프로덕트 본부는 이렇게 일해요 3 : 커뮤니케이션을 통해 협업합니다.`** <br > 
쏘카 프로덕트 본부에는 아이데이션, 논리 검증, 영향도 측정을 위한 데이터 분석 등 다양한 커뮤니케이션 채널과 인프라가 탄탄하게 마련되어 있어 다양한 팀의 동료분들과 목표 달성을 위한 과정을 함께 할 수 있습니다.
쏘카 PM의 실무에 대한 자세한 내용이 궁금하시다면 [[쏘카 PM의 차량 예약 퍼널 단계 개선기(feat. AB TEST]](https://tech.socarcorp.kr/product/2022/06/02/reservation-funnel-improvement-with-abtest.html) 글을 참고해 주세요!


## 6. 느낀 점

과제를 통해 크게 고객 관점, 비즈니스 관점, 다양한 지표 등을 통해 ‘PM적 사고'를 함양해 보는 과정을 가질 수 있었습니다. 서비스 기획자였던 제가 고민을 하는 과정에서 어려움도 존재했는데요. 

어려움을 자유롭게 이야기하고, 다양한 관점으로 ‘함께' 자랄 수 있었던  1on1, 동료 리뷰, 스터디가 있어 과제 완료 및 이 글을 공유할 수 있게 되어 감사한 마음입니다. 

이 과정을 통해 제가 가장 크게 얻게 된 PM적 사고는 제품을 기준으로 전사&프로젝트 목표 맥락안에서 자유롭게 사고할 수 있는 역량이라고 스스로 회고하고 있습니다. 

현재는 소중한 기회인 과제를 통해 함양하게 된 ‘PM적 사고'로 아이템의 로드맵과 방향성 설정부터 다양한 동료들과의 적극적인 협업까지 할 수 있는 예약 전환율 프로젝트 팀에서 실무를 펼쳐나가고 있습니다. 

쏘카 PM으로 마주할 실무를 통해 더욱 무르익어 갈 앞날을 기대하며 혼자가 아닌, 함께 성장하고자 하는 예비 PM 동료분이 있다면 주저말고 함께 할 날을 기대하겠습니다.