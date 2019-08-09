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
예시는...`socar-dorma` ID로 `feature/dorma-readme` 브랜치로 작업한 예시입니다.

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
    - 나중에 PR을 만들기 위해 

  ```
  $ git add .
  $ git commit -m 'readme 작성중'
  $ git push
  $ git checkout master
  $ git merge feature/dorma-readme
  $ git push
  ```