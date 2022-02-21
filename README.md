# socar-inc.github.io

[SOCAR Tech Blog](https://tech.socarcorp.kr/) 를 위한 Github Repository입니다.

<br>

## 1. 블로그의 목적
- 외부에 쏘카의 기술력을 알립니다.
- 글을 작성해 쏘팸들의 개인 역량 증진을 증진합니다.

<br>


### 1.1. 글을 쓰면 무엇이 좋나요? 
- 생각을 정리하는 습관을 기를 수 있어요
- 글을 작성하는 동안 더 많은 지식을 쌓을 수 있어요
- 특정 내용을 다시 떠올릴 때 활용할 수 있어요
- 개인 브랜딩을 할 수 있어요

<br>


### 1.2. 어떤 사람이 글을 쓸 수 있나요?
- `개발자`, `기획자`, `디자이너`, `QA`, `데이터 분석가` 등 CTO 산하의 모든 분들이 글을 작성할 수 있습니다
- 개발 블로그라고 개발자만 글을 작성할 수 있다는 생각보다, `기획자 관점에서 개발자와 협업하기`, `디자이너가 데이터 분석을 만날 경우`, `효율적으로 QA하는 방법` 등 다양한 직군들이 글을 작성하면 좋을 것 같습니다
- 고민되시면 언제든 카일(@socar-kyle)과 하디(@socar-hardy)를 찾아주세요

<br>

### 1.3. 어떤 글을 써야 할까요?
- 회사 업무하면서 배운 내용, 에러 디버깅 등 다양한 내용을 작성할 수 있습니다
	- 단, 회사의 기밀이 있는 경우엔 팀장님/본부장님 확인 후 올려주세요! (애매하다 생각되면 여쭤보시는 것이 좋습니다)
- 프로젝트 글 (예시 : 새 버전을 출시하며 익힌 점)
- 회고 글 (예시 : 신입 데이터 분석가 1년 회고)
- 행사 후기 (예시 : O'Reilly Strata Newyork 2019 참여 후기)
- 사내 스터디 후기 (예시 : 기획자와 개발자가 같이 진행한 UX 스터디 후기)

> 글쓰기에 대해 전반적인 경험담이 궁금하시면 카일이 발표한 [개발자를 위한 (블로그) 글쓰기 intro](https://www.slideshare.net/zzsza/intro-102870757)를 참고하시면 좋을 것 같습니다

<br>

### 1.4. 자주 하시는 고민 

- 저는 글을 잘 못쓰는 것 같아요
	- 괜찮습니다. 처음부터 글을 잘 못쓰실 수 있고, 글을 점점 많이 작성하다보면 글쓰는 실력이 개선됩니다. 이런 걱정하시는 분들을 위해 글을 발행하기 전에 `모든 글을 리뷰할 예정`입니다
	- 글 리뷰는 Github Pull Request를 활용할 예정이며, 주로 피드백은 해당 [Slideshare](https://www.slideshare.net/zzsza/ss-137831892#39)에서 나오는 내용들 위주로 드릴 예정입니다
- 어떤 도구로 글을 써야하나요?
	- 이 블로그는 Jekyll로 되어있고, Markdown 또는 HTML으로 글을 작성할 수 있습니다
	- HTML 퍼블리싱이 가능하신 분들은 HTML로 작성하셔도 됩니다
	- Markdown을 사용할 경우 Mac에선 [MacDown](https://macdown.uranusjr.com/), Window에선 [Typora](https://typora.io/)가 가볍게 사용하기 좋습니다. 도구일 뿐이니 편하신 도구가 있으면 해당 도구를 사용하시면 됩니다


<br>

## 2. 글쓰기 방법

### 2.1. GitHub Repo Fork 하고 Clone 받기

- 이 [GitHub Repository](https://github.com/socar-inc/socar-inc.github.io)를 본인의 계정으로 Fork 합니다.

  ![레포지토리 Fork 하기](/img/readme/fork.png)

- Fork 한 본인 계정의 Repository의 `Settings` - `General` 페이지에 들어가 `Repository name`을 `{본인 Github 계정}.github.io`로 변경합니다.

  ![Fork한 레포지토리 이름 변경하기](/img/readme/rename.png)

- Fork 받은 본인의 Repository를 clone 합니다.

  ```bash
  $ git clone git@github.com:{본인 Github 계정}/{본인 Github 계정}.github.io.git
  ```

<br>

### 2.2. 글 작성하기

- `_posts` 디렉토리에 post를 생성 합니다.
  - 파일명은 `YYYY-MM-DD-<post-name>.md`로 합니다.
  - 본문을 html로 작성할 경우 `.html`도 사용 할 수 있습니다.

<br>

### 2.3. 글 확인하기

내가 쓰고 있는 글이 실제 블로그에서는 어떤 형태인지 직접 확인하고 싶을 때, 다음처럼 2가지 방법으로 확인하실 수 있습니다.

- 로컬에 직접 [Jekyll](https://jekyllrb.com/docs/)를 설치해서 확인하는 방법
- 로컬에 직접 Jekyll을 설치하지 않고 확인하는 방법

#### 2.3.1. 로컬에 직접 [Jekyll](https://jekyllrb.com/docs/)를 설치해서 확인하는 방법

- 먼저, Jekyll과 필요한 의존성을 설치합니다.

  * MacOS

    ```shell
    /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)" #homebrew 설치
    brew install ruby # ruby 설치
    gem install --user-install bundler jekyll # jekyll , bundler 설치
    ```


  * Window

    * https://rubyinstaller.org/ 에서 ruby 를 다운받습니다.

    ```shell
    gem update
    gem install jekyll bundler # jekyll, bundler 설치
    ```


- 이제 `bundle exec jekyll serve`  명령어로 로컬 ( http://127.0.0.1:4000/) 에서 확인할 수 있습니다.

> Jekyll 설치 시, 전체적인 글 작성 및 확인 흐름은 다음과 같습니다. 
>
> ```shell
> # 레포지토리 클론 및 브랜치 생성
> $ git clone https://github.com/socar-{닉네임}/socar-{닉네암}.github.io.git
> $ cd socar-{닉네임}.github.io
> $ git checkout -b feature/{닉네임}-{주제}
> 
> # 글 작성 중 로컬에서 확인
> $ bundle exec jekyll serve 
> 
> # 수정 사항을 레포지토리에 반영
> $ git add .
> $ git commit -m '초안을 작성합니다'
> $ git push 
> ```


#### 2.3.2. 로컬에 직접 Jekyll을 설치하지 않고 확인하는 방법

- fork 한 레포지토리에서 글을 작성하고 있는 feature branch 를 master branch에 merge 하여 확인합니다.  
- master branch에 변경사항이 push 되면 1~2분 이내에 수정사항이 반영 됩니다. socar-{닉네임}.github.io 에서 확인할 수 있습니다.

> Jekyll 설치 시, 전체적인 글 작성 및 확인 흐름은 다음과 같습니다. 
>
> ```bash
> # 레포지토리 클론 및 브랜치 생성
> $ git clone https://github.com/socar-{닉네임}/socar-{닉네암}.github.io.git
> $ cd socar-{닉네임}.github.io
> $ git checkout -b feature/{닉네임}-{주제}
> 
> # 글 작성 후 레포지토리 반영
> $ git add .
> $ git commit -m '초안을 작성합니다'
> $ git push
> 
> # feature 브랜치를 master로 머지하여 확인
> $ git checkout master
> $ git merge feature/{닉네임}-{주제} 
> $ git push
> ```

<br>

### 2.4. PR 요청하기

* post가 완성되면 리뷰를 받기 위해 원본 [Repository](https://github.com/socar-inc/socar-inc.github.io)로 PR을 생성합니다. 

* PR(Pull Request)를 생성할때 from / to는 아래와 같이 설정 합니다.

  ```
  - base repository: socar-inc/socar-inc.github.io
  	- base: master
  - head repository: {본인 GitHub 계정}/{본인 GitHub 계정}.github.io
    - compare: master
  ```

* PR 생성시 default 리뷰어는 카일 (@socar-kyle), 하디(@socar-hardy)입니다. 추가적으로 팀에서 리뷰어를 지정해주세요 :) 

> 현재 Merge 조건 및 권한은 다음과 같습니다.
>
> * Merge 조건으로 Approve 2명 이상 필요
> * `Squash and merge` 를 원칙으로 함
> * master push 권한은 하디와 카일에게만 있음 
> * resolve 되지 않은 conversation 이 있으면 merge 불가

<br>

## 3. 글 작성 팁

### 3.1. 이건 지켜주세요! 

#### 3.1.1. 머리말 설정하기

* 글 맨위 부분에 다음과 같이 머리말을 설정해주세요. 내용은 마음대로 바꾸셔도 됩니다. 

```
---
layout: post
title:  "쏘카 신입 데이터 엔지니어 디니의 4개월 회고"
subtitle: 입사 지원부터 팀 온보딩, 실무 투입까지
date: 2021-12-28 17:00:00 +0900
category: data
background : "/assets/images/onboarding-bg.jpg"
author: dini
comments: true
tags:
    - data
    - data-engineering
---
```

> 참고로, date 를 현재 시간보다 늦게 설정하면 Jekyll 등을 이용하여 블로그를 띄워도 글이 보이지 않습니다. (로컬에서 확인할 때는 하루 정도 전 시간으로 하는 설정해두는 것 추천 - 글이 작성 완료된 후에는 공개 시점에 맞도록 수정합니다. )

#### 3.1.2. PR 리뷰 전에 먼저 master에 머지하기 <a name="master-merge-before-pr"></a>

* PR 리뷰 전에, 최종 버전의 글을 fork 한 레포지토리에서 master branch 로 merge 해서 리뷰어들이 작성한 글을 웹에서 확인할 수 있게 해주세요. 

```shell
# feature 브랜치에서 글 작성 후 레포지토리 반영
$ git add .
$ git commit -m '글을 완성합니다'
$ git push

# feature 브랜치를 master로 머지하여 확인
$ git checkout master
$ git merge feature/{닉네임}-{주제} 
$ git push
```

#### 3.1.3. 기타 컨벤션 정리

* fork 한 repository 명은 socar-{닉네임}.github.io 로 합니다.
* 브랜치 명은 feature/{닉네임}-{주제} 로 합니다.
* post 명은 `YYYY-MM-DD-<post-name>.md` 으로 합니다.

<br>

### 3.2. 이렇게 하면 더 좋아요!

#### 3.2.1. `subtitle` 설정 하기
- subtitle를 설정하지 않으면 글 목록에 본문의 앞 몇글자를 잘라서 보여 주게 됩니다.
- subtitle를 사용하면 글 목록에 노출 되기 원하는 문구를 별도로 지정 가능합니다.

#### 3.2.2. 상단 배너 이미지 넣기
- `https://unsplash.com`에서 적절히 고릅니다. (저작권 고민안하고 써도 되는 이미지 사이트 입니다.)
- 다운받은 후 `/assets/images/`에 넣고 `background`에 경로를 적습니다.
- (선택) Download시 나오는 `Photo by *** on Unsplash` 문구를 Post 상단에 넣습니다.
- 예시 입니다. href의 링크는 적절히 수정해서 사용합니다.
  
    ```html
    <div class="photo-copyright">
    Photo by <a href="https://unsplash.com/@zhenhu2424?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Zhen Hu</a> on <a href="https://unsplash.com/search/photos/lock?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>
    </div>
    ```

#### 3.2.3. [mermaid.js](https://mermaidjs.github.io/#/) 사용하기
- Text를 이용해 다이어그램을 그릴 수 있습니다. 
- Flowchart / Sequence diagrams을 그릴 수 있습니다.
- 아래 예시 처럼 `<div class="mermaid">`를 본문에 사용하시면 됩니다.

    ```
    <div class="mermaid">
    graph TD
        Start --> Stop
    </div>
    ```
    
#### 3.2.4. markdown 중간에 html을 섞어서 사용해도 됩니다
- 위 mermaid.js 예시처럼 markdown 내부에 html을 사용해도 상관 없습니다
- jekyll에서 html 태그는 변환하지 않고 그대로 그 위치에 출력해 줍니다

#### 3.2.5. 글 구조
- 글의 구조는 취향껏 작성하시면 되지만, 어려우신 분들은 아래의 흐름 정도를 가지셔도 좋을 것 같습니다
		
	```
	1. 이 글에서 다루는 문제는 무엇인가요? 
	2. 이 글을 작성하는 이유는? 이 글을 보는 독자가 얻을 수 있는 내용은?
	3. 문제에 대한 해결 방안
	4. 해결 후, 남은 이슈는 있나요?
	5. 또 어떤 것을 고민해보면 좋을까요?
	6. 참고 자료(Reference)
	```
