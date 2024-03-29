---
layout: post
title: "쏘카에서 기술 블로그를 운영하는 방법"
subtitle: "회사에서 기술 블로그를 작성하는 노하우를 공유합니다"
date: 2023-02-15 09:00:00 +0900
category: data
background: '/img/how-to-organize-tech-blog/thumbnail.jpg' 
author: kyle, dini
comments: true
tags:
    - blog
    
---

  
<br>

안녕하세요. 쏘카 데이터 비즈니스 본부의 카일, 디니입니다.
카일은 2019년부터 2022년 퇴사 전까지 쏘카 기술 블로그를 운영하였고 현재는 데이터 코칭, 데이터 교육을 하고 있습니다. 
디니는 2021년에 입사하여 2022년부터 카일과 함께 기술 블로그를 운영했습니다.

쏘카의 기술 블로그는 2019년도부터 시작해 현재 44개의 글을 퍼블리싱 했습니다. 쏘카에서 기술 블로그를 어떻게 운영했는지 회고하고, IT 기업에서 기술 블로그를 운영하는 분들에게 도움을 드리기 위해 이 글을 작성하였습니다.



예상 독자는 다음과 같습니다

- 개인 블로그가 아닌 회사에서 기술 블로그를 운영해야 하는 분
- DevRel(Developer Relation) 직무에 계신 분
- 쏘카의 기술 블로그 운영 노하우가 궁금하신 분




---

## 목차

1. [쏘카 기술 블로그 운영 목적](#1-쏘카-기술-블로그-운영-목적)
2. [글 작성 플랫폼 선정하기](#2-글-작성-플랫폼-선정하기)
3. [글 작성 프로세스](#3-글-작성-프로세스)
4. [글 배포 후](#4-글-배포-후)
5. [성과](#5-성과)
6. [운영하며 느낀 어려움](#6-운영하며-느낀-어려움)
7. [정리하며](#7-정리하며)

---


## 1. 쏘카 기술 블로그 운영 목적

요즘 많은 IT 회사에서 기술 블로그를 운영하고 있습니다. 회사들이 어떤 맥락에서 기술 블로그를 시작했고, 어떤 목적으로 운영하고 있을까요? 

쏘카에선 기술 블로그를 "쏘카 구성원의 성장의 밑거름으로 활용하기 위해" 만들었습니다. 여기서 성장이라는 단어가 나오는데, 최초에 기술 블로그 운영을 제시해 주신 CTO님께서 기술 조직의 성장을 돕기 위해 블로그를 운영하면 좋겠다는 의견을 제시했습니다. 기술 블로그에 글을 작성할 경우 자신이 진행한 업무를 정리할 수 있으며, 추후 레퍼런스로 활용할 수 있습니다. 

또한 회사에서 어떤 문제를 어떤 방식으로 해결하는지와 해결하는 과정의 여러 생각을 최대한 공유하면 글을 보는 분들이 간접적으로 업무를 경험할 수 있을 것이라 판단했습니다.
회사 차원으로도 기술 블로그에 글을 꾸준히 작성하면 회사에서 어떤 문제를 해결하고 있는지, 어떤 사람들이 있는지 등 회사의 업무와 기술력을 공유할 수 있습니다. 

정리하면 다음과 같습니다

- 기술 조직 구성원의 성장 지원
- IT 개발 업계에 지식 확산
- 회사가 해결하는 문제, 기술력 공유

즉, 기술 블로그 운영은 **글을 작성하는 분과 회사 모두 성장하는 방법**이기 때문에 기술 블로그를 운영하고 있습니다.


## 2. 글 작성 플랫폼 선정하기

기술 블로그를 운영하기 위해 여러 가지를 고민해야 하지만, 제일 먼저 고민한 것은 **글을 어디에 작성할 것인가?**입니다. 기술 블로그 플랫폼은 매우 다양합니다. 직접 웹페이지를 구축하는 방법과 [Medium](https://medium.com/), [Tistory](https://www.tistory.com/) 혹은 [Velog](https://velog.io/) 같은 플랫폼을 사용하는 방법 등이 있습니다.

당시 사내에 [Jekyll](https://jekyllrb.com/) + [Github Pages](https://pages.github.com/)로 기술 블로그를 운영해 본 경험이 있는 사람들이 존재했고, 관련 레퍼런스가 많기 때문에 Jekyll + Github Pages을 사용하기로 했습니다. 특히 처음 기술 블로그를 만들던 2019년엔 가볍게 시작하는 것이 목표였기에 사내에 익숙한 사람들이 존재하는 Jekyll이 적절하다 판단했습니다. 



## 3. 글 작성 프로세스

플랫폼을 선정한 후엔 글을 작성하는 프로세스를 고민했습니다. 처음엔 간단히 "글을 작성할 수 있는 사람이 있다면 누구든 마음껏 자유롭게 작성해도 괜찮다"라고 할 수 있지만, 기술 블로그를 운영하는 사람이 기술 블로그만 운영하지 않고 본업이 있기에 시간을 효율적으로 사용해야 했습니다. 시간을 효율적으로 사용하기 위해 글 작성 프로세스를 만들었고, 그 이후에도 계속 보완해서 발전시키고 있습니다.

우선 글 작성 프로세스는 노션 칸반을 사용해 운영하고 있습니다. 노션 칸반을 사용해 기술 블로그 업무 진행 상황을 볼 수 있는 공간을 만들었습니다.

![notion-kanban](/img/how-to-organize-tech-blog/notion-kanban.png)*기술 블로그 노션 페이지 칸반*

글 작성 프로세스는 크게 다음과 같습니다. 여기서 기술 블로그를 운영하는 사람은 기술 블로그 운영자로, 글을 작성해 주는 직원분은 글 기고자로 지칭하겠습니다.

![writing-process](/img/how-to-organize-tech-blog/writing-process.png)*쏘카 기술 블로그 운영 Process*

1. 글감 등록
2. 글감 리뷰
3. 글 작성
4. 초안 작성 후 1차 피드백
5. 1차 피드백 반영해 글 보완
6. 글 Publish

모든 프로세스는 노션 페이지 및 Github README를 통해서 글 기고자에게 상세히 공유하고 있습니다. 이 글에서도 프로세스의 단계별로 내용을 공유드리겠습니다.


### 3.1 글감 등록

이 단계는 글을 작성해 주는 글 기고자분이 글감을 등록합니다. 글감은 쏘카 기술 조직에서 발생한 일, 조직 문화, 기술 조직의 사람들 등 다양한 이야기를 할 수 있습니다. 글감 리뷰를 통해 글의 큰 흐름을 잡기 때문에 글감을 제한하고 있진 않습니다. 글 기고자분은 슬랙 기술 블로그 채널 또는 노션 칸반에 글감을 추가합니다. 

![writing-material-card](/img/how-to-organize-tech-blog/writing-material-card.png)*[데이터허브 글](https://tech.socarcorp.kr/data/2022/03/16/metdata-platform-02.html)에 대한 글감 카드 내용 예시*

### 3.2 글감 리뷰

글감을 등록한 후, 기술 블로그 운영자와 글 기고자가 만나 글감 리뷰를 진행합니다. 글감 리뷰에선 주로 "어떤 목적을 가지고 내용을 작성하려고 하는지", "글에 대한 내용"을 이야기하며 글감을 더 매력적으로 보이도록 만들기 위한 목적을 가지고 임합니다. 

주로 고려하는 요소는 다음과 같습니다

- 예상 독자
- 이 글을 읽은 독자가 어떻게 변화하길 바라는가?
- 이 글에서 나오는 이야기에서 제일 강조하고 싶은 것은?

위 내용에 대해 대화를 진행하며 내용을 구체화합니다. 이 리뷰를 통해 나온 내용이 글의 큰 뼈대가 됩니다. 

![writing-material-feedback](/img/how-to-organize-tech-blog/writing-material-feedback.png)*[Airflow 글](https://tech.socarcorp.kr/data/2022/11/09/advanced-airflow-for-databiz.html) 글감 피드백 내용*

### 3.3 글 작성 후 리뷰 요청

글감 리뷰가 끝난 후, 글 기고자분은 글 작성을 시작합니다. 최초에 글 작성분이 글 작성 마감일을 스스로 정하고, 그 기간 내에 자유롭게 글을 작성합니다.

글을 작성하는 플랫폼은 Jekyll + Github Page로 진행하고 있다고 위에서 말씀드렸는데, 이 플랫폼에 익숙하신 분과 아닌 분에 따라 글 작성하는 방식이 나뉩니다.

- Github 활용에 익숙한 경우 (개발자 등) 직접 Jekyll에 마크다운으로 글을 작성하고 Fork 한 Github에 Push 합니다
- Github 활용에 익숙하지 않은 경우 (PM, 디자이너 등) 노션에 마크다운으로 글을 작성하면 블로그 운영자가 대신 Jekyll + Github Page에 업로드합니다

Github에 올리기 전에 보안 상으로 문제가 될 부분을 검토합니다. 이때 기술 블로그 운영자도 물론 보안 관련 내용을 검토하지만, 조금 더 정확한 피드백을 위해 글 기고자의 팀장 혹은 팀원들에게 검토를 요청하고 있습니다.

글을 작성할 때 공통적으로 제공하는 팁이나 피드백은 다음과 같습니다.

- 일단 글을 작성해 보세요. 작성을 하는 것 자체가 정말 대단하신 거예요!
- 모호한 표현보단 명확하고 간결한 표현이 좋습니다
- 글감 리뷰 때 이야기한 글을 작성하는 목적과 예상 독자, 독자가 어떻게 변화하길 바라는가에 대해 생각해 보며 글을 작성해 보세요
- 구조화된 생각을 하시며 작성해 보세요
- 중간중간에 이미지나 공백을 활용해 주세요
- 너무 걱정하지 마세요! 피드백을 통해 더 좋은 글을 만들 수 있어요


### 3.4 초안 작성 후 1차 피드백

초안 작성 후, 피드백을 진행합니다. 이 피드백은 대면으로 진행하지 않고 Github Pull Request 또는 노션의 댓글을 통해 진행됩니다.

피드백 과정에서 특히 신경 쓰는 부분은 다음과 같습니다

- 문체
	- 글을 작성하는 스타일을 확인합니다. 글 기고자의 개성을 유지하는 선에서 기술 블로그의 톤 앤 매너를 비슷하게 유지하려고 노력합니다.
	- 비문이 있는지 확인합니다.
- 저작권
	- 이미지 저작권을 확인합니다. 주로 저작권 사용이 자유로운 [Unsplash](https://unsplash.com/)에서 주제와 맞는 이미지를 선정하도록 권고하고 있습니다.

위와 같은 부분을 토대로 Github Pull Request Template을 만들어 활용합니다.

![writing-material-feedback](/img/how-to-organize-tech-blog/github-pr-template.png)*기술 블로그 PR Template*


### 3.5 1차 피드백 반영해 글 보완

1차 피드백에서 나온 내용을 토대로 글 기고자분이 글을 보완합니다. 어느 정도 글이 완성되면 마지막으로 문장을 수정하고 맞춤법을 고치는 등의 퇴고를 진행합니다.

![contents-review-github-pr](/img/how-to-organize-tech-blog/contents-review-github-pr.png)*각 글에 대해서 최대한 많은 피드백을 드리려고 노력하고 있습니다.*


### 3.6 글 배포 - Publish

마지막으로 글을 배포하기 전에 글의 제목과 부제를 고민하고 확정합니다. 
제목은 매우 중요하기 때문에, 검색어 추세 데이터를 확인하며 제목을 정하고 있습니다. [Google Trend](https://trends.google.co.kr/trends/)에 검색어 트렌드를 확인해 보고, [구글](google.com)에서 해당 단어를 입력해서 자동 완성이나 추천 단어 목록을 확인합니다. 이 추천 단어 목록이 많은 사람들이 검색하고 있는 단어라 추천한다고 생각해서 추천해 주는 목록에서 최종 제목을 선정합니다.

또는 아직 국내에 한글로 된 자료가 많이 없는 경우엔 해당 키워드로 제목을 작성하곤 합니다. [쏘카 디자인 시스템 글](https://tech.socarcorp.kr/design/2020/06/23/socar-design-system-01.html)이 이 방식을 사용해 제목을 정했습니다. 

![google-search-output](/img/how-to-organize-tech-blog/google-search-output.png)*구글에서 디자인 시스템을 검색할 때 아래 나오는 키워드가 많이 검색된다고 추측합니다*


디자인 시스템이라 검색하면 디자인 시스템이란, 디자인 시스템 컬러, 디자인 시스템 사례, 디자인 시스템 원칙이란 검색어가 추천됩니다. 만약 추가적으로 디자인 시스템 글을 작성한다면 이 내용으로 글을 작성하면 SEO 관점에서 좋을 것이라 생각합니다.


## 4. 글 배포 후

기술 블로그 운영의 마지막은 글 배포가 아닌, 작성한 글을 외부에 공유하는 일입니다. 글을 배포한 후에 이 글이 더 많이 읽힐 수 있도록 커뮤니티에 공유합니다. 


### 4.1. 콘텐츠 공유

글을 작성한 후 적절한 커뮤니티 또는 SNS에 업로드합니다. 과거엔 페이스북 그룹에 공유했지만 최근엔 링크드인에 업로드하는 것이 효과가 좋은 것 같습니다. 
글이 좋다면 그 이후엔 자연적으로 바이럴이 될 것이라 생각합니다. 블로그에 유입이 많이 진행되면, 그 후엔 SEO를 통해 구글 검색어에 노출되는 것을 목표로 합니다.

이때 링크드인에 몇 시에 올려야 제일 많은 사람들에게 효과가 좋을까를 확인하기 위해 매번 다른 시간에 링크드인에 업로드했습니다. 글 확산엔 여러 가지 요소가 있지만, 저희가 컨트롤할 수 있는 부분은 글의 퀄리티와 시간이라 판단했습니다. 글의 퀄리티는 최선을 다해 올리고 있고, 시간은 여러 가설을 두고 진행했습니다.

예를 들어, 업무 시간에 업로드해야 좋을까? 주말에 업로드해야 좋을까? 출근 직후가 좋을까? 퇴근 전이 좋을까? 등을 생각하며 다르게 Action 했습니다.
배포 타이밍에 정해진 답은 없지만 경험상으로 기술 블로그는 화~목에 접속자가 많았고 주말에 접속자가 확 떨어지는 경향이 있습니다. 따라서 가급적이면 화~목 퇴근 1~2시간 전에 배포할 수 있도록 일정을 조정했습니다.

![weekly-page-view](/img/how-to-organize-tech-blog/weekly-page-view.png)*쏘카 기술 블로그의 일간 페이지 뷰 수는 시즈널리티가 매우 강합니다.*

### 4.2. 데이터를 통한 분석

글을 홍보한 후, 데이터를 확인하며 사람들의 반응을 확인합니다. 쏘카 기술 블로그는 Google Analytics 4를 활용해 지표를 확인하고 있습니다. 다만 AB Test를 진행한 것은 아니기 때문에 추세 정도만 확인합니다. 글을 홍보한 날짜의 일간 페이지 뷰 수가 1000 이상인지 정도를 확인하고 있습니다

추가적으로 Google의 Search Console에서 데이터를 확인합니다. Search Console 개요 상단에 Search Console Insight가 있는데, 여기서 어떤 콘텐츠가 조회 수가 높고 체류 시간이 높은지 확인할 수 있습니다. 또한 인기 검색어를 확인해 어떤 검색어로 기술 블로그에 유입하고 있는지 알 수 있습니다

![search-console-insight](/img/how-to-organize-tech-blog/search-console-insight.png)*Google Search Console Insight*

### 4.3 댓글에 대한 답변

글을 업로드한 후, 글을 보신 분들이 궁금한 점을 댓글에 남기는 경우도 있습니다. 쏘카 기술 블로그의 댓글은 [Utterances](https://utteranc.es/)를 사용하고 있습니다. Github Repository를 생성한 후 연결할 수 있으며, 댓글이 달리면 Github Issue에 저장됩니다. 이 Repo를 Watch 해서 댓글이 달렸을 때 글 기고자분에게 전달합니다.


## 5. 성과

쏘카에선 Google Analytics 4를 사용해 블로그의 지표를 확인하고 있습니다. 그중에서도 기술 블로그가 얼마나 확산되었는지 확인하기 위해 WAU(Weekly Active Users)를 파악하고 있습니다. 
기술 블로그 운영 KPI를 따로 지정하지 않고 운영했지만, 사람들이 쏘카 기술 블로그에 한번 방문하고, 또 방문하면(즉, 리텐션) 그것이 쌓여서 좋은 이미지를 줄 수 있을 것이라 판단했습니다.
또한 WAU를 추적하고 있긴 하지만 오로지 숫자를 기반으로 성과를 측정하기보다는 아직까지는 조직 내에서 다양한 주제로 꾸준히 글을 낼 수 있도록 하는 것이 중요하다고 생각합니다. 
따라서 한 달에 한 번 정도 글을 퍼블리싱 하는 것을 목표로 하고 있습니다. 

![google-analytics-trend](/img/how-to-organize-tech-blog/google-analytics-trend.png)*쏘카의 기술 블로그 WAU는 꾸준히 증가하고 있습니다.*

이런 정량적인 메트릭 외에도 각종 개발 커뮤니티에서 쏘카 블로그가 꾸준히 거론되는 것을 또 하나의 정성적인 성과로 생각하고 있습니다. 
예를 들면 데이터 엔지니어링을 주제로 하는 글(예: [Airflow 운영기](https://tech.socarcorp.kr/data/2022/11/09/advanced-airflow-for-databiz.html) )이 데이터 엔지니어 오픈 카톡 방에서 긍정적으로 언급될 때 뿌듯함을 느끼기도 하고, 회사에 대한 이미지가 좋아지는 것을 느낄 수 있었습니다.


글 피드백과 전략을 치열하게 고민하여 성과가 좋았던 글들은 특별히 기억에 남기도 합니다. 
예를 들면 2019년에 작성된 [쏘카의 디자인 시스템 글](https://tech.socarcorp.kr/design/2020/06/23/socar-design-system-01.html)이나 2022년에 작성된 [쏘카 예약 테트리스 글](https://tech.socarcorp.kr/data/2022/06/10/reservation-tetris.html) 등은 당시 비슷한 실제 사례를 소개한 글이 많지 않을 때 좋은 글감을 선택하여 성과를 낸 사례입니다.

디자인 시스템의 경우 당시 기술 블로그에 디자이너 직무분들의 글이 많이 없었고, 회사 내부에서는 디자인 시스템이 이제 막 시작되던 시기였습니다.
많은 회사에서 디자인 시스템과 관련하여 비슷한 시도를 하고 시행착오를 하고 있을 것 같아 디자인팀을 설득해 글을 작성하였는데, 그 결과 약 3년이 지났지만 여전히 기술 블로그에서 꾸준히 상위 조회 수를 기록하고 있습니다.

예약 테트리스 글의 경우도 실제 쏘카 서비스에 사용되는 최적화 모델링을 상세하게 소개하여, 배포 당일 기술 블로그 역사상 최고 일간 페이지 뷰 수를 기록했습니다.


## 6. 운영하며 느낀 어려움

위에 작성한 내용은 기술 블로그를 운영하는 과정에 대한 내용입니다. 기술 블로그를 운영하면서 어려움이 없었나요?라고 질문하면 없다고 말하긴 어렵습니다. 그 이유는 기술 블로그에 운영적인 요소와 사람을 동기부여하는 요소 등이 모두 포함되기 때문입니다. 하지만 이것들도 개선할 방법이 없는 것은 아니기에 저희가 진행한 경험을 공유드리려고 합니다



### 6.1. 반복적인 피드백

기술 블로그를 운영하는 과정에서 제일 많이 소요되는 부분은 피드백입니다. 글을 꼼꼼히 읽으면서 의견을 드려야 하기에 많은 시간이 소요되는 업무입니다. 업무를 조금 더 구체화하면 "글의 큰 맥락을 이해하는 과정", "글의 논리적 흐름이 잘 연결되었는지 확인하는 과정", "맞춤법 검사" 등이 있습니다. 이 부분 중 앞의 두 가지는 당장 자동화를 하기엔 어렵다고 판단했고, 단순 반복 작업인 맞춤법 검사를 자동화하는 방법을 고민했습니다.

이런 상황엔 문제를 정의하면 해결 방법이 보이는 경우가 존재합니다. 이 당시 문제를 정의해 보면 다음과 같습니다.


- 문제 상황 : 글 기고자분이 작성한 글을 복사 붙여넣기해서 맞춤법 검사를 한다. 그 후 맞춤법 검사 수정 제안을 한다. 이 과정이 오래 걸린다

이 문제 상황을 해결하기 위해 자동화하는 방법을 고민했습니다. 자동화하는 방법은 여러 가지 방법이 나왔으나 저희가 선택한 방법은 Github Action을 통해 맞춤법 검사를 진행하고 Suggestion을 달아주는 것입니다. Github를 기반으로 업무가 진행되기에 Pull Request가 오는 경우 Github Action을 실행시킬 수 있다 생각했습니다. PR이 형성되는 시점에 진행할 것인가에 대한 고민도 했으나, 글을 처음 제출할 때 맞춤법 검사를 해도 결국 마지막에 글을 올리기 전에 맞춤법 검사를 해야 했습니다. 따라서, PR이 올라오는 시점이 아닌 우리가 실행하고 싶은 시점에 실행하기로 했습니다. 

따라서 Issue Comments가 생성될 때 Github Action이 실행되는 ChatOps 방법을 적용하기로 했습니다. 쏘카 기술 블로그 Github의 PR 코멘트에 `/맞춤법` 입력하면 맞춤법 검사가 실행됩니다. 

맞춤법 자동화를 위한 Github Action의 `yaml` 파일은 다음과 같습니다.


{% raw %}
```yaml
name: Check korean grammar on pull request comment

on:
  issue_comment:
    types:
      - created

jobs:
  grammar_check:
    if: ${{ github.event.issue.pull_request && github.event.comment.body == '/맞춤법'}}
    runs-on: ubuntu-latest
    steps:
    - name: "맞춤법 검사"
      uses: heumsi/korean-grammar-action@v0.2.8
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```
{% endraw %}


`/맞춤법`을 입력하면 다음과 같이 Github Action이 순차적으로 맞춤법 검사를 해줍니다.(과정에서 코멘트가 많게는 50개 정도 달리기도 합니다)


![spell-check-results](/img/how-to-organize-tech-blog/spell-check-results.png)*Github Action이 진행해 주는 맞춤법 검사*





이렇게 작성된 내용을 하나씩 보면서 맞춤법을 적용할지 말지 결정합니다. 때론 맞춤법 검사가 잘못된 결과를 제시하는 경우도 있기 때문에, 마지막에 한번 검수를 진행했습니다. 이 방식을 통해 맞춤법 검사에 소요되는 시간을 비약적으로 줄일 수 있었습니다. 이렇게 아낀 시간을 글감 리뷰, 글 피드백에 더 활용했습니다



### 6.2. 동기부여

기술 블로그를 운영하는 분들이 모이면 종종 하는 이야기가 "어떻게 해야 사람들이 글을 작성하도록 동기부여를 할 수 있을까요?"입니다. 이때 동기는 내재적 동기와 외재적 동기로 나눌 수 있습니다.

- 내재적 동기 : 구성원이 가지고 있는 흥미, 호기심, 자기만족, 성취감에서 비롯되는 동기
- 외재적 동기 : 과제의 해결이 가져다줄 보상이나 벌에서 비롯되는 동기

두 개의 동기를 적절히 활용하는 것이 필요합니다. 다만, 외재적 동기는 제약이 존재하고 계속 유지되기 어렵습니다. 이에 내재적 동기를 높일 수 있는 방법을 고민했습니다. 

블로그 글 작성은 회사가 아닌 개인 블로그여도 어려워하는 분들이 많습니다. 

따라서 기술 블로그에 글을 업로드하기 위해 **"글"을 새로 써야 한다는 생각이 들면 부담이 될 수 있기 때문에, 사내에서 이미 만든 콘텐츠를 "재가공"해서 글 작성을 유도하면 상대적으로 수월합니다.** 

사내에서 만든 콘텐츠를 찾기 위해 다음 방법들을 사용하고 있습니다.

* 사내 위키, 컨플루언스, 노션 등을 확인
* 동료분들에게 최근에 진행한 업무나 들었던 내용 중 좋았던 게 있는지 질문
* 커피챗, 조직장분들과 이야기, 최근 입사자분에게 질문하기 등 

이런 과정을 통해 글감이 될만한 내용들을 선정하고, 글 기고자 분과 글감 방향성에 대해서 충분한 대화를 나눕니다.

이때 글감이 인상 깊은 이유를 구체적으로 피드백하기도 하고, 글감의 희소성과 바이럴 가능성을 공유하기도 하며 (예: 구글 검색 결과 비슷한 주제로 작성된 글이 없는 결과) 작성한 내용을 더 매력적으로 보이게 할 수 있는 여러 재구성 방법을 제시하기도 합니다. 

위 과정이 잘 진행되기 위해선 회사의 문화도 중요합니다. 기록 문화나 스터디 문화가 잘 정착되어 있으면 좋은 소재를 발굴할 가능성이 높아집니다. 

쏘카는 문화적으로 구성원의 성장을 적극적으로 지원하고 있으며, 많은 팀에서 스터디를 진행하고 있습니다. 그리고 조직의 리더이신 CTO님과 이런 문화를 어떻게 만들고 유지시킬 수 있을지에 대해서 꾸준히 대화도 나누었습니다.

또한 전반적인 기술 블로그 기고 경험을 개선하기 위해 글 기고자분들과 대화 및 설문조사를 통하여 어떤 점이 좋았는지, 어떤 점이 어려웠는지 확인합니다.  긍정적인 경험은 점은 지속적으로 좋게 유지할 수 있도록 하고 어려웠던 점은 개선할 수 있도록 고민하고 있습니다.

![survey](/img/how-to-organize-tech-blog/survey.png)*기술 블로그 만족도 설문조사*

예를 들어 기술 블로그 설문 조사 의견 중 "글 마지막에 기고자가 스스로를 PR 할 수 있는 기회가 있으면 동기부여가 될 것 같다"라는 피드백이 있었고, 이 피드백을 반영하여 희망하는 사람에 한 해 글 말미에 작가 카드를 추가할 수 있도록 했습니다.
구체적으로는 `authors.yaml`에 다음과 같이 작가 정보를 추가하면 작가 카드가 추가되도록 `post.html` 코드를 수정하였습니다. (자세한 디자인은 이 글의 마지막에서 확인할 수 있습니다.)

```yaml
dini:
    name: 디니
    image: "/assets/authors/dini.jpeg"
    position: 데이터 엔지니어
    introduction: 쏘카에서 가격/면책 시스템 서버 개발과 테크 블로그 운영을 맡고 있습니다. 커피챗 환영합니다. :)
    links:
        blog: "https://diana-lab.tistory.com/"
        linkedin: "https://www.linkedin.com/in/hyejinyoon/"
```

{% raw %}
```html
{% for page_author in page_authors %}
  {% assign author = site.data.authors[page_author] %}
  {% if author.introduction %}
  <div class="author-card">
	<div class="card mb-3">
	  <div class="row no-gutters">
		<div class="col-md-4">
		  <img src="{{ author.image }}" class="card-img" alt="프로필사진">
		</div>
		<div class="col-md-8">
		  <div class="card-body">
			<p class="card-title">
			  <a href="/authors/#{{ author.name }}">{{ author.name }}</a>
			  <small class="text-muted"> {{ author.position }}</small>
			</p>
			<p class="card-text">{{ author.introduction }} </p>
			<p class="card-text">
			  {% for link in author.links %}
			  <a href="{{ link[1] }}"><img src="/assets/icon/{{ link[0] }}.png" class="link-img" alt="{{ link[0] }}"></a>
			  {% endfor %}
			</p>
		  </div>
		</div>
	  </div>
	</div>
  {% endif %}
{% endfor %}
```
{% endraw %}

### 6.3. 글 작성의 어려움

위와 같은 설문 결과에서 나온 글 작성 과정의 어려움에는 대표적으로 다음이 있었습니다. 

1. 구조화된 형태로 글을 작성하는 것이 어렵다.
2. Github으로 글을 작성하는 과정이 어렵다.

첫 번째로 글 투고 경험이 많이 없는 분의 경우 글의 목차와 구조를 어떻게 구성해야 할지가 큰 고민일 수 있습니다. 
이때 구조화된 형태로 글을 작성하는 게 어렵다면 템플릿을 사용해서 글을 작성할 수 있도록 도움을 줄 수 있습니다. 
예를 들어 다음과 같이 템플릿을 제시하여 글을 구조화할 수 있습니다.

- 문제 정의
- 해결 과정 및 해결 방법
- 시행착오
- 결론
- 글을 보는 사람에게 제공하는 TIP, 노하우

실제로 해외 기술 블로그를 보면 모든 글의 목차가 비슷한 형태인 경우도 있습니다. (참고 :[우버 기술 블로그](https://www.uber.com/en-KR/blog/seoul/engineering/))
다만 어떤 글을 작성하냐에 따라 글의 형태가 달라질 수 있기 때문에 이런 템플릿은 권장 정도만 하고 있습니다. 구체적으로는 노션 페이지에 글쓰기 가이드를 만들어 처음 글 작성 시 참고할 수 있도록 하고 있습니다.

![writing-guide](/img/how-to-organize-tech-blog/writing-guide.png)*기술 블로그 글쓰기 가이드*

장기적으로는 글 주제에 따라 여러 Cookiecutter를 제공하여 Markdown Template 형태로 사용할 수 있도록 하는 방법을 고민하고 있습니다.

두 번째로는 일반적인 블로그 플랫폼(티스토리, 네이버 블로그)가 아닌 Github과 Markdown을 통해서 블로그 글을 작성하는 것이 낯설다는 피드백이 있었습니다. 

이 경우는 Jekyll + Github Pages 형태의 블로그를 사용하고 있기 때문에 발생하는 문제인데, 여러 대안을 고려할 수 있습니다. 기존 URL과 SEO를 유지하면서 티스토리나 Medium, Brunch 같은 다른 플랫폼으로 옮기는 방법도 있고 [Contentful](https://www.contentful.com/) 같은 서비스를 사용하는 방법도 존재합니다. 

Contentful을 사용할 경우 Jekyll 을 통한 블로그 운영을 유지하면서 글 기고자가 훨씬 편리한 UX를 통해 글을 작성할 수 있다는 장점이 있습니다. (이 경우, 서비스 사용을 위해 추가적인 비용이 발생합니다)


## 7. 정리하며

지금까지 기술 블로그를 운영했던 3~4년을 정리했습니다. 내용을 정리하면 다음과 같습니다

- 글 작성 프로세스 구축
- 글 작성의 어려움을 인지하고 이미 사내에 만들어진 자료를 재구성해서 글을 작성하도록 제안
- 자동화 진행하기(Github Action 등)

그 외에도 [ChatGPT](https://chat.openai.com/)를 이용하여 블로그 운영 혹은 글 피드백을 자동화할 수 있는 부분도 분명히 있다고 생각합니다. 예를 들면 블로그 글 내용을 ChatGPT에 전달하고 글에 대한 요약이나 간단한 피드백을 요청할 수 있습니다(과거에는 웹 URL을 제공하고 해당 글을 피드백 요청을 할 수 있었는데 현재는 이런 외부 URL 접근을 막아놓아 이용이 어렵습니다). 아직 실제로 적용해 보지는 않았고 Use Case가 없지만 한번 시도해 보셔도 좋을 것 같습니다. 

개인 블로그가 아닌 회사의 기술 블로그를 운영하는 분에게 도움이 되었길 바라며 글을 마무리하겠습니다. 읽어주셔서 감사합니다.

기술 블로그를 함께 운영해 준 하디, 최초에 기술 블로그를 만들어주신 도마 감사합니다.

 > 추천 자료 : [44bits의 좋은 기술 블로그를 만들어 나가기 위한 8가지 제언](https://www.44bits.io/ko/post/8-suggestions-for-tech-programming-blog)


