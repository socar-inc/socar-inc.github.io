---
layout: post
title: "쏘카 데이터 그룹 - Airflow on Kubernetes 구축기"
subtitle: "쿠버네티스를 도입하여 에어플로우 운영을 개선한 후기"
date: 2021-03-25 18:00:00 +0900
category: data
background: '/assets/images/parrish-freeman-58QVNWSB6qQ-unsplash.jpg'
author: hardy
comments: true
tags:
    - data
    - data-engineering
---

안녕하세요. 데이터 엔지니어링팀의 하디입니다.  
이번 글에서는 최근에 쿠버네티스 위로 에어플로우를 옮기는 작업을 하며 팀에서 에어플로우 운영을 개선해나간 과정을 소개합니다.  

<br>

## 목차

- 기존의 배포와 운영
  - 배경
  - 배포 형태
  - 운영 형태
  - 문제점
- 새롭게 개선한 배포와 운영
  - 배경
  - 배포 형태 (운영 환경)
  - 배포 형태 (개발 환경)
  - 배포 방법
  - 운영 형태
  - 기타 추가 작업
- 해야할 남은 작업
- 참고한 자료

---

<br>

## 기존의 배포와 운영

<br>

### 배경

기존의 에어플로우는 대략 2년 전 처음으로 배포(1.10.4 버전)되었습니다.  
소수의 데이터 분석가들이 팀 내에 있었고 데이터 엔지니어링팀은 이제 막 만들어질 시기였습니다.  
이 때 에어플로우 배포와 운영에 초점을 둔 사항은 다음과 같았습니다.

- 빠르게 배포할 수 있어야 한다.
- 데이터 분석 팀원이 쉽게 DAG 작성이 가능해야 한다.
- 운영이 쉬워야 한다.

팀 초기 상황에 맞게 전반적으로 "속도"와 "편의성"에 초점을 두었습니다.

<br>

### 배포 형태

배포 형태는 다음과 같았습니다.

![image-20210323133054666](/img/airflow-on-kubernetes/image-20210323133054666.png){: width="100%"}

- GCP의 GCE 인스턴스를 호스팅 서버로 사용합니다.
- 에어플로우의 각 컴포넌트들인 WebServer, Scheduler, Database, Worker, Redis를 각각 도커 컨테이너로 실행합니다.
- 에어플로우 스케쥴러는 Celery Executor를 사용합니다.
- Jupyter Notobook 를 도커 컨테이너로 띄운 뒤 Airflow DAG 폴더에 Volume Mount 합니다.

<br>

### 운영 형태

- DAG 작성자는 Jupyter Notebook 웹을 접속하여 DAG을 작성합니다. 
- Airflow의 Connections에 BigQuery 등의 세팅을 미리 해두어 재사용 가능하게 합니다.

위와 같은 방식은 에어플로우를 가장 간단하게 배포, 운영 일반적인 패턴인거 같습니다.  
그러나 조직과 서비스가 점점 성장하고, 이 형태로 지속해서 해나가다보니 점점 여러 문제점들이 보이기 시작했습니다.

<br>

### 문제점

<br>

#### 1) 늘어나는 DAG들

데이터와 서비스가 커감에 따라 DAG의 개수도 점점 늘어났습니다. DAG의 실행속도가 점점 느려지게 되고, 제 시간에 실행되지 않고 시작 시간이 밀리는 경우도 종종 보게되었습니다. 배치 스케쥴링 주기가 짧은 DAG들에게 이런 현상은 매우 심각한 문제였습니다.  
이 때 마다 할 수 있는 일은 GCE의 머신 유형을 한 단계 업그레이드 (컴퓨팅 리소스를 수직 확장) 하는 방법 뿐이었습니다. 그러나 이렇게 해도 DAG이 실행되는 특정 시간시간 외에는 리소스를 쓰는 일이 거의 없으므로 리소스 사용 효율 측면에서는 매우 비효율적이었습니다. 리소스를 유연하고 효율적으로 사용할 수 있는 방법에 대해 고민이 들기 시작했습니다.

<br>

#### 2) 규칙 없는 제각각의 DAG 코드

여러 사람들에 의해서 작성된 각각의 DAG들은 모양도 제각각이었습니다. 예를 들어 어떤 DAG에는 `on_failure_callback` 값이 있는 반면, 어떤 DAG들은 이 값들이 보이지 않았습니다. 그리고 Jupyter Notebook 환경에서 DAG 코드가 작성되다보니, PEP8과 같은 스타일 컨벤션을 찾아보기 어려웠습니다. 무엇보다 가장 큰 문제는 DAG 코드가 담긴 모듈 이름에 규칙 없어서, 에어플로우 웹에서 문제가 생긴 DAG이 실제로 어떤 파일인지 찾기 쉽지 않았다는 점입니다. 또한 누군가 언제 어떤 코드를 만들고 수정했는지에 대한 히스토리가 없어서 문제가 생겼을 때 원인을 찾기가 어려웠습니다.  
일관된 DAG의 템플릿과 PEP8과 같은 코드 스타일 규칙의 필요성이 느껴졌습니다. 또한 코드 변경을 추적할 수 있도록 Git과 같은 버전관리 툴 도입이 필요성이 느껴지기 시작했습니다.

<br>

#### 3) 운영과 테스트의 혼재

많은 DAG들은 데이터 레이크나 마트를 위한 파이프라인인 경우가 많았고, 이 파이프라인들의 목적지는 주로 GCP BigQuery 였습니다. 대부분의 DAG 작성자들은 자신의 파이프라인이 잘 작동하는지 테스트하기 위해 Bigquery 의 최종 테이블 이름의 끝에 `_test` 를 붙여서 실행하는 경우가 많았습니다. 이렇게 실행 후, 테스트한 데이터셋을 지우지 않는 경우가 많아 빅쿼리에는 `_test` 로 끝나는 테이블들이 여기저기 지저분하게 생기게 되었습니다. 하지만 이것보다 더 큰 문제는 테스트하는 과정에서 실제 운영 중인 테이블에 직접 접근하는 로직이 포함되어 있다는 것입니다. 예를 들어 DAG 작성자가 `BigqueryOperator` 를 사용하여 테스트할 때, 깜빡하고 테이블 이름 끝에  `_test` 를 붙이지 않고 DAG 을 실행하는 경우 실제 운영되는 테이블을 덮어쓴다거나 날려버릴 수 도 있었습니다. 운영환경과 격리되어 안전하게 테스트할 수 있도록 별도의 개발 환경이 필요했습니다.

<br>

## 새롭게 개선한 배포와 운영

<br>

### 배경

2년 사이에 회사 서비스와 데이터 조직은 빠르게 성장했습니다. 이러한 성장에 맞춰 데이터 파이프라인 개수도 늘게 되었고, 에어플로우의 DAG의 개수는 600개를 넘어서게 됐습니다. DAG을 작성하는 사람도 다양해지고, 사용하는 `Operator` 더 다양해졌습니다. 더 많은 리소스가 필요하게 되었고, 더 많은 관리 포인트들이 생기게 되었습니다.

이제 다음과 같은 부분에 초점을 두고 에어플로우를 고도화하게 됩니다.

- DAG이 늘어나도 실행시간의 늘어나거나 지연이 없도록 **리소스 확보 및 유연하게 할당**
- **깔끔하고 일관된 DAG 코드**와 파라미터 값의 표준화
- 운영 환경에 영향을 주지 않고 **테스트 가능한 별도의 환경**



위와 같은 부분을 초점에 두고 다음의 내용들을 결정 및 도입하기로 했습니다.

- 단일 컴퓨팅이 아닌, 쿠버네티스 환경을 도입합니다.
    - 노드 오토 스케일링을 적용하여, 필요에 따라 리소스를 유연하게 확보하고 할당할 수 있습니다.
- 에어플로우 1.10.14로 버전을 업데이트하고, Kubernetes Executor를 사용합니다.
    -  Task 단위로 Pod를 생성하기 때문에 동시에 여러 Task를 빠르게 실행할 수 있습니다.
    - 노드 오토 스케일링이 적용되어 있어 DAG이 늘어나도 리소스로 인한 실행 속도에 문제가 없습니다.
- DAG 코드 작성에 규칙을 줍니다.
    - 별도로 제작한 CLI 툴을 이용하여 DAG 보일러 플레이트 코드를 만듭니다.
    - 코드 포매터로 Black을 사용하고, CI 과정에서 포매팅을 검사합니다.
- DAG 코드를 git과 github로 버전관리 합니다.
    - 누가 언제 어떤 코드를 추가, 변경했는지 추적할 수 있습니다.
    - 덤으로, 코드 리뷰를 통해 보다 나은 DAG 코드를 유지합니다.
- 개발용 GCP 프로젝트에 격리된 테스트용 에어플로우 환경을 만듭니다.
    - DAG 코드가 담긴 github 리포지터리의 브랜치에 따라 동적으로 테스트용 에어플로우를 배포합니다.
    - 개발 환경에서는 운영 Bigquery에 READ 가능하도록 합니다.

<br>

### 배포 형태 (운영 환경)

운영과 개발환경의 배포 형태가 미세하게 다릅니다.  
먼저 운영 환경은 개발 환경과 다른 별도의 GCP 프로젝트 안에서 구성됩니다.  
형태는 다음과 같습니다.

![image-20210323152353144](/img/airflow-on-kubernetes/image-20210323152353144.png){: width="100%"}

- GKE(Google Kuberentes Engine)에 별도의 네임스페이스 위에 에어플로우가 배포됩니다.
- 스케쥴러는 Kubernetes Executor를 사용합니다.
    - 이에 따라 에어플로우 컴포넌트는 Webserver, Scheduler, Database, Worker 만 필요하게 됩니다.
    - 각 컴포넌트는 Pod 단위로 배포됩니다.
    - Worker Pod는 DAG 내 Task 단위로 동적으로 생성되었다가 내려갑니다.

위에서 핵심 컴포넌트들을 하나하나 살펴보겠습니다.

<br>

#### 1) External Database (Cloud SQL) 사용

![image-20210323153112845](/img/airflow-on-kubernetes/image-20210323153112845.png){: width="100%"}

에어플로우 컴포넌트 중 하나인 Database는 Stateful 하므로, Kubernetes 내부에서 관리하는 것보다 외부에서 쓰는게 더 적합하다고 생각했습니다. 따라서 외부 Database 인스턴스인 GCP Cloud SQL을 사용하고 있습니다. GKE 내부에서는 Cloud SQL과 통신하기 위해 Cloud SQL Proxy를 Daemonset 형태로 배포합니다. 에어플로우는 이 Proxy를 거쳐 외부 Database 와 통신하게 됩니다.  
이렇게 하면 에어플로우가 쿠버네티스가 위에서 재배포되어도 데이터는 여전히 남아있게 됩니다.

<br>

#### 2) DAG을 담는 Github Repo 생성 및 sync

![image-20210323153832010](/img/airflow-on-kubernetes/image-20210323153832010.png){: width="100%"}

Github 리포지터리에 DAG들을 저장합니다. 운영 환경에 반영되는 DAG들은 `main` 브랜치에 담습니다.  
Webserver, Scheduler Pod 내부에 Git-sync는 주기적으로 이 Github Repository를 Pull 합니다. Worker의 경우 처음에만 (Initial container) Clone 합니다. Git-sync가 Pull 혹은 Clone 한 리포지토리는 Airlfow DAG 폴더에 마운트되어 있습니다. 따라서 Git-sync 작동에 따라 에어플로우의 DAG 폴더가 주기적으로 업데이트 됩니다.

<br>

#### 3) Remote Logging (Cloud Storage) 사용

![image-20210323154732016](/img/airflow-on-kubernetes/image-20210323154732016.png){: width="100%"}

각 에어플로우 컴포넌트가 Pod 단위로 배포되므로, Pod간 공유할 수 있는 별도의 로그 저장소가 필요합니다.  
이에 대한 대안으로 에어플로우의 `AIRFLOW__CORE__REMOTE_LOGGING` 값을 `True` 로 두어 Remote Logging 기능을 사용합니다.  
워커 팟에서 발생하는 로그 파일들은 모두 GCP Cloud Storage에 쓰고 읽게 됩니다.

<br>

#### 4) Kubernetes Executor 사용

![image-20210323155040109](/img/airflow-on-kubernetes/image-20210323155040109.png){: width="100%"}

Kubernetes Executor를 사용하면 Worker를 Pod 형태로 동적으로 생성하게 됩니다. DAG이 실행될 때 Task 하나 당 하나의 Worker Pod가 배포 및 실행된 후 삭제됩니다. 실행할 DAG이 없는 경우 쿠버네티스 위에 Worker Pod가 존재하지 않습니다. 즉 실행할 테스크가 없을 때는 리소스 사용이 줄었다가, 실행할 테스크가 생기면 필요한 만큼 리소스를 할당하고 사용하게 됩니다. 결과적으로 리소스를 더 유연하고 효율적으로 사용하게 됩니다.

<br>

### 배포 형태 (개발 환경)

개발 환경은 운영 환경과는 별도의 GCP 프로젝트에 구성됩니다.  

![image-20210323160518620](/img/airflow-on-kubernetes/image-20210323160518620.png){: width="100%"}

전반적으로 운영환경에서의 배포와 동일합니다. 다만 DB가 외부 DB가 아닌 내부 DB입니다.  

개발 환경에서의 에어플로우는 항상 배포되어 떠있지 않고, DAG Github 리포지토리에 `feature/` 로 시작하는 브랜치를 생성할 때 동적으로 생성되었다가 브랜치 삭제 시에 내려갑니다. 즉 개인이 DAG 작업할 때에만 테스트하기 위한 용도로 배포되는 방식입니다. 테스트 이후에는 테스트에 사용된 DB 역시 삭제해야 하기 때문에 외부 DB가 아닌 클러스터 내부 DB로 배포하였습니다.

<br>

### 배포 방법

쿠버네티스 클러스터에 앱 배포는 [ArgoCD](https://argoproj.github.io/argo-cd/)를 사용하고 있습니다. ArgoCD는 Git-ops형태로 쿠버네티스에 앱을 배포할 수 있는 CD 툴입니다.  
쿠버네티스에 배포할 helm 차트를 다음처럼 별도의 Github Repository에 보관합니다.  
(차트는 [커뮤니티 버전의 에어플로우 차트 7.7.0 버전](https://github.com/helm/charts/tree/master/stable/airflow)을 활용하여 커스터마이징 했습니다.)

![image-20210325162545630](/img/airflow-on-kubernetes/image-20210325162545630.png){: width="100%"}

이후 ArgoCD 웹서버에서 이 Github Repository와 연결을 맺도록 세팅합니다. 그리고 App을 배포합니다.  
(ArgoCD로 앱을 배포하는 방법은 [커피 고래님 블로그 글](https://coffeewhale.com/kubernetes/gitops/argocd/2020/02/10/gitops-argocd/)을 참고하시면 좋습니다.)

![image-20210325162455581](/img/airflow-on-kubernetes/image-20210325162455581.png){: width="100%"}

<br>

### 운영 형태

<br>

#### 1) DAG 작성 프로세스

이제 이전처럼 Jupyter Notebook 으로 DAG을 작성하지 않습니다. 
각자 로컬의 IDE와 Git branch를 통해 DAG을 작성하고 Github Repository에 Push 하여 에어플로우에 DAG을 올립니다.  

DAG을 작성할 때에는 엔지니어링팀에서 만든 보일러플레이트 툴을 통해 만듭니다. 보일러 플레이트 툴에는 DAG 파일 이름이나  `dag_id` 를 어떤 규칙으로 생성할 지에 대한 로직이 담겨있습니다. 이렇게 함으로써 전체적으로 DAG의 기본 모양새를 일관성있게 관리할 수 있고, DAG 작성하는 사람 입장에서도 고민없이 쉽게 작성을 시작할 수 있습니다.  
DAG 작성 및 테스트 후에는 `main` 브랜치로 PR을 보냅니다. 다른 팀원들의 리뷰를 받을 후 본격적으로 공용 브랜치인 `main` 에 자신이 만든 DAG이 합류하게 됩니다.

좀 더 구체적인 프로세스를 기술하면 다음과 같습니다.

- DAG 작성자는 먼저 DAG을 저장하고 있는 GIthub Repository 를 Clone 받습니다.
- 본인이 작업할 브랜치 (`feature/*`)를 만들어 [DAG 보일러플레이트 CLI 툴]()로 DAG을 작성합니다.
    - 예를 들면 `feature/hardy` 와 같이 브랜치 이름을 정합니다.
- Github remote origin 으로 push 합니다.
    - 예를 들면 `git push origin feature/hardy` 입니다.
- Github 에 새로운 `feature/*` 브랜치가 생성됨과 동시에 본인이 작업할 수 있는 에어플로우가 할당됩니다.
    - 이 에어플로우는 개발 환경에 배포됩니다.
    - 이 에어플로우는 본인이 작업하는 Github 브랜치와 동기화 됩니다.
    - 위의 예시의 경우 `airflow.socar-data/feature/hardy` 로 접속가능 합니다.
- 본인이 할당받은 에어플로우에서 이런저런 테스트를 마친 후 `main` 브랜치로 PR을 보냅니다.
    - CI 파이프라인에서 코드가 Black 으로 포매팅되어있는지 확인합니다.
    - 여러 팀원들에게 리뷰를 받은 후 최종적으로 Squash & Merge 합니다.
    - Branch 가 삭제되면 할당받은 에어플로우도 삭제됩니다.

![image-20210323162247055](/img/airflow-on-kubernetes/image-20210323162247055.png){: width="100%"}

<br>

> *** 2) DAG 보일러플레이트 CLI 툴**
>
> DAG 작성자가 DAG 초기 작성을 어떻게 할지 모르거나, 당장 뼈대가 되는 코드가 필요한 경우 비슷한 다른 DAG 코드를 복사하여 사용하는 경우가 많았습니다. 그러나 다른 비슷한 DAG을 찾는 것도 번거롭고, 무엇보다 다른 DAG 코드의 형태에 의존하다보니 DAG의 기본 형태가 일관되지 않았습니다.
>
> 이 때문에 DAG의 기본 코드 형태를 규칙적으로 만들어주는 별도의 CLI 툴을 만들게 되었습니다.  
> 다음처럼 이 툴을 사용할 수 있습니다. (실행 결과로 DAG 파일을 만들어줍니다.)
>
> ![image-20210323175842093](/img/airflow-on-kubernetes/image-20210323175842093.png){: width="100%"}

<br>

#### 3) 작업 에어플로우를 위한 CI/CD

위와 같이 작업자의 `feature/*` 브랜치에 해당하는 에어플로우를 할당하기 위한 CI/CD 파이프라인은 다음과 같습니다.

![image-20210323162546775](/img/airflow-on-kubernetes/image-20210323162546775.png){: width="100%"}

- Github Repository Webhook에서 Branch creation, deletion 이벤트 발생 시, CI/CD 파이프라인이 동작하는 트리거 설정합니다.
- 저희 팀은 쉽게 CI/CD 파이프라인을 구축할 수 있는 [BuddyWorks](https://app.buddy.works/)를 사용하고 있습니다.
- Branch creation 시, Buddyworks 내 파이프라인에서는 다음의 일들을 처리합니다.
    - 클러스터와 통신할 수 있는 Bastion Host에 ssh 접속합니다.
    - Bastion Host에서 ArgoCD Client를 통해 클러스터에 배포되어있는 ArgoCD Server에 로그인합니다.
    - ArgoCD Client로 현재 HEAD branch 와 연동된 Airflow 앱을 배포합니다.
- Branch deletion 시, 같은 방법으로 위에서 생성된 Airflow 앱을 삭제합니다.

CI/CD 파이프라인 동작 모니터링은 Slack 특정 채널을 통해 알람을 받습니다.

![image-20210325115107953](/img/airflow-on-kubernetes/image-20210325115107953.png){: width="100%"}

<br>

#### 4) 리소스 모니터링

Kubernetes에서 동적으로 에어플로우를 운영하다보니, 리소스 모니터링이 또한 중요하게 되었습니다.  
아직 모니터링을 고도화하지는 않았지만 KubeLens와 Grafana로 모니터링을 하고 있습니다.

![kubelens](/img/airflow-on-kubernetes/kubelens.png){: width="100%"}

![image-20210323164836014](/img/airflow-on-kubernetes/image-20210323164836014.png){: width="100%"}

<br>

### 기타 추가 작업

에어플로우 일부를 좀 더 커스터마이징하여 사용하기 위해 Airflow 1.10.14 버전을 Clone 받아 일부 코드를 아래 내용처럼 수정했습니다.

<br>

#### 1) Pod 이름 생성 로직 수정

에어플로우 1.10.14 버전을 그대로 사용하면 Worker Pod 이름이 다음처럼 생성됩니다.

```bash
# dag_id가 replication_database_table_one 이고
# task_id가 db_replicaion_task 인 경우

replicationdatabasetableonedbreplicaiontask-hashvalue
```

Pod 이름이 읽기 어렵습니다. (Pod 이름이 생성되는 코드는 Airflow Github 코드에서 [이 부분](https://github.com/apache/airflow/blob/c743b95a02ba1ec04013635a56ad042ce98823d2/airflow/executors/kubernetes_executor.py#L541)과 [이 부분](https://github.com/apache/airflow/blob/c743b95a02ba1ec04013635a56ad042ce98823d2/airflow/executors/kubernetes_executor.py#L508)을 보면 알 수 있습니다.)  
Pod 이름을 `{dag_id}.{task_id}-hashvalue` 와 같은 형태로 바꾸어 좀 더 보기 쉽게 만들고 싶었습니다. 예를 들면 다음과 같이 말입니다.

```bash
replication-database-table-one.db-replicaion-task-hashvalue
```

이를 위해 에어플로우 코드에서 Pod 이름을 생성하는 로직 일부를 다음과 같이 수정했습니다. (주석 처리한 부분이 수정한 기존 코드입니다.)

```python
# airflow/executors/kubernetes_executor.py 

@staticmethod
def _create_pod_id(dag_id, task_id):
    safe_dag_id = AirflowKubernetesScheduler._strip_unsafe_kubernetes_special_chars(dag_id)
    safe_task_id = AirflowKubernetesScheduler._strip_unsafe_kubernetes_special_chars(task_id)
    # return safe_dag_id + safe_task_id
    return safe_dag_id + "." + safe_task_id

...

def _strip_unsafe_kubernetes_special_chars(string):
    ...
    # return "".join(ch.lower() if ch.isalnum() else ch for ind, ch in enumerate(string))
    return string.lower().replace("_", "-")
```

<br>

#### 2) Operator WeightRule  변경

에어플로우 오퍼레이터의 기본 `weight_rule` 은 `WeightRule.DOWNSTREAM` 입니다.  
이 때문에 DAG 내에서 뒤쪽에 실행되는 Task들은 낮은 실행 우선순위를 갖게되고, DAG이 늘어날수록 전반적인 DAG 실행시간이 길어지는 경향이 있습니다. (이와 관련한 자세한 내용은 [Line engineering 블로그 글](https://engineering.linecorp.com/ko/blog/data-engineering-with-airflow-k8s-2/)에서 보실 수 있습니다.)  
Task 가 어느 위치에 있건 같은 우선순위를 주기 위해 `weight_rule` 의 기본 값을 `WeightRule.ABSOLUTE` 로 수정하였습니다.

```python
# airflow/models/baseoperator.py 

class BaseOperator:
    def __init__(
        ...
        # weight_rule=WeightRule.DOWNSTREAM,  # type: str
        weight_rule=WeightRule.ABSOLUTE,  # type: str
        ...
    )
```

<br>

#### 3) Scheduler/Webserver 성능 관련 Configuration 값 수정

DAG 수가 늘어남에 따라 스케쥴러가 전체 DAG을 파싱하는 속도는 점점 느려지는 문제가 있습니다. 따라서 스케쥴러의 성능을 올려주어야 하는데, CPU 리소스를 더 늘리는 방법 말고도 에어플로우의 Configuration 값을 다음과 같이 수정함으로써 성능을 일부 향상시킬 수 있었습니다.

```yaml
AIRFLOW__SCHEDULER__PARSING_PROCESSES: 8  #  기본 값은 2입니다.
```

물론 `parsing_process` 의 수를 늘리는 것에 비례해서 CPU 리소스는 더 잡아먹습니다. (따라서 쿠버네티스에서 컨테이너 리소스도 더 늘려서 줘야합니다.) 하지만 이런 트레이드오프를 통하여 DAG 실행 속도를 늘리는게 더 좋다고 판단했습니다.

또한 웹 서버 접속 시 페이지 로드 지연을 줄이기 위해 다음 Configration 값들도 수정했습니다.

```yaml
AIRFLOW__WEBSERVER__PAGE_SIZE: 50  # 기본 값은 100입니다.
AIRFLOW__WEBSERVER__WORKER_REFRESH_INTERVAL: 1800   # 기본 값은 30입니다. 
AIRFLOW__WEBSERVER__WEB_SERVER_WORKER_TIMEOUT: 300  # 기본 값은 120입니다.
```

<br>

#### 4) 빌드 과정에 써드파티 라이브러리 추가

에어플로우에서 제공하지않는 써드파티 라이브러리를 사용하는 경우, 수동으로 설치해야합니다. 예를 들면 `beautifulsoup4` 가 이런 써드파티 라이브러리의 대표적인 예입니다.

문제는 Webserver, Scheduler, Worker 모든 Pod에 설치해야하는데, 매번 필요할 때마다 설치하는게 번거롭다는 것입니다. 그래서 이렇게 자주 사용할  써드파티 라이브러리는 에어플로우 프로젝트 소스 내에 `extra-requirements.txt` 를 만들어 이 안에 명시하고 에어플로우 이미지를 빌드할 때  이 `extra-requirements.txt` 를 설치하도록 `Dockerfile` 에 커맨드를 추가하였습니다.

```python
# extra-requirements.txt 

beautifulsoup4==4.9.3
lxml==4.6.2
```

```dockerfile
# Dockerfile

...
COPY ./extra-requirements.txt /extra-requirements.txt
RUN pip install -r extra-requirements.txt
...
```


<br>


## 해야할 남은 작업

에어플로우를 쿠버네티스로 옮기고 쿠버네티스 환경에 알맞게 사용하도록 하는 큰 흐름은 마무리지었습니다.  
그러나 아직 운영 고도화를 위한 몇 가지 작업들이 남아있습니다.

<br>

### Airflow 2.x 으로 업그레이드

작년 말 [에어플로우 2.0.0 이 발표](https://airflow.apache.org/blog/airflow-two-point-oh-is-here/)되었습니다. 메이저 버전이 바뀐만큼 기존 Feature 수정 및 새로운 Feature가 추가되었습니다. 그 중 특히 눈에 띄는 것은 [Scheduler의 로직의 퍼포먼스 향상](https://airflow.apache.org/blog/airflow-two-point-oh-is-here/)과 [HA (High Availity) 지원](https://airflow.apache.org/blog/airflow-two-point-oh-is-here/)입니다. 앞으로 하나의 통합 에어플로우 환경을 유지하면서 더 늘어날 DAG을 고려하면 2.x 버전으로 버전 을 올리는게 매우 좋아보입니다. 다만 아직은 쿠버네티스에 에어플로우를 배포한지 얼마 안된 운영 초기이고, 기존의 에어플로우에서 DAG들을 하나씩 리팩토링하며 옮기고 있기 때문에, 당장 버전을 올리지는 않을 계획입니다.  

기존 DAG을 옮기는 작업을 마치고 추후 운영에 노하우가 좀 더 생기면 에어플로우 버전 올리는 작업을 진행해볼 예정입니다. 특히 에어플로우의 공식 Helm Chart가 공개되면 개발 환경에서 테스트한 뒤 도입해볼 계획입니다. 

<br>


### Grafana 대시보드 고도화

현재 Grafana 대시보드는 Pod의 cpu, memory 리소스 정도만 보여주고 있습니다.  
앞으로는 DAG 별 평균 처리 시간, 평균 지연 시간, 리소스 사용량 등을 대시보드에서 모니터링할 수 있도록 고도화할 계획이 있습니다. 당장은 문제가 없도록 리소스를 넉넉히 주었지만, 모니터링 후에 리소스 최적량을 찾아 리소스 다이어트도 해야합니다. 결과적으로 성장하는 조직에 문제가 없도록 쿠버네티스 클러스터 자체와 에어플로우 운영에 대해 계속해서 공부해나가는 것이 다음의 목표가 되겠습니다.

<br>

## 참고한 자료

- [Line engineering 블로그 - Kubernetes를 이용한 효율적인 데이터 엔지니어링(Airflow on Kubernetes VS Airflow Kubernetes Executor) 시리즈](https://engineering.linecorp.com/ko/blog/data-engineering-with-airflow-k8s-1/)
- [커피 고래님 블로그 - GitOps와 ArgoCD](https://coffeewhale.com/kubernetes/gitops/argocd/2020/02/10/gitops-argocd/)
- [Swalloow님 블로그 - Airflow On Kubernetes 시리즈](https://swalloow.github.io/airflow-on-kubernetes-1/)
- [Subicura님 발표자료 - 쿠버네티스를 이용한 기능 브랜치별 테스트 서버 만들기 (GitOps CI/CD)](https://www.slideshare.net/subicura/gitops-cicd-156402754)

<br>