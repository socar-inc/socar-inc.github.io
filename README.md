# socar.github.io

## **깔끔한 commit history 관리를 위해 `Squash Merge`를 사용해 주세요.**

## 글쓰기 방법

#### 1. 로컬에 [Jekyll](https://jekyllrb.com/docs/)를 설치 하는 경우
글 수정 후 확인은 로컬에 설치된 Jekyll을 이용해서 바로 확인 할 수 있습니다.
  - 이 [Repository](https://github.com/socar-inc/socar-inc.github.io)를 바로 clone 해서 사용.
  - `feature/<id>-*` 형태의 브랜치 생성 후 작업 -> PR 생성 -> `Squash Merge`
  - PR 생성

#### 2. Jekyll을 설치 하지 않고 하는 경우
Jekyll을 로컬에 설치 하지 않은 경우 github의 `master branch에 push`해야 합니다. 이 경우 `작성 중인 글이 사이트에 노출` 되고 `commit history 가 지저분`해지는 문제가 있으므로 아래 방법으로 진행 하시길 추천 드립니다.
  - 이 [Repository](https://github.com/socar-inc/socar-inc.github.io)를 본인의 계정으로 `fork` 합니다.
  - `fork 한 본인 계정의 Repository`의 `Settings`에서 `Repository name`을 수정합니다.
    - `<본인ID>.github.io`로 변경.
    - 변경 하고 잠시 후 (1~2분?) `https://<본인ID>.github.io`로 접속하면 사이트가 열립니다.
  - Fork 받은 본인의 Repository를 clone 합니다.
  - `feature/<id>-*`