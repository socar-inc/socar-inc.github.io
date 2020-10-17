# socar-inc.github.io
- SOCAR Tech Blog


## **깔끔한 commit history 관리를 위해 `Squash Merge`를 사용해 주세요.**

<br><br><br>


## 블로그의 목적
- 외부에 쏘카의 기술력을 알리기 위함
- 글을 작성해 쏘팸들의 개인 역량 증진


### 블로그에 글을 작성하면 좋은 점
- 생각을 정리하는 습관을 기를 수 있어요
- 글을 작성하는 동안 더 많은 지식을 쌓을 수 있어요
- 특정 내용을 다시 떠오를 때 활용할 수 있어요
- 브랜딩


### 어떤 사람이 글을 쓸 수 있나요?
- `개발자`, `기획자`, `디자이너`, `QA`, `데이터 분석가` 등 CTO 산하의 모든 분들이 글을 작성할 수 있습니다
	- 개발 블로그라고 개발자만 글을 작성할 수 있다는 생각보다, `기획자 관점에서 개발자와 협업하기`, `디자이너가 데이터 분석을 만날 경우`, `효율적으로 QA하는 방법` 등 다양한 직군들이 글을 작성하면 좋을 것 같습니다
	- 고민되시면 언제든 kyle을 찾아주세요

### 어떤 글을 써야 할까요?
- 회사 업무하면서 배운 내용, 에러 디버깅 등 다양한 내용을 작성할 수 있습니다
	- 단, 회사의 기밀이 있는 경우엔 팀장님/본부장님 확인 후 올려주세요(애매하다 생각되면 여쭤보시는 것이 좋습니다)
- 프로젝트 글
	- 예시 : 새 버전을 출시하며 익힌 점
- 회고 글
	- 예시 : 신입 데이터 분석가 1년 회고
- 행사 후기
	- 예시 : O'Reilly Strata Newyork 2019 참여 후기
- 사내 스터디 후기
	- 예시 : 기획자와 개발자가 같이 진행한 UX 스터디 후기
- 글쓰기에 대해 전반적인 경험담이 궁금하시면 카일이 발표한 [개발자를 위한 (블로그) 글쓰기 intro](https://www.slideshare.net/zzsza/intro-102870757)를 참고하시면 좋을 것 같습니다


### 자주 하시는 고민 
- 저는 글을 잘 못쓰는 것 같아요
	- 괜찮습니다. 처음부터 글을 잘 못쓰실 수 있고, 글을 점점 많이 작성하다보면 글쓰는 실력이 개선됩니다. 이런 걱정하시는 분들을 위해 글을 발행하기 전에 `모든 글을 리뷰할 예정`입니다
		- 글 리뷰는 Github Pull Request를 활용할 예정이며, 주로 피드백은 해당 [Slideshare](https://www.slideshare.net/zzsza/ss-137831892#39)에서 나오는 내용들 위주로 드릴 예정입니다
- 어떤 도구로 글을 써야하나요?
	- 이 블로그는 Jekyll로 되어있고, Markdown 또는 HTML으로 글을 작성할 수 있습니다
	- HTML 퍼블리싱이 가능하신 분들은 HTML로 작성하셔도 됩니다
	- Markdown을 사용할 경우 Mac에선 [MacDown](https://macdown.uranusjr.com/), Window에선 [Typora](https://typora.io/)가 가볍게 사용하기 좋습니다. 도구일 뿐이니 편하신 도구가 있으면 해당 도구를 사용하시면 됩니다
	- 참고로 제목은 `YEAR-MONTH-DAY-title.MARKUP`으로 저장하시면 됩니다. 참고 : [포스트 파일 생성하기](https://jekyllrb-ko.github.io/docs/posts/#포스트-파일-생성하기)


<br><br><br>

## 글쓰기 방법
#### 1. 로컬에 [Jekyll](https://jekyllrb.com/docs/)를 설치 하는 경우
- 글 수정 후 확인은 로컬에 설치된 Jekyll을 이용해서 바로 확인 할 수 있습니다
	- 이 [Repository](https://github.com/socar-inc/socar-inc.github.io)를 바로 clone 해서 사용.
	- `feature/<id>-*` 브랜치 생성 후 작업 -> PR 생성 -> `Squash Merge`
	- PR 생성

	```
	git clone https://github.com/socar-inc/socar-inc.github.io.git
	cd socar-inc.github.io
	git checkout -b feature/<id>-*
	
	# 로컬에서 작성한 글 테스트
	bundle exec jekyll serve
	# 글 작성 후
	git add _posts/YYYY-MM-DD-<post-name>.md
	git commit _posts/YYYY-MM-DD-<post-name>.md
	git push -m "..."
	```

#### 2. Jekyll을 설치 하지 않고 하는 경우
- Jekyll을 로컬에 설치 하지 않은 경우 github의 `master branch에 push`해야 합니다. 이 경우 `작성 중인 글이 사이트에 노출` 되고 `commit history 가 지저분`해지는 문제가 있으므로 아래 방법으로 진행 하시길 추천 드립니다.

- **예시는...`socar-dorma` ID로 `feature/dorma-readme` 브랜치로 작업한 예시입니다.**
  - 이 [Repository](https://github.com/socar-inc/socar-inc.github.io)를 본인의 계정으로 `fork` 합니다.
  - `fork 한 본인 계정의 Repository`의 `Settings`에서 `Repository name`을 수정합니다.
    - `socar-dorma.github.io`로 변경.
    - 변경 하고 잠시 후 (1~2분?) `https://socar-dorma.github.io`로 접속하면 사이트가 열립니다.
  - Fork 받은 본인의 Repository를 clone 합니다.

    ```
    $ git clone git@github.com:socar-dorma/socar-dorma.github.io.git
    ```
  
  - `feature/dorma-readme` 브랜치 생성, checkout, push

    ```
    $ git branch feature/dorma-readme
    $ git checkout feature/dorma-readme
    $ git push -u origin feature/dorma-readme
    ```

  - 이제 clone 받은 소스의 `_posts` 디렉토리에 post를 생성 합니다.
    - 파일명은 `YYYY-MM-DD-<post-name>.md`로 합니다.
    - 본분을 html로 작성할 경우 `.html`도 사용 할 수 있습니다.

  - 수정 한 글을 사이트에서 확인 하려면 master branch에 push 합니다.
    - 원본 [Repository](https://github.com/socar-inc/socar-inc.github.io)로의 PR을 만들기 위해 수정은 생성한 `feature/dorma-readme` branch에서 진행 하고 수정사항 확인을 위해 master branch에 merge 합니다. 

  ```
  $ git add .
  $ git commit -m 'readme 작성중'
  $ git push
  $ git checkout master
  $ git merge feature/dorma-readme
  $ git push
  $ git checkout feature/dorma-readme
  ```

  - master branch에 변경사항이 push 되면 1~2분 이내에 수정사항이 반영 되었다고 메일이 옵니다.
  - post가 완성되면 원본 [Repository](https://github.com/socar-inc/socar-inc.github.io)로 PR을 생성합니다.
    - PR(Pull Request)를 생성할때 from / to는 아래와 같이 설정 합니다.
        - base repository: socar-inc/socar-inc.github.io
            - base: master
        - head repository: socar-dorma/socar-dorma.github.io
            - compare: feature/dorma-readme

  - PR 생성시 default 리뷰어는 kyle, shua입니다. 추가적으로 팀에서 리뷰어를 지정해주세요 :)
  - **PR 생성 후 Merge 시에는 `Squash and merge`를 선택해서 Merge 합니다.**


<br><br><br>

## 글 작성 팁
#### 1. `subtitle` 설정 하기
- subtitle를 설정하지 않으면 글 목록에 본문의 앞 몇글자를 잘라서 보여 주게 됩니다.
- subtitle를 사용하면 글 목록에 노출 되기 원하는 문구를 별도로 지정 가능합니다.

#### 2. 상단 배너 이미지 넣기
- `https://unsplash.com`에서 적절히 고릅니다. (저작권 고민안하고 써도 되는 이미지 사이트 입니다.)
- 다운받은 후 `/assets/images/`에 넣고 `background`에 경로를 적습니다.
- (선택) Download시 나오는 `Photo by *** on Unsplash` 문구를 Post 상단에 넣습니다.
- 예시 입니다. href의 링크는 적절히 수정해서 사용합니다.
    
    ```html
    <div class="photo-copyright">
    Photo by <a href="https://unsplash.com/@zhenhu2424?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Zhen Hu</a> on <a href="https://unsplash.com/search/photos/lock?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>
    </div>
    ```

#### 3. [mermaid.js](https://mermaidjs.github.io/#/) 사용하기
- Text를 이용해 다이어그램을 그릴 수 있습니다. 
- Flowchart / Sequence diagrams을 그릴 수 있습니다.
- 아래 예시 처럼 `<div class="mermaid">`를 본문에 사용하시면 됩니다.

    ```
    <div class="mermaid">
    graph TD
        Start --> Stop
    </div>
    ```
    
#### 4. markdown 중간에 html을 섞어서 사용해도 됩니다
- 위 mermaid.js 예시처럼 markdown 내부에 html을 사용해도 상관 없습니다
- jekyll에서 html 태그는 변환하지 않고 그대로 그 위치에 출력해 줍니다

#### 5. 글 구조
- 글의 구조는 취향껏 작성하시면 되지만, 어려우신 분들은 아래의 흐름 정도를 가지셔도 좋을 것 같습니다
		
	```
	1. 이 글에서 다루는 문제는 무엇인가요? 
	2. 이 글을 작성하는 이유는? 이 글을 보는 독자가 얻을 수 있는 내용은?
	3. 문제에 대한 해결 방안
	4. 해결 후, 남은 이슈는 있나요?
	5. 또 어떤 것을 고민해보면 좋을까요?
	6. 참고 자료(Reference)
	```
