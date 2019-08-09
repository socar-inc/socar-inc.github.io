# socar.github.io

## **깔끔한 commit history 관리를 위해 `Squash Merge`를 사용해 주세요.**

## 글쓰기 방법

#### 1. 로컬에 [Jekyll](https://jekyllrb.com/docs/)를 설치 하는 경우
글 수정 후 확인은 로컬에 설치된 Jekyll을 이용해서 바로 확인 할 수 있습니다.
  - 이 [Repository](https://github.com/socar-inc/socar-inc.github.io)를 바로 clone 해서 사용.
  - `feature/<id>-*` 브랜치 생성 후 작업 -> PR 생성 -> `Squash Merge`
  - PR 생성

#### 2. Jekyll을 설치 하지 않고 하는 경우
Jekyll을 로컬에 설치 하지 않은 경우 github의 `master branch에 push`해야 합니다. 이 경우 `작성 중인 글이 사이트에 노출` 되고 `commit history 가 지저분`해지는 문제가 있으므로 아래 방법으로 진행 하시길 추천 드립니다.

**예시는...`socar-dorma` ID로 `feature/dorma-readme` 브랜치로 작업한 예시입니다.**

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

  - **PR 생성 후 Merge 시에는 `Squash and merge`를 선택해서 Merge 합니다.**


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
  * Text를 이용해 다이어그램을 그릴 수 있습니다. 
  * Flowchart / Sequence diagrams을 그릴 수 있습니다.
  * 아래 예시 처럼 `<div class="mermaid">`를 본문에 사용하시면 됩니다.

    ```
    <div class="mermaid">
    graph TD
        Start --> Stop
    </div>
    ```