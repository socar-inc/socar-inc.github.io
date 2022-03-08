---
layout: post
title: 데이터 디스커버리 플랫폼 도입기 - 2편. GKE에 Datahub 구축하기
subtitle: feat. 메타데이터 플랫폼이 실제로 배포되기까지 
date: 2022-03-07 17:00:00 +0900
category: data
background : "/assets/images/cloud-bg.jpg"
author: dini
comments: true
tags:

    - data
        - data-engineering
---



안녕하세요, 데이터 플랫폼 팀의 디니입니다. 

이번 글은 '쏘카의 데이터 디스커버리 플랫폼 도입기' 3부작 중 2편입니다. [1편 : 데이터 디스커버리 플랫폼 도입기 - 데이터 디스커버리란?]() 에서는 데이터 디스커버리의 개념과, 쏘카가 데이터 디스커버리 플랫폼으로 Datahub 를 선택하게 된 의사결정 과정을 소개했습니다. 

2편에서는 실제로 Datahub 를 GKE 환경에 배포한 과정과, 데이터 디스커버리 플랫폼에 필요한 추가 기능들을 어떻게 구현하였는지에 대해 소개하려고 합니다. 다음과 같은 분들이 읽으시면 도움이 되리라 생각합니다. 

- 메타데이터 플랫폼 도입에 관심이 있는 개발자
- 클라우드 환경에 메타데이터 플랫폼 배포하는 과정에 관심이 있는 사람
- 쏘카 데이터플랫폼팀 업무에 관심이 있는 사람

목차는 이렇습니다.

1. [Datahub on GKE 배포 과정](#datahub-on-gke)

   1.1 GKE 배포 

   1.2 CloudSQL DB migration 

   1.3 Keycloak 인증

2. [메타데이터 주입 과정](#metadata-ingestion) 

   2.1 ingestion 정책 결정

   2.2 ingestion 과정 자동화 (with Airflow)

   2.3 메타데이터 추출 과정의 권한 축소 

3. [마무리](#wrap-up)



이 글에서는 이러한 과정을 통해 데이터 허브를 구축합니다

* 첫번째로는, 플랫폼을 사내 클라우드 환경에 안정적으로 배포합니다. 

* 두번째로는, 플랫폼에 메타데이터를 주입하는 파이프라인을 자동화합니다.

먼저, Datahub 를 어떻게 사내 클라우드 환경에 안정적으로 배포했는지 알아보겠습니다.

>  Datahub 는 오픈소스 기반으로 매우 빠르게 업데이트 되고 있습니다. 해당 배포는 6개월 전에 이루어진 것으로, 현재 Datahub 배포 및 metadata ingestion 과정과는 다소 차이가 있을 수 있습니다. 이 점 양해 부탁 드립니다. 




## 1. Datahub on GKE 배포 과정 <a name="datahub-on-gke"></a>

### 1.1 GKE 배포 

쏘카 데이터엔지니어링 그룹의 쿠버네티스 환경은 GCP(Google Cloud Platform)의 GKE(Google Kuberentes Engine) 를 사용하고 있습니다. 또한 대부분의 어플리케이션을 [Helm](https://helm.sh/) Chart 를 이용하여 클러스터에 배포하고 있습니다. Datahub 역시 공식 [Helm Chart](https://github.com/acryldata/datahub-helm) 를 제공하고 있으며, 총 2 벌의 차트로 구성되어 있습니다. 구성은 다음과 같습니다.

* datahub : Datahub 어플리케이션에 필요한 요소들 설치 (frontend, gms 등)
* prerequisites : Datahub 에 필요한 사전 요소들을 설치 (MySQL, Kafka, ElasticSearch, neo4j 등)

![datahub-helm-chart-tree](/img/data-discovery-platform-02/datahub-helm-chart-tree.png) *Datahub 차트 구조*

공식 Helm Chart 의 ingress 를 쏘카의 환경에 맞게 수정한 뒤 배포하였습니다. 또한 원활한 테스트를 위해 개발 클러스터, 운영 클러스터에 각각 배포하였습니다.  

![datahub-pods](/img/data-discovery-platform-02/datahub-pods.png) *Datahub 최초 배포시 pod 상태*

### 1.2 CloudSQL DB migration

Datahub 는 자체 db (storage) 로 mysql pod 을 사용합니다. 물론 PVC(PersistentVolumeClaim) 이 붙어있긴 했지만, 앞으로 Datahub 어플리케이션 상에서 쌓일 데이터가 점점 늘어날 것이며 데이터의 내용 또한 중요하기 때문에, 앞으로의 확장성과 만에 하나라도 있을 유실 가능성을 방지하는 방향으로 아키텍쳐를 고민했습니다. 결국에는 mysql pod 대신 외부 데이터베이스로 GCP CloudSQL instance 를 연결하기로 결정했습니다.

구체적으로는 다음 과정으로 진행했습니다. 

* CloudSQL db (혹은 새로운 instance) 생성
* (Optional) 사용자 생성
* datahub 가 CloudSQL 가리키게 하기

기존 datahub 의 Helm Chart 는 sql host 로 mysql pod 을 가리키고 있습니다. 위에서 만든 cloudsql db 를 가리키게 하기 위해서 Helm Chart 를 다음과 같이 수정합니다.

```yaml
# charts/datahub/values.yaml
sql:
    datasource:
      host: <cloudsql_host_address>:<port>
      url: "jdbc:mysql://<cloudsql_host_address>:<port>/<cloudsql_db_name>?verifyServerCertificate=false&useSSL=true&useUnicode=yes&characterEncoding=UTF-8&enabledTLSProtocols=TLSv1.2"
      hostForMysqlClient: <cloudsql_host_address>
      port: <port>
      driver: "com.mysql.jdbc.Driver"
      username: <username>
      password:
        secretRef: <별도로 생성한 secret 이름>
        secretKey: <별도로 생성한 secret 키>
```

```yaml
# charts/datahub/subcharts/datahub-gms/values.yaml

sql:
  datasource:
    host: <cloudsql_host_address>:<port>
    url: "jdbc:mysql://<cloudsql_host_address>:<port>/datahub?verifyServerCertificate=false&useSSL=true"
    driver: "com.mysql.jdbc.Driver"
    username: <username>
    password:
      secretRef: <별도로 생성한 secret 이름>
      secretKey: <별도로 생성한 secret 키>
```

실제로는, 민감한 정보들을 Helm Chart 에 직접 명시하지 않고 다음 처럼 별도의 secret 으로 생성하여 참조하였습니다.

```yaml
  sql:
    datasource:
      host:
        secretRef: mysql-secrets
        secretKey: mysql-host
      url:
        secretRef: mysql-secrets
        secretKey: mysql-url
      hostForMysqlClient:
        secretRef: mysql-secrets
        secretKey: mysql-hostForMysqlClient
      port: "3306
      driver: "com.mysql.jdbc.Driver"
      username:
        secretRef: mysql-secrets
        secretKey: mysql-username
      password:
        secretRef: mysql-secrets
        secretKey: mysql-password
```

 최종적으로 정상 작동을 테스트합니다. 예를 들어, datahub ui 상에서 데이터를 수정한 뒤 해당 CloudSQL db에서 동일한 데이터가 업데이트 되는지 확인합니다.

![create-test-policy](/img/data-discovery-platform-02/datahub-create-policy.png)*test_policy 라는 정책을 생성해보았습니다.*

![check-data](/img/data-discovery-platform-02/datahub-check-data.png)*CloudSQL db에서 동일한 데이터가 확인됩니다.*



### 1.3 Keycloak 인증

 Datahub 이 배포되고 나면, 인증된 사용자만 어플리케이션에 접속되어야 합니다. Datahub는 okta, keycloak 등 여러 SSO 를 지원하는데, 쏘카에서 이미 keycloak 을 이용하고 있기 때문에 keycloak 을 사용하기로 결정했습니다. 

 Datahub 에 keycloak 로그인을 적용하는 방법은 간단했습니다. Helm Chart의 `values.yaml`파일에서 frontend 부분에 몇 줄의 설정만 넣어주면 가능했습니다. 

```yaml
datahub-frontend:
  ...
  extraEnvs:
    - name: AUTH_OIDC_ENABLED
      value: "true"
    - name: AUTH_OIDC_CLIENT_ID
      value: your-client-id
    - name: AUTH_OIDC_CLIENT_SECRET
      value: your-client-secret
    - name: AUTH_OIDC_DISCOVERY_URI
      value: your-provider-discovery-url  
    - name: AUTH_OIDC_BASE_URL
      value: your-datahub-url  
```



또한 다음과 같은 다양한 설정을 쉽게 정의할 수 있었습니다. 

```yaml
# User and groups provisioning

# OIDC 로그인시, datahub 상에 유저 없으면 자동 생성 여부
AUTH_OIDC_JIT_PROVISIONING_ENABLED=true 

# OIDC 로그인시, datahub 상에 유저가 이미 존재해야 로그인 성공하는지 여부
AUTH_OIDC_PRE_PROVISIONING_REQUIRED=false

# OIDC 의 그룹 정보를 datahub 에 연동
AUTH_OIDC_EXTRACT_GROUPS_ENABLED=true 
AUTH_OIDC_GROUPS_CLAIM=<your-groups-claim-name>
```



## 2. 메타데이터 주입 과정  <a name="metadata-ingestion"></a>

이렇게 Datahub을 클라우드 상에 안정적으로 구축했습니다. 하지만 아직 데이터 디스커버리 플랫폼으로서의 기능을 하지는 못합니다. 데이터 소스에서 실제로 메타데이터를 가져와서 플랫폼에서 보여줘야 하고, 이렇게 메타데이터를 가져오는 과정(=Metadata Ingestion)이 자동화 되어야 합니다. 

### 2.1 메타데이터 주입 정책 결정

먼저 "어떤 데이터 소스"에서 메타데이터를 "얼마나 자주" 가져올건지 결정해야 합니다.

쏘카에서는 주요 데이터소스로 MySQL Aurora (운영) 와 BigQuery (분석) 을 사용하고 있습니다. 하지만 이 데이터소스의 모든 테이블을 가져올 필요는 없었습니다. 예를 들면 DB에 따라서 개인 정보 관련 민감한 데이터들도 있고, 분석 DB 쪽에는 굳이 전사에 공유될 필요는 없는 임시 테이블들도 다수 존재했습니다. 따라서 각 데이터 소스 별 DB의 목록을 사전에 정하고, 해당 DB 의 메타데이터를 주입하기로 했습니다.

참고로, 다음과 같이 table 혹은 db 의 이름을 regex pattern으로 감지하여 선택적 메타데이터 주입이 가능합니다. (물론 데이터 소스마다 방법이 약간 다를 수 있습니다 - 예시는 빅쿼리의 경우입니다.)

```yaml
source:
  type: bigquery
  config:
    schema_pattern:
      allow: # 허용하는 dataset 패턴
      - "my_dataset"
    table_pattern:
      allow: # 허용하는 table 패턴
      - "^my_project.my_dataset.my_good_table\$"
      deny: # 허용하지 않는 table 패턴
      - "^my_project.my_dataset.my_bad_table\$"
    include_views: false
```

그러면 얼마나 자주 가져와야 할까요? 매일매일 필요에 따라 테이블이 생겨났다가 사라지기도 하고, 테이블의 컬럼이 추가되거나 변경되는 일도 있을 것입니다. 하지만 이런 변화들을 꼭 실시간으로 봐야 할 필요는 없다고 생각했습니다. 하루에 한번 정도 업데이트한다면, 들이는 리소스도 효율화하고 사내 데이터 현황을 파악하는 데 충분하다고 결정을 내렸습니다. 

### 2.2 메타데이터 주입 과정 자동화 (with Airflow)

이렇게 하루에 한번 메타데이터 주입을 결정하고 난 뒤, 메타데이터 주입을 어떻게 자동화했는지 알아보겠습니다. 

하루에 한번 batch 성 주입이라면 airflow DAG 로 간단하게 구현할 수 있었습니다. 데이터소스에서 메타데이터를 주입하는 task 를 만들고, 하루 한번만 돌려주면 됐습니다. 그런데 이 작업에는 `datahub` 패키지를 설치해야 하는 의존성이 필요합니다. 

데이터플랫폼 팀에서는 이런 경우 airflow 에 직접 의존성을 설치하지 않고, 필요한 의존성을 담은 docker image 를 실행하는 K8sPodOperator 를 만들어 해결하고 있습니다. 이렇게 하면 DAG 가 아무리 많아도 의존성의 충돌을 방지할 수 있습니다. 

```dockerfile
# datahub-ingestion 이미지를 이용합니다. 
FROM linkedin/datahub-ingestion:v0.8.20 

# 미리 정의한 recipe 파일을 복사해 가져옵니다. 
COPY datahub-ingestion-bigquery /datahub-ingestion-bigquery 

# datahub CLI 를 이용하여 recipe 를 실행합니다. 
CMD ["ingest", "-c", "/datahub-ingestion-bigquery/recipe_bigquery.yaml"] 
```



### 2.3 메타데이터 추출 과정의 권한 축소

#### 어떻게 하면 최소한의 권한으로 메타데이터를 추출할 수 있을까?

 Datahub는 메타데이터를 끌어오는 모든 대상 db에 select 권한을 허용해야 메타데이터 추출이 가능하도록 만들어져 있습니다. 예를 들면 MySQL DB에 3000개의 db가 있다고 가정할때, Datahub의 서비스 계정은 3000개의 db에 대해 모두 권한이 있어야 하는 것입니다. 

 하지만 이 권한을 단독 솔루션에 부여하기에는 보안상 너무 무겁다고 판단했습니다. 그래서 어떻게 하면 최소한의 db에 접근하면서 같은 기능을 구현할 수 있을지가 큰 고민거리였습니다.

#### Information Schema 에서 직접 뽑아내보자

사실 대부분 DB의 메타데이터는 information_schema 라는 파일에 별도로 저장이 되어 있습니다. 여기만 접근해서 가져와도 될것 같은데 꼭 모든 DB에 권한이 필요할지를 고민하던 와중에, 당시 데이터엔지니어링팀 팀장 (이시고 지금은 그룹장이신) 토마스가 아이디어를 주셨습니다.

 Datahub에는 데이터 소스의 메타데이터를 특정 형태의 json file 로 변환하여 저장하는 기능이 있습니다. 또한 같은 형식의 json file 을 기반으로 메타데이터를 Datahub 플랫폼에 주입하는 것도 가능했습니다. 그리고 해당 파일 형식을 확인해본 결과, information_schema 에서 대부분(사실 모두) 가져올 수 있는 정보였습니다. 

 그러면 information_schema 에서 정보를 가져와서 file 형식을 맞춰 만들어주는 기능을 개발하고, 그 file 기반으로 metadata ingestion 하면 되지 않을까? 하는 생각이 들었습니다. 그래서 앞부분은 python script 로 개발하고, file 을 기반으로 메타데이터를 주입하는 부분은 기존 Datahub 프레임워크를 그대로 이용했습니다. 

```dockerfile
# 파이썬 이미지를 이용합니다. 
FROM python:3.8 

COPY datahub-ingestion-mysql /datahub-ingestion-mysql 

WORKDIR /datahub-ingestion-mysql/src

USER root

# datahub을 포함한 필요한 의존성을 설치합니다. 
RUN pip install --no-cache-dir mysql-connector-python==8.0.27 && pip install --no-cache-dir --upgrade acryl-datahub==0.8.20 

# information_schema 에서 메타데이터를 추출하는 python script 를 실행하고, datahub CLI 로 datahub 플랫폼에 주입합니다.
CMD python main.py ; datahub ingest -c recipe_mysql_prod.yaml 
```



```python
# python script 중, information_schema 에서 column info 를 뽑아내는 부분

def get_column_info_query(pattern_clause) -> str:
    column_info_query = f"""
        SELECT concat(table_schema,'.',table_name) as schemaName,
        column_name, column_comment, data_type, column_type, is_nullable, column_key
        FROM information_schema.columns
        {pattern_clause}
        ORDER BY schemaName, ordinal_position;
        """
    return column_info_query
 
```



#### 최종 테스트

이렇게 기능을 구현하고, 프로젝트에 같이 참여하시고 계시는 인프라팀(현재 CloudDB팀)의 제이든과 직접 테스트를 해보았습니다. 좌충우돌 끝에 결과적으로는 information_schema 에만 권한이 있는 계정으로, 원하는 DB의 모든 메타데이터를 가져와서 Datahub에 주입하는 데에 성공했습니다. 

```
스크린샷 (제이든 허락맡기)
```

이 기능 개발로 쏘카의 DB에 접근하는 Datahub 계정의 권한이 크게 축소되어, 내부 정보보호 규칙에 맞게 보안을 개선할 수 있습니다. 그리고 Datahub 최종 도입 결정에 긍정적인 영향을 미쳤습니다. 



## 3. 마무리  <a name="wrap-up"></a>

이러한 여러 과정 끝에, Datahub 가 쏘카에 도입될 준비를 마쳤습니다. 현재는 이렇게 테스트 배포를 마치고, 사내 공개를 위한 준비 작업을 하고 있습니다. 이자리를 빌어 Datahub 도입에 힘써주신 모든 분들과 부족한 저를 도와주신 분들에게 감사드립니다. 

 입사하고 맡은 첫 프로젝트였는데, 쏘카 데이터 플랫폼팀의 전반적인 인프라와 배포 흐름에 대해 알수 있는 좋은 기회였습니다. 여담으로, 구현하면서 슬랙에서 질답을 너무 많이 한 나머지 커뮤니티 기여자로 Datahub 개발팀과 원격 인터뷰를 하는 기회도 얻었습니다 (!) 

 ![datahub-swag-all](/img/data-discovery-platform-02/datahub-swag-all.jpeg) *인터뷰 기념품으로 준 Datahub 기념품 세트*

![datahub-swag-thanks](/img/data-discovery-platform-02/datahub-swag-thanks.jpeg)*기념품과 함께 온 감사 메세지*

다음 편에서는 실제로 데이터 디스커버리 플랫폼이 도입된 후의 운영 방식과 효과에 대해서 살펴보겠습니다.

긴 글 읽어주셔서 감사합니다. 

> 데이터 플랫폼팀이 하는 업무가 궁금하시다면 [데이터 엔지니어링 팀이 하는 일](https://tech.socarcorp.kr/data/2021/03/24/what-socar-data-engineering-team-does.html)과 [쏘카 데이터 엔지니어 채용공고](https://www.notion.so/socarcorp/d458b6b77a2243fb873d1ac800c321f7?p=1e895c6f8d6c49d0962d9c3af3e37f81)를, 데이터 플랫폼 팀의 신입 온보딩 과정이 궁금하시다면 [쏘카 신입 데이터 엔지니어 디니의 4개월 회고](https://tech.socarcorp.kr/data/2021/12/28/data-engineering-team-onboarding.html)를 보시기를 추천드립니다. 

