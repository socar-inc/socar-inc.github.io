---
layout: post
title:  "노코드 자동화 툴로 일잘러 데이터 분석가 되기"
subtitle: Integromat 과 Zapier를 활용한 업무 자동화
date: 2023-01-16 10:00:00 +0900
category: data
background : "/img/nocode-tool/bg.jpg"
author: clover
comments: true
tags:
    - data
    - data analyst
    - nocode
    - automtaion
    - zapier
    - integromat
    - make
---
안녕하세요, 비즈니스 데이터 팀에서 데이터 분석가로 일하고 있는 클로버입니다.

비즈니스 데이터 팀은 데이터로 사업/운영/마케팅 다양한 분야의 비즈니스를 직접 개선하여 회사의 성장을 가속화하고 운영을 효율화하여 지속 가능한 가치를 만들어내는 팀으로 다음의 업무를 하고 있습니다.

* 회사 내 데이터 기반 비즈니스 운영 인프라/로직 확보 
* 가격 책정, 할인 정책 개정, 운영 정책 개정, 증배차 알고리즘 제작, 마케팅 개선 등

저는 이 글에서 데이터 분석가(혹은 데이터 사이언티스트) 분들이 노코드 자동화 툴을 사용하여 업무 범위를 확장 시키는 방법에 대해 소개하고자 합니다. (데이터 분석가 뿐만 아니라 모든 직장인 분들에게 도움이 될 것 같습니다.) 

다음과 같은 분들이 읽으면 좋습니다.

- 노코드 툴 활용을 통해 업무 능력을 향상하고 싶은 데이터 분석가
- 데이터 분석가가 아니더라도 노코드 자동화에 관심이 있으신 모든 분
    - 단순 반복 업무 자동화를 통해 업무 효율을 높이고 싶은 분
    - 비즈니스 운영 프로세스 자동화가 필요하신 분
    - 간단한 트리거 설정을 통한 IT 서비스를 만들고 싶으신 분 등

목차는 아래와 같습니다.

1. [들어가며](##-1.-들어가며)
2. [노코드 툴이란?](##-2.노코드-툴이란?)
3. [노코드 툴을 활용해 보자](##-3.-노코드-툴을-활용해-보자)
4. [실제 활용 사례와 전파](##-4.-실제-활용-사례와-전파)
5. [마치며](##5.-마치며)

-----

## 1. 들어가며

### 1.1 데이터 분석가의 업무

비즈니스 운영 인프라/로직을 수립하는 팀의 데이터 분석가로서 일을 하다 보면 다양한 업무를 직면하게 됩니다. 
구체적으로는 데이터 분석을 통한 인사이트 도출, 대시보드 제작, 비즈니스 운영 인프라/로직 수립, A/B 테스트 기획 및 진행, 데이터 적재 등의 다양한 업무를 진행합니다. 
각종 비즈니스 지표를 기반으로 운영 현황을 파악하기도 하고, 운영 정책에서 에러가 발생하면 빠르게 대처하기도 합니다.
또한 더 나은 비즈니스 로직을 만들기 위한 A/B 테스트 및 베타 테스트 등을 진행하기도 합니다. 
![nocode-intro](/img/nocode-tool/1.1_task.png)*1. 에러에 놀라서 달려온 팀원들 / 2. 새로운 로직 도입 진행 / 3. 새로운 로직을 위한 테스트 공유*

### 1.2 업무의 한계

위와 같은 업무를 하다 보면 ‘스프레드시트와 슬랙을 연결시켜서 봇을 만들려면 어떻게 해야 하지?’ 혹은 ‘데이터를 저장해야 하는데 어떻게 매일 같은 시간에 복사를 할 수 있지?’ 등의 고민을 하게 됩니다.
그런데 데이터 분석가 이런 업무를 직접 프로그래밍으로 구현하기엔 한계가 있습니다.
이러한 한계를 극복할 수 있는 방법 중 하나는 노코드 툴을 사용하는 것입니다. 기술적인 백그라운드가 없더라도 노코드 툴을 사용하면 간단한 연동을 통해 업무상 많은 부분을 효율화할 수 있습니다.

-----

## 2. 노코드 툴이란?

### 2.1 노코드 툴에 대한 정의
노코드(No Code) 툴이란 말 그대로 코드 작성 없이 단순한 드래그 앤 드롭과 같은 방식으로 프로그램 혹은 앱 개발을 할 수 있는 툴을 뜻합니다. 
노코드의 활용 범위는 단순한 자동화에서부터 홈페이지 제작, 결제, 앱 제작 등 다양한 작업을 포함합니다. 
대개 이러한 노코드 툴은 SaaS (Software as a Service) 연동을 지원하기 때문에 SaaS 기반의 툴을 많이 사용하는 회사에 근무하신다면 더욱 익숙하게 사용하실 수 있습니다. 
노코드 틀의 전반적인 분류는 아래 사진을 참고해 주시면 좋을 것 같습니다. 

![nocode-state](/img/nocode-tool/2.1_state_of_nocode_tool.png)*[출처 : 광범위한 노코드 툴의 세계](https://www.indiehackers.com/post/state-of-nocode-db1b90cc99)*

> 참고 할 만한 no-code tool 정리 링크 : 
> - [<U>NoCode Journal (노코드 툴 정리 및 소개 사이트)</U>](https://www.nocodejournal.com/state-of-nocode)
> - [<U>노코드 툴로 정리된 노코드 툴 목록</U>](https://airtable.com/shrrIaogPEh9rLNQO/tbl4sLn4bzxQj3YKe) : [<U>출처</u>](https://medium.com/@mtommasi94/no-code-democratizing-software-9421c2208cb])

### 2.2 노코드 툴을 사용하는 이유

현재 저는 현업에서 노코드 툴을 아래와 같이 활용하고 있습니다. 
- 운영 현황 파악 : 매일 가격 정책이 잘 들어갔는지 확인
- 데일리 지표 파악 : 일자별 적용된 가격 정책의 지표 확인
- 간단한 데이터 적재, 파이프라인 생성 등 : 매일 변하는 시트의 데이터를 자동으로 복사하여 데이터를 적재
![nocode-metric](/img/nocode-tool/2.2_metric%20and%20update.png)*지표 현황 파악/데이터 적재*

노코드 툴을 사용하면 코딩 없이도 프로그램이나 앱을 구현할 수 있어 개발 리소스에 의존하지 않고 직접 필요한 툴을 만들 수 있습니다. 
또한 어떤 방식으로 구현할 것인지에 대해 커뮤니케이션하는 시간도 감소하여 리소스 + 시간 모두 아낄 수 있다는 장점이 있습니다. 본인이 필요한 것을 스스로 쉽게 개발할 수 있어 사용자 친화적인 점과 버그 발생 시 수정이 쉬운 것 또한 장점입니다. 

그러나 노코드 툴에서 제공하는 기능 외에는 구현할 수 없다는 한계점도 있습니다. 노코드 툴을 넘어서 로우 코드(Low Code), 그리고 실제 개발까지 넘어가는 것에는 큰 도전이 필요하기 때문입니다. (로우 코드와 노코드의 차이가 궁금하시다면 [<U>이 블로그</U>](https://www.ibm.com/cloud/blog/low-code-vs-no-code)를 참고해 보시면 좋을 것 같습니다.)


### 2.3 소개할 노코드 툴 스포일러

저는 수많은 노코드 툴 중 업무 자동화 툴인 Integromat(현재는 make.com으로 변경) 과 Zapier에 대해 소개해 보려 합니다. (제가 사용했던 경험에 기반하여 작성되어 주관적일 수 있습니다.) 

- Zapier  
    Zapier는 웹 애플리케이션으로 5,000 가지 이상의 앱과 연동하여 사용할 수 있는 워크플로 자동화 툴입니다. Zapier 무료 플랜을 사용할 경우 5개의 무료 Zap(워크플로 자동화 로직) 을 만들 수 있고, 100개의 Task를 수행할 수 있습니다. Trigger → Action으로 구성된 Single-Step Zap만 사용 가능합니다. (2개 초과 조건 설정 불가) 
- Integromat (Make)<br >
    IntegromatI(Make) 또한 웹 애플리케이션으로 1,600가지 이상의 앱과 연동하여 사용할 수 있는 워크플로 자동화 툴입니다. Integromat 무료 플랜을 사용할 경우 2개의 Scenario(워크플로 자동화 로직)를 만들 수 있고 1,000개의 모듈을 사용 가능합니다. 
    

Zapier와 Integromat (Make)는 기본적으로는 같은 구조를 가지고 있지만, 사용하는 용어가 다르기 때문에 실습 내용을 보시기 전 참고하시면 좋을 것 같습니다.

| 용어 | Zapier&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | Integromat (Make) |
| --- | --- | --- |
| 자동화 로직 명칭  | Zap | Scenario |
| 이벤트 명칭 | Trigger | Module |
| 수행 명칭 | Action | Module - <br >Actions, Searches, Triggers<br >Aggregators, Iterators |

-----

## 3. 노코드 툴을 활용해 보자

위에 정리된 표를 보면 이미 눈치채셨을 수도 있지만 Integromat 이 Zapier에 비해서 조금 더 복잡합니다. 따라서 처음 입문하시는 분이라면 Zapier를 통해 노코드 툴에 대해 친숙해지신 뒤 Integromat으로 넘어가시는 것을 추천합니다. 

### 3.1 실습 내용

저는 BigQuery에서 기본적으로 제공해 주는 Dataset을 활용하여 하루 전 날의 검색어 Top 10을 매일 오전 10시에 발송해 주는 Bot 을 제작해 보겠습니다. 예상되는 작업 흐름은 아래와 같습니다.

- 구글 스프레드시트에 BigQuery 데이터를 연동하여 사용할 데이터 생성
- Zapier / Integromat에 스프레드시트 연동
- 전 날의 검색어 Top 10을 Slack 으로 메시지 전송
![nocode-practice](/img/nocode-tool/3.1_sheet.png)*시트명 : top_term*

### 3.2 Zapier 사용해 보기

Zapier 사용을 위해서는 먼저 [Zapier 사이트](https://zapier.com/) 에 회원가입을 진행합니다. 무료 플랜으로 회원가입 후 [Dashboard](https://zapier.com/app/dashboard) 에서 좌측 상단 Create Zap 을 클릭하여 Zap을 만들 수 있습니다. 
![nocode-practice](/img/nocode-tool/3.2_login.png)*회원 가입 및 대시보드 버튼*
Zapier는 자동화 생성을 위해 만든 프로그램을 Zap이라는 단어로 일컫습니다. Zap은  “특정 Trigger 발생 시 특정 Action을 수행한다”는 내용을 담고 있습니다. 
구체적으로 Trigger가 스프레드 시트에 새로운 칼럼 추가, 구글 캘린더에 새로운 일정 추가 등의 특정 조건이라면, 
Action은 Mail 전송, Slack 메시지 전송 등 Trigger 로 인해 실제로 수행되는 작업을 의미합니다. 
![nocode-practice](/img/nocode-tool/3.2_trigger%20and%20action.png)*Trigger / Action*

1. Trigger을 설정합니다. 여기서는 구글 시트가 생성되거나 업데이트 되는 상황을 Trigger 로 지정하겠습니다. 그리고 사용할 구글 계정 세팅 후 `Continue` 를 선택하면 시트 내 데이터를 확인할 수 있습니다.
![nocode-practice](/img/nocode-tool/3.2.1.png)*Trigger 셋팅 -> 스프레드시트 업데이트 / 계정 설정 이후 스프레드시트 연동 /테스트 데이터 확인*

2. Action을 설정합니다. 특정 Trigger 가 만족될 때 Slack 메시지를 전송 할 것이기 때문에 `Send Direct Message`를 선택합니다. 
![nocode-practice](/img/nocode-tool/3.2.2.png)*Action 설정 / 전송할 데이터 설정 / 스케쥴링 옵션*

3. 작업이 완성되었습니다. ![nocode-practice](/img/nocode-tool/3.2.3_zap.png)*Zap의 모습*
![nocode-practice](/img/nocode-tool/3.2.3_final.png)*완성된 Zap 결과물*

이때 아쉽게도 10개의 모든 검색어가 아닌 Rank 10의 값 1개만 슬랙으로 오는 것을 확인할 수 있습니다. 시트에 있는 데이터를 한줄한줄 모두 받아서 데이터를 발송하는 방법은 없었습니다. 
Zapier 에서 제공하는 Action 중 하나인 [Loop](https://help.zapier.com/hc/en-us/articles/8496106701453-Loop-your-Zap-actions#create-a-loop-from-text-0-1) 을 활용하여 반복 수행을 진행해보려 했으나,  받아온 데이터의 형태가 시트 전체의 내용이 아닌 몇개의 행만 가져와지기 때문에 반복 수행을 통한 데이터 전송을 진행할 수 없었습니다. 


### 3.3 Integromat (Make) 사용해보기

이번에는 같은 작업을 Integromat(Make 이지만 편의상 Integromat 으로 부르겠습니다.)을 통해 만들어 보겠습니다. Integromat 사용을 위해 [Integromat(Make) 사이트](https://www.make.com/en) 로 접속하여 회원가입을 진행합니다. 이후 Dashboard의 우측 상단에 있는 `Create a new scenario` 버튼을 클릭하여 Scenario를 제작 할 수 있습니다. 
![nocode-practice](/img/nocode-tool/3.3_login.png)*회원 가입*
Integromat 은 자동화 생성을 위해 만든 프로그램을 Scenario 라고 부릅니다. Scenario 는 여러개의 Module 의 집합으로 이루어지며 module 안에는 해당 moduled의 다양한 활용법이 존재합니다. Module 을 통해 데이터를 불러온 경우, Function 을 사용하여 조건 적용을 진행 할 수 있고, Tools 를 활용하여 반복 수행, 중단 등 다양한 Flow control 를 진행할 수 있습니다.
![nocode-practice](/img/nocode-tool/3.3.png)*Module / Function / Tools - Flow Control*

1. 시트 데이터를 가져오는 Module을 생성합니다. `Google sheet module` 을 선택하여 데이터의 범위를 지정합니다.
![nocode-practice](/img/nocode-tool/3.3.1.png)*Module 생성*

2. Flow Control → Repeater 를 선택하여 테이블 형태의 데이터에 Row 를 1개씩 늘려가며 접근 할 수 있도록 반복 설정을 진행합니다. (조금 까다로운 부분이긴 하나 반복문의 i + 1 을 생성해주는 부분이라고 생각하시면 이해가 쉽습니다.) 

3. Array aggeregator를 사용하여 가져올 데이터를 선정합니다. 저는 날짜, 검색어, 순위를 가져오고 반복을 위해 Repeater의 i 를 가져오겠습니다. 
![nocode-practice](/img/nocode-tool/3.3.2-3.png)*2. Repeater / 3. Array aggregator*

4. Slack module 을 선택한 뒤 계정 연동을 진행합니다. 개인 계정으로 보낼 경우에는 IM channel 을 사용하고, 채널에 전송을 하고 싶을 경우에는 Public / Private 채널을 선택하시면 됩니다.
![nocode-practice](/img/nocode-tool/3.3.4.png)*slack module*

5. Rank : top_term 형태로 출력되도록 repeater + array_aggregator 조합으로 만들어진 array에 접근하여 내용을 가져옵니다. 내용물의 모습은 [1,2,3,4,5,6,7,8,9,10] / [검색어들] 로 이루어져 있다고 상상하시면 됩니다. 따라서 i 를 1개씩 늘려가면서 반복을 할 수 있도록 설정을 해놓은 것으로 생각하시면 됩니다.
![nocode-practice](/img/nocode-tool/3.3.5.png)*message setting*

6. 매일 10시에 스케쥴을 걸고, 하단 시계 모양 버튼을 누른 뒤 schedule 을 on 으로 변경하여 스케쥴링을 걸어줍니다.
![nocode-practice](/img/nocode-tool/3.3.6_scheduling.png)*스케쥴링 옵션*

7. 작업이 완성되었습니다. 처음에 의도했던 1~10위 까지의 검색어를 확인 할 수 있는 결과물이 생성되었습니다.
![nocode-practice](/img/nocode-tool/3.3.7.png)*완성된 Scenario의 모습과 결과물*

### 3.4 Zapier와 Integromat 비교
 
위의 예시를 통해 알수 있듯이 Zapier 는 단순한 조건을 자동화 할 수 있어 입문용 노코드 툴로는 적합하나, 복잡한 조건 구현에는 한계가 있습니다. 
Integromat 은 조건이 복잡한 업무도 자동화 할 수 있으나, 초심자가 작성하기엔 어려울 수 있습니다. 
또한 두 툴의 무료 플랜 조건이 차이가 있기 때문에 아래 표를 참고하셔서 사용하시면 좋을 것 같습니다. 

| 간단 설명 | Zapier | Integromat |
| --- | --- | --- |
| 추천 | 업무 자동화 툴 입문자 | 업무 자동화 툴 익숙자 |
| 추천 업무 | 조건이 단순한 자동화 | 조건이 복잡한 자동화 |
| 워크플로우 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | 선형적 | 유동적 |
| 프로그램 조건 &nbsp;&nbsp;| - Trigger가 무조건 셋팅되어야 함 <br >- 3개 이상의 Trigger 셋팅이 어려움 (webhook 사용을 추천함) | - 제한 없음 |
| 오류 트랙킹 방법  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;| Zap을 실시간으로 재실행 하여 오류가 난 부분을 찾아내야함 | 대시보드를 활용하여 오류가 난 부분을 찾아낼 수 있음 |
| 무료 플랜 비교 | - 매 월 100개의 task 수행 가능 <br >- 5개 Zap 사용 가능 <br >- trigger → Action 으로 구성된 Zap만 사용 가능 (2개 초과 조건 설정 불가)<br >- 1달마다 reset  | - 매 월 1,000개의 module 사용 가능 (1개의 Scenario 에 2개의 Module이 사용 될 경우 500번 수행)<br >- 2개의 Scenario 생성 가능<br >- 1달 마다 reset |

-----

## 4. 실제 활용 사례와 전파

### 4.1 활용 사례

저는 실제 업무에 다음과 같이 노코드 툴을 활용하여 업무의 효율을 높일 수 있었습니다.

이전 회사에서 프로덕트 매니저로 일하던 때, Zapier를 활용하여 많은 CRM 업무 자동화와 비즈니스 파이프라인을 구축하였습니다.
- CRM 업무 자동화<br >
    마케팅 채널을 통해 인입된 예약 건을 스프레드시트로 자동으로 기입되도록 제작합니다. 이후 예약이 생성된 경우 즉 스프레드시트에 Row가 추가될 경우, 슬랙으로 메시지를 전송할 수 있도록 하여 고객 인입을 바로 확인하고 이탈을 방지할 수 있도록 하였습니다.

- 비즈니스 파이프라인 구축<br >
    세일즈 툴을 활용한 업무에 있어서, 파이프라인에서 특정 단계로 옮겨졌을 시 해당 Deal을 다른 파이프라인으로 복사하는 작업을 Zapier를 통해 진행하였습니다. 이를 통해 파이프라인 간 Deal 연동을 통해 하나의 통합된 Deal로 관리를 할 수 있었고, 통합된 Deal을 사용할 수 있었습니다.

현재는 데이터 분석가 업무를 진행하면서 Integromat 을 활용하여 서비스 운영 안정과 임시 데이터 파이프라인을 구축하고 있습니다.
- 안정적인 서비스 운영<br >
    실 서비스에 도입되어 있는 정책이 데이터 파이프라인의 오류로 정책이 적용되지 않았던 일이 있었습니다. Integromat 을 도입하기 전에는 오류 제보나 직접 실시간으로 앱을 확인하지 않는 이상 오류가 발생하였는지도 알 수 없었습니다. 
    그러나 Integromat으로 정책이 잘 적용되었는지 확인하는 봇을 적용하여 지표에 기민하게 반응할 수 있는 환경을 만들었고, 이를 통해 오류 상황이나 정책의 급격한 변동을 모니터링함으로써 안정적인 운영을 진행할 수 있었습니다. 
    ![nocode-practice](/img/nocode-tool/4.1.png)*2022년 신의 한수 / 운영 현황 파악*
- 임시 데이터 파이프라인 구축 (a.k.a 급한 불 끄기)<br >
    급하게 테스트 데이터 적재가 필요하여 팀 내부적으로 빠르게 해결을 해야 하는 경우가 있었습니다. Airflow DAG를 작성하는 방법도 있었으나, 좀 더 가볍고 익숙한 Integromat 을 통해 시트 복제를 활용하여 데이터 임시 적재를 진행하였습니다. 
    ![nocode-practice](/img/nocode-tool/4.1_fire.png)*해결된 급한 불*

### 4.2 노코드툴 전파

실제 업무에 노코드 툴을 활용하면서 얻은 노하우와 팁들을 전파하는 것이 많은 도움이 될 것이라고 생각하여 데이터 비즈니스 본부를 대상으로 노코드 툴에 대한 온보딩 교육을 진행하였습니다. 또한 주변에 노코드 툴에 관심 있는 동료분들에게 개인적으로 교육을 진행하여 노코드 툴을 전파하기도 했습니다. 이를 통해 스스로도 노코드 툴을 깊게 이해하게 되었고 다양한 에러들을 트러블슈팅하면서 툴 활용 능력을 향상시킬 수 있었습니다. 
![nocode-practice](/img/nocode-tool/4.2.png)*온보딩 강의자료 / 수많은 도움 요청*

-----

## 5. 마치며

제가 소개한 노코드 툴의 사용법은 전체 활용 방법의 새 발의 피 수준이라고 생각합니다. 회사마다 사용하는 SaaS 툴의 종류와 활용 방법이 다양한 만큼 노코드 툴도 많은 분들이 사용하면 사용할수록 지식이 공유될 것으로 기대합니다. 만약 제 글을 읽고 노코드 툴을 실제 업무에 적용하셨다면 좋은 경험을 공유해 주세요! 저도 인사이트를 얻어 긍정적인 자극을 받아 가겠습니다.

글 배포에 많은 도움을 준 디니 와 우리 팀 셀라, 알렉스, 모리, 엘모 그리고 저의 Integromat 온보딩 세션을 들어줬던 많은 데이터 비즈니스 본부 분들 마지막으로 긴 글 읽어주신 독자분들 모두 감사합니다. 모두 모두 프로 툴러 되세요! 