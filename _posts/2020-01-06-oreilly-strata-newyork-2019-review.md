---
layout: post
title: "Oreilly Strata Data Conference New York 2019 후기"
subtitle: "Oreilly Strata Data Conference New York 2019 후기"
date: 2020-01-06 00:00:00 +0900
category: data
background: '/assets/images/andrea-enriquez-cousino-4hBCxfrlpoM-unsplash.jpg'
author: kyle
comments: true
tags:
    - machine-learning
    
---



- O'Reilly Strata Data Conference New York 2019 다녀온 후 작성하는 후기입니다. 아래 내용을 다룹니다
	- 스트라타 데이터 컨퍼런스 소개
	- 유익한 세션 소개
	- 참고 자료
- 사내 전파용으로 만든 행사 후기 슬라이드 자료를 [SOCAR Speakerdeck]()에 업로드했습니다! 참고하시면 좋을 것 같습니다 :)


---


### O'Reilly Strata Conference 소개
- O'reilly는 개발 관련된 책을 많이 만들고 있는 출판사로, 아마 개발 관련 책을 보셨다면 O'Reilly 마크를 보셨을 겁니다

![](/img/oreilly-strata-2019-review/1.png){: width="100%" height="100%"}
	
- 출판사지만 동시에 컨퍼런스도 다양하게 열고 있습니다. 매년 다양한 장소에서 여러 테마로 컨퍼런스가 열리고 있습니다
- 2019년엔 Software Architecture, TensorFlow, AI, Data 등 다양하게 열렸습니다

![](/img/oreilly-strata-2019-review/2.png){: width="100%" height="100%"}

- 저희가 이번에 참여한 컨퍼런스는 Data Conference로 데이터에 관한 포괄적인 세션들이 많았습니다
- 이번 New York 행사장은 한국의 코엑스와 비슷한 장소였습니다

![](/img/oreilly-strata-2019-review/4.png){: width="100%" height="100%"}

- 행사장 입장시 안내 데스크에 사람이 있는 한국과 달리, 신청자가 직접 기기를 통해 이름표를 직접 인쇄합니다. 등록한 메일로 QR Code가 전달되는데, 기기에서 QR Code를 입력하면 됩니다

![](/img/oreilly-strata-2019-review/3.png){: width="100%" height="100%"}

- 행사장엔 세션 장소와 기업 부스로 나뉘어져 있었습니다
	- Anaconda 같은 기업 부스도 있었고, Google Cloud Platform 등이 있었습니다
	- ![](/img/oreilly-strata-2019-review/5.png){: width="100%" height="100%"}
	- O'reilly 부스에선 컨퍼런스 참여자 전원에게 원하는 책 1권을 주고 있었습니다. Stream Processing with Apache Spark를 받았습니다
	- ![](/img/oreilly-strata-2019-review/6.png){: width="100%" height="100%"}
- 신기한 광경
	- 1) DJing
		- 음악이 계속 나오고 있는데, DJ가 계셨습니다
		- ![](/img/oreilly-strata-2019-review/7.png){: width="100%" height="100%"}
	- 2) 프로필 사진 촬영
		- 프로필 사진 찍어주는 공간도 있었습니다. 사람들이 줄서서 자신의 프로필 사진을 찍고 있었습니다
		- ![](/img/oreilly-strata-2019-review/8.png){: width="100%" height="100%"}
	- 이런 부분을 통해 행사를 주최한 오라일리가 행사에 얼마나 신경쓰는지를 알 수 있었습니다
	- 3) 세션 종료 후 파티
		- 세션이 끝난 후 기업 부스에서 파티 음식을 제공했습니다
		- 간단한 먹거리, 맥주, 아이스크림 등
		- 개발자와 파티의 결합하는 모습을 보는듯 했습니다
		- ![](/img/oreilly-strata-2019-review/9.png){: width="100%" height="100%"}


## Session 소개
- 다양한 발표들이 있었습니다
	- ![](/img/oreilly-strata-2019-review/10.png){: width="100%" height="100%"}
- 그 중 저희가 들은 세션을 소개드리겠습니다


### How machine Learning meets optimization
- 최적화(Optimization)란 단어는 데이터 사이언스를 공부하다 보면 자주 접할 수 있는 단어입니다. 최적화는 다양한 방식으로 진행할 수 있는데, 산업 공학에선 Linear Programming(LP), Constraint Programming 등으로 문제를 풀 수 있습니다
- 발표자분은 Optimization과 머신러닝이 함께 필요한 문제의 예시로 Scheduling Problem과 Routing Problem을 예시로 들었습니다. 이 문제들은 저희가 속한 모빌리티 산업에서 많이 풀고있는 문제로 제한된 리소스로 최대(혹은 최소)의 효과를 낼 수 있는 문제들입니다
- South West는 Tankering 전략을 최적화해 연간 19million 달러 절감했습니다
	- ![](/img/oreilly-strata-2019-review/11.png){: width="100%" height="100%"}
- 최적화와 머신러닝을 활용하는 방법은 다음과 같습니다. 예측 모델링 후, 그 값을 토대로 최적화를 돌리고 시나리오를 선택합니다
	- ![](/img/oreilly-strata-2019-review/12.png){: width="100%" height="100%"}
- 최적화와 머신러닝 둘 다 Input을 넣으면 Output이 나오는 것은 동일하고, 최적화는 입력 x에 대해 제약 조건과 objective value가 있어 optimal solution을 찾고 머신러닝은 레이블된 데이터가 존재하는 상태에서 Error를 최소화합니다


---

### ML is not enough: Decision automation in the real world
- 이 세션은 위에서 말씀드린 "How machine Learning meets optimization" 같이 원론적인 내용을 다루기보다, 머신러닝과 최적화를 같이 사용한 실용적인 후기에 대한 발표였습니다
- 대부분 현실의 의사 결정은 머신러닝 모델의 예측값만으론 충분하지 않고, 사람의 판단은 Decision Making과 Machine Prediction으로 이루어집니다
	- ![](/img/oreilly-strata-2019-review/13.png){: width="100%" height="100%"}
- Decision Making은 여러 대안 중 가장 Best 행동을 찾는 것이고, Machine Predictions은 미래의 Unknown event를 예측하는 것입니다
	- ![](/img/oreilly-strata-2019-review/14.png){: width="100%" height="100%"}
	- 이 두가지를 조합하기 위해 머신러닝의 예측값을 Decision Engine의 Input으로 넣고 결과를 받도록 설계합니다
	- Decision Engine의 제약 조건(constraint)로 비즈니스 로직상 중요하게 여기는 것들을 넣을 수 있습니다. 예를 들면 특정 제품이 프로모션하고 있다면, 그 부분을 우선적으로 추천해줄 수 있습니다. 그 외에도 새로운 제품에 대한 비즈니스 로직, 재고에 대한 로직 등을 추가할 수 있습니다
	- ![](/img/oreilly-strata-2019-review/15.png){: width="100%" height="100%"}
	- Decision Engine은 Mixed Integer Programming을 사용해 예측 값과 비즈니스 룰을 제약 조건으로 삼고 목적 함수를 최대화하는 값이 나오게 됩니다
- 다른 예시로 Supply Chain(공급 관리) 최적화에 대한 내용을 들었습니다
- 준비하고 있어야 하는 안전한 재고량보다 수요 예측이 높게 될 경우와 낮게 될 경우 생길 비용이 비대칭적이다라는 점을 말하며, 수요 예측 결과에 따른 비용을 다르게 정의했습니다
	- ![](/img/oreilly-strata-2019-review/16.png){: width="100%" height="100%"}
	- ![](/img/oreilly-strata-2019-review/17.png){: width="100%" height="100%"}
- 두번째 예시에선 모델의 예측 값을 Deterministic한(한 숫자로 나오는) 수요를 uncertainty range로 대체하는 내용이 나왔습니다. 일종의 불확실한 예산을 도입한 것입니다. Uncertainty Range를 표현하기 위해 Two quantile regression, 분포를 근사하기 위한 앙상블 활용, 모델의 Output을 명시적으로 분포로 표현(Bayesian structural time series) 등으로 표현하고 Robust Optimization를 사용해 최악의 상황을 피하는 방식으로 최적화했습니다
	- ![](/img/oreilly-strata-2019-review/18.png){: width="100%" height="100%"}

---


### A Practical guide to algorithmic bias and explainability in machine learning
- 이 세션은 MLOps 관련 Github Repository 중 유명한 [Awesome production machine learning](https://github.com/EthicalML/awesome-production-machine-learning) 소속이신 분이 발표해주셨습니다
- 머신러닝 모델을 만들어 Accuracty가 98%가 나와서 Production 환경에 배포할 경우 어떤 문제가 발생할 수 있을까요?
	- 1) 데이터에 Bias가 존재해서 인종에 성차별이 존재할 수 있음 
	- 2) Production 환경에서 정확도 55%
- 우선 편향을 줄이기 위해 모델의 의사결정에 대한 설명이 요구되고 있습니다. 또한 최근 트렌드로 유럽에선 AI 윤리가 법제화가 되고 있습니다. 유럽위원회는 AI의 자동화된 의사결정에 대해 설명받을 수 있는 개인의 권리를 명시합니다
	- ![](/img/oreilly-strata-2019-review/19.png){: width="100%" height="100%"}
	- ![](/img/oreilly-strata-2019-review/20.png){: width="100%" height="100%"}
- 발표자가 속한 Ethical AI라는 조직에선 각 단계에 필요한 오픈소스를 찾고 개발하고 있습니다. 그 중 대표적으로 데이터 분석 단계에선 eXplainable AI(XAI)란 라이브러리를 소개하고, 모델 개발 및 평가 단계에선 ALIBI란 라이브러리에 대해 소개했습니다. 모델을 설명하기 위한 다양한 방식을 알려주며 각각의 방법에 대해 간단하게 알려주었습니다
	- ![](/img/oreilly-strata-2019-review/21.png){: width="100%" height="100%"}
	- ![](/img/oreilly-strata-2019-review/22.png){: width="100%" height="100%"}

---


### 참고 자료
- 쏘카 데이터그룹의 타다데이터팀 카일과 윤이 다녀와서 사내 전파용으로 만든 발표 자료 : [SOCAR Speakerdeck]()에 업로드했습니다
- O'reilly Strata Data Conference 영상을 다시 보는 방법
	- [https://learning.oreilly.com/](https://learning.oreilly.com/) 가입 후 시청 가능합니다. 첫 가입시 1달 무료고, 그 이후엔 구독 비용이 존재합니다
- O'reilly Starata Data Conference New York 발표 자료
	- [https://conferences.oreilly.com/strata/strata-ny/public/schedule/proceedings](https://conferences.oreilly.com/strata/strata-ny/public/schedule/proceedings) 해당 링크에 발표 자료가 업로드되어 있습니다. 모든 발표 자료가 업로드되진 않았지만, 관심있으신 부분에 대한 자료를 보시면 좋을 것 같습니다


