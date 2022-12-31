---
layout: post
title: "빠르고 정확하게, 신사업 FMS 데이터 파이프라인 구축기 - 2"
subtitle: 빠르고 정확하게, 신사업 FMS 데이터 파이프라인 구축기
date: 2022-12-11 09:00:00 +0900
category: data
background: "/img/advanced-airflow-for-databiz/background.jpg"
author: grab
comments: true
tags:
    - data
    - data engineering
    - iot streaming
    - data platform
    - kafka connect
    - aws
---

안녕하세요. 데이터 플랫폼 팀의 그랩입니다.

이번 글은 '빠르고 정확하게, 신사업 FMS 데이터 파이프라인 구축기'의 2편으로 안정적으로 파이프라인을 운영하기 위해 했던 노력들을 소개해 드리려고 합니다.

글을 읽으시면서 궁금한 점들이 있다면 편하게 질문 남겨주시면, 확인 후 답변드리겠습니다.

목차는 아래와 같습니다.  
....

## 5. 견고한 파이프라인을 위한 E2E 테스트 환경 구축하기

본 장에서는 1편에서 소개드린 주요 컴포넌트(Kafka Connect, Lambda 등)을 안정적으로 운영하기 위해 적용한 E2E 테스트에 대해 소개드리겠습니다.

### 데이터 파이프라인에 테스트가 필요한 이유

왜 데이터 파이프라인에서 테스트가 필요할까요? 만약 데이터 파이프라인의 구성요소가 SaaS나 클라우드 서비스로만 구성된다면 테스트가 크게 필요하지 않을 수 있습니다. 하지만 1부에서 다룬 것처럼 Kafka Sink Connector, Lambda 함수 같이 요구사항에 맞는 소프트웨어를 직접 개발해야 한다면 어떨까요? 소프트웨어의 신뢰성을 높이고 잦은 변경과 배포를 가능하도록 하게 테스트를 작성하는 것이 중요할 수 있습니다.

데이터 파이프라인에서 특정 컴포넌트가 제대로 동작하지 않는다면 Downstream 컴포넌트는 당연히 동작을 하지 않고 제대로 된 데이터 변형/적재가 힘들어 집니다. 즉 각 컴포넌트가 SPoF(Single Point of Failure)가 되기 쉽기 때문에 충분히 신뢰할 수 있는 환경을 구성해주는 것이 중요합니다.  
그리고 데이터 파이프라인의 특성상 Input과 Output이 명확합니다. 따라서 테스트 작성이나 결과를 검증하는 과정이 쉬운 편입니다.

데이터 파이프라인에서 많이 언급되는 Data Quality 테스트는 7장에서 더 자세하게 다루도록 하겠습니다.

### E2E 테스트란

소프트웨어를 테스트 할 때 많이 나누는 분류 기준으로 Unit Test(단위 테스트), Integration Test(통합 테스트), E2E Test(종단간 테스트)가 있습니다.

**유닛 테스트**

-   유닛(Unit)이라는 말 그대로, 가장 작은 단위의 테스트입니다. 단일 기능을 가지는 함수, 클래스의 메서드가 잘 작동하는지 테스트할 때 많이 사용됩니다.
-   가장 간단하고, 직관적이며, 빠르게 실행과 결과를 볼 수 있는 테스트입니다.

**통합 테스트**

-   통합(Integration)이라는 말 그대로, 여러 요소를 통합한 테스트를 말합니다. 데이터베이스와 연동한 코드가 잘 작동하는지, 여러 함수와 클래스가 엮인 로직이 잘 작동하는지 등을 확인합니다.
-   유닛 테스트보다는 복잡하고 느리지만, 소프트웨어는 결국 여러 코드 로직의 통합이라는 점에서 통합 테스트 역시 중요합니다.

**E2E 테스트**

-   E2E는 End To End의 약자로, 끝에서 끝, 즉 클라이언트 입장에서 테스트해보는 것입니다.
    예를 들어 Lambda 함수의 Input으로 Event 정보를 넣었을 때 Output으로 S3 객체가 잘 생성되었는지 확인하는 것도 E2E 테스트로 볼 수 있습니다.

-   테스트하는 대상의 Input과 Output이 명확해졌을 때 작성해야 하며 외부 의존성도 함께 구현해서 테스트하게 됩니다. 예를 들어 Lambda 함수의 E2E 테스트를 할 때 S3라는 외부 의존성이 필요합니다.

-   테스트 중 가장 신경써야 할 것이 많지만, 내부의 구현이 변경되도 검증이 유효하며 있고 시나리오에 맞춰 테스트할 수 있어 검증의 폭이 넓다는 장점이 있습니다.

PoC 단계에서는 빠르게 개발을 해야 했고, 데이터 파이프라인을 구성하기 위한 명확한 요구사항이 있다는 점에서 다른 테스트 보단 E2E 테스트를 중심으로 테스트 환경을 구축하였습니다.

### Docker Compose를 통한 E2E 환경 구성

E2E 테스트를 하기 위해선 테스트할 애플리케이션 뿐만 아니라 외부 의존성들도 함께 실행해야 합니다. 기본적으로 애플리케이션의 실행 및 운영은 도커 컨테이너 기반으로 수행되고 있었기에 테스트 환경 구성에 `Docker Compose`를 사용하였습니다. Docker Compose는 다중 컨테이너 서비스를 정의하고 실행하기 위해 많이 사용되는 도구입니다. 여러 도커 애플리케이션을 yaml 파일에서 정의/실행할 수 있어 E2E 테스트처럼 여러 의존성들을 직접 띄워야 할 때 유용하게 사용됩니다.

```yaml
version: '3.8'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:6.2.0
    container_name: zookeeper
    ...
  broker:
    image: confluentinc/cp-kafka:6.2.0
    container_name: broker
    depends_on:
      - zookeeper
    ...
  localstack:
    image: localstack/localstack
    container_name: localstack
    ...
  kafka-ui:
    image: provectuslabs/kafka-ui
    container_name: kafka-ui
    depends_on:
      - broker
    ...
  kafkacat:
    image: confluentinc/cp-kafkacat:5.4.9
    container_name: kafkacat
    depends_on:
      - broker
    ...
networks:
  shared-network:
    name: shared-network
    driver: bridge
```

위는 현재 파이프라인의 컴포넌트들을 띄우기 위한 docker-compose.yaml 파일의 일부입니다.
Kafka 환경을 중심으로 모니터링 툴인 kafka-ui, aws 서비스를 로컬 환경에서 실행할 수 있도록 mocking해주는 localstack 등이 있습니다. 이를 통해 로컬, CI 환경에서 각 컴포넌트들을 빠르게 띄우게 되며 호스트에서 localhost를 통해서 접근하거나 각 컨테이너 서비스 사이에 통신이 가능해졌습니다.

또한 외부 Docker Compose로 실행되는 컨테이너와 통신할 수 있도록 하기 위해서 network를 추가했습니다.

### Git Submodule을 통한 컴포넌트 별 E2E 테스트

파이프라인을 띄우는 Docker Compose 코드는 독립적인 Git Repository에서 관리되고 있습니다. FMS 프로젝트에 여러 컴포넌트들을 테스트해야 하는 것 뿐만 아니라 다른 프로젝트에서 사용할 수 있기 때문에 이를 분리하였습니다.

E2E 테스트 대상이 되는 Kafka Sink Connector, Lambda 같은 컴포넌트도 마찬가지로 독립적인 Git Repository로 관리되고 있습니다. 따라서 테스트할 Repository에서 파이프라인 구성 Repository를 연결할 수 있는 `Git Submodule`을 사용하였습니다.

예시로 Kafka Sink Connector의 E2E 테스트 폴더를 보면 아래와 같습니다. Git Submodule을 통해 Clone된 Repsoitory가 있는 걸 확인할 수 있습니다. 또한 `tests/`에 테스트 코드들이 들어있고 해당 테스트를 실행하는 `docker-compose.e2e.yaml`이 있습니다.

```bash
e2e
├── ...
├── docker-compose.e2e.yaml # 테스트할 대상들을 띄우기 위한 Docker Compose 파일
├── socar-fms-pipeline-docker # Git Submodule
│   ├── Makefile
│   ├── README.md
│   ├── docker-compose.yaml
│   └── scripts
└── tests
    ├── messages
    ├── test_ddb_sink_connector.sh
    └── test_s3_sink_connector.sh
```

Kafka Sink Connector의 E2E 테스트를 실행하기 위해서 Kafka Connect 애플리케이션과 테스트 스크립트(Bash로 작성)을 실행하는 Docker Compose 파일이 필요합니다. 아래와 같이 Service에 작성하였으며 위에서 다룬 스트리밍 파이프라인 쪽 컨테이너들과 통신하기 위해 External Network로 연결하였습니다.

```yaml
version: '3.8'
x-connect-test-configs: &connect-common
  environment: &connect-common-env
    ...

services:
  kafka-connect:
    <<: *connect-common
    image: ...
    build:
      context: ..
      dockerfile: Dockerfile
    container_name: kafka-connect
    networks:
      - network
      - default
    ...
  test-container:
    image: confluentinc/cp-kafkacat:5.4.9
    container_name: test-container
    volumes:
      - ./tests:/app/tests
    command:
      - /bin/bash
      - -c
      - |
        ...
        for f in /app/tests/*.sh; do
          echo "run $$f..." && bash $$f
        done
    networks:
      - network
      - default
    ...
networks:
  network:
    external:
      name: shared-network
```

그리고 실제로 컴포넌트가 제대로 동작하는지 검증하기 위한 테스트 코드는 아래와 같습니다. REST API로 테스트 할 Kafka Connector를 등록하고 Kafka Topic에 메시지를 보냈을 때 Sink에 제대로 적재가 되었는지 확인합니다. E2E 환경을 Docker Compose를 통해 전부 구축된 상황에서 Input을 넣고 Output을 확인하는 방식임을 알 수 있습니다.

```bash
#!/bin/bash
set -eo pipefail

Red='\033[0;31m'          # Red
Green='\033[0;32m'        # Green
Yellow='\033[0;33m'       # Yellow

kafkaConnectServer=kafka-connect:8083
targetTopic=s3-topic
deadletterTopic=deadletterqueue

echo "\n=============\n 0. Waiting for Kafka Connect to start listening on localhost \n============="
while [[ $(curl -s -o /dev/null -w %{http_code} $kafkaConnectServer) -ne 200 ]]; do
    echo "waiting..." && sleep 5
done;

echo "\n=============\n 1. test s3 sink connector exists \n============="
command=$(curl -s $kafkaConnectServer/connector-plugins | grep "kr.socar.fms.connector.s3.S3SinkConnector" | wc -l)
if [[ ${command} == 0 ]]; then
  echo "${Red}Fms kafka connect doesn't have S3SinkConnector"
  exit 1
else
  echo "${Green}passed"
fi

echo "\n=============\n 2. test s3 sink connector registered \n============="
command=$(echo '
            {
                "connector.class" : "kr.socar.fms.connector.s3.S3SinkConnector",
                "topics": "'"$targetTopic"'",
                ...
            }
          ' | curl -X PUT -d @- -s $kafkaConnectServer/connectors/s3-sink-connector/config --header "content-Type:application/json")


if [[ -z $(echo $command | jq -r ".error_code" ) ]]; then
  echo "${Green}passed"
else
  echo $command
  exit 1
fi

echo "\n=============\n 3. test s3 objects exist after putting messages \n============="
kafkacat -P -b broker:29092 -t $targetTopic -l /app/tests/messages/s3-sink-test.message
sleep 3 # wait for a work of s3 connector

result=$(aws --endpoint-url http://localstack:4566 s3api list-objects --bucket test-bucket)

s3FileSize=$(echo $result | jq -r '.Contents | length')
if [[ $s3FileSize == 2 ]]; then
  echo "${Green}passed"
else
  echo "${Red}S3SinkConnector doesn't put messages to s3 (object size : $s3FileSize)"
  exit 1
fi

echo "\n=============\n 4. test dlq after putting error messages \n============="
...

```

Kafka Sink Connector Docker Compose 환경에서 전부 실행했지만, Lambda 함수에 대한 E2E 테스트는 아래와 같이 python + pytest 환경에서 실행할 수 있습니다. 아래 코드는 Docker Compose의 일부 서비스(LocalStack S3)만 실행하여 E2E 테스트를 구현하였습니다.

```python
...

S3_BUCKET = "test-bucket"
S3_PREFIX = "raw"


def is_port_in_use(port: int) -> bool:
    import socket

    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        return s.connect_ex(("localhost", port)) == 0

...

@pytest.mark.skipif(
    not is_port_in_use(4566), reason="localstack s3 (포트번호 4566)를 실행해야 합니다"
)
def test_save_to_s3_in_parquet_with_partitions(s3_source_json_key, s3_parquet_parser):
    import awswrangler as wr

    wr.config.s3_endpoint_url = "http://localhost:4566"
    paritition_col = "partition"
    s3_prefix = "formatted"
    messages = [
        {"object": "vehicle", "type": "kinematic", paritition_col: "0"},
        {"object": "vehicle", "type": "status", paritition_col: "0"},
    ]

    result = s3_parquet_parser.save_to_s3_in_parquet_with_partitions(
        messages=messages,
        partition_cols=[paritition_col],
        s3_bucket=S3_BUCKET,
        s3_prefix=s3_prefix,
    )

    assert result["paths"][0].startswith(f"s3://{S3_BUCKET}/{s3_prefix}")

...
```

### Github Action을 통한 E2E 테스트 자동화

위에서 구성한 E2E 테스트는 사용자가 로컬에서 수행하는 것 뿐만 아니라 CI(Continuous Integration) 단계에서도 수행됩니다. 쏘카에서는 CI 도구로 `Github Action`을 사용하고 있습니다. 아래는 Github Action에서 E2E Test 관련 워크플로우 설정 파일입니다. 보통 Pull Request와 배포하기 전에 해당 워크플로우가 실행됩니다.

```yaml
name: E2E Test On Kafka Connect

on:
    pull_request:
    ...
jobs:
    e2e-test:
        runs-on: ubuntu-latest
        steps:
            - uses: actions/checkout@v3
              with:
                  persist-credentials: false
            - name: Checkout socar-data-pipeline-docker
              uses: actions/checkout@v3
              with:
                  repository: socar-inc/socar-data-pipeline-docker
                  ref: main
                  token: ${{ secrets.SOCAR_BOT_ACCESS_TOKEN }}
                  path: socar-data-pipeline-docker
            - name: Run e2e pipleine
              run: |
                  cd socar-data-pipeline-docker
                  docker-compose -p pipeline-docker --project-directory $PWD up  -d --force-recreate
                  cd ..
            - uses: actions/checkout@v3
              with:
                  persist-credentials: false
                  clean: false # pipeline-docker를 유지하기 위해서 사용합니다.
            - name: run e2e test
              run: |
                  cd e2e
                  docker-compose --project-directory $PWD -f docker-compose.e2e.yaml up --abort-on-container-exit
```

[`Checkout`](https://github.com/actions/checkout) Action을 사용하면 외부의 Github Repostiory를 손쉽게 checkout 할 수 있습니다. Github Action에서 하나의 Job은 하나의 격리된 환경에서 동작합니다. 따라서 E2E 환경을 구성하는 Repository로 Checkout하여 Docker Compose로 실행한 후, 다시 원 Repository로 checkout하여 E2E 테스트를 수행합니다. 이때 주의할 점은 checkout 할 때 `clean` 속성을 false로 줘야 기존 볼륨을 유지할 수 있습니다 (기수행된 컨테이너에 마운트 된 볼륨이 갑자기 사라지는 이슈로 꽤 골치가 아팠습니다)

## 6. 시뮬레이터를 활용한 실 데이터 기반 부하 테스트

### 실 데이터 기반의 메시지 시뮬레이터

-   MVP 서비스 런칭 전까지 데이터를 처리/저장하는 소프트웨어는 계속해서 데이터가 흘러야 했음
-   메시지 시뮬레이터를 구현해 실데이터 기반으로 메시지를 전송할 수 있도록 제공함
-   병렬 처리, 데이터 번형 전송이 가능하도록 구현

### 부하 테스트

-   부하 테스트 계획 세우기
-   주요하게 확인한 지표
-   부하테스트 진행
    -   차량 데이터 설정
    -   각 프로토콜 메시지를 병렬로 전송
-   부하테스트 결과 정리

## 7. 데이터 신뢰성을 위한 정합성과 무결성 검증하기

### 데이터 정합성/무결성이란?

-   정합성
-   무결성
-   중요한 이유

### 검사할 대상 정의하기

-   유형
    -   원본 데이터에 대한 차량 데이터 이상 현상 검사
    -   마트 테이블의 정합성 검사
    -   마트 테이블의 무결성 검사
-   검사지 만들기

### 정합성/무결성 검사 모니터링 구현하기

-   Airflow 활용
-   프로세스
    -   마트 테이블 무결성 검사는 마트 생성 Dag에서 진행
    -   원본 데이터 이상 여부 검사, 마트 테이블 정합성 검사는 하나의 Dag에서 진행
-   Dag 구현 코드

### 검사 결과 모니터링하기

-   Grafana를 통한 대시보드에서 확인
-   이상이 있는 경우 Slack Alert를 통해 확인

## 8. 마무리

### 남은과제

### 결론
