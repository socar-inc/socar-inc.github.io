---
layout: post
title: "차량용 단말을 위한 IoT 파이프라인 구축기 #1"
subtitle: "더 빨리, 더 많이, 더 믿을수 있게"
date: 2021-11-22 09:00:00 +0900
category: dev
background: '/assets/images/mike-benna-X-NAMq6uP3Q-unsplash.jpg'
author: spock
comments: true
tags:
    - iot
    - developer

---

<div class="photo-copyright">
Photo by <a href="https://unsplash.com/@mbenna?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Mike Benna</a> on <a href="https://unsplash.com/s/photos/pipeline?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>
</div>

### 뗄레야 뗄수없는 관계, 단말과 서비스

안녕하세요. 모빌리티 시스템 그룹 - 모비딕 팀의 스팍입니다.

쏘카가 서비스를 제공하기 위해서는 차량의 상태에 대한 정보가 필수적입니다. 사용이 끝난 차량이 정상적으로 제위치에 안전한 상태로 돌아왔는지 알지 못하면 무인으로 서비스를 운영할 수 조차 없지요.

따라서 차량의 정보를 수집하여 서버에 전달하며 고객의 요청에 따라 차량을 제어해주는 장치가 필수적으로 필요합니다.

![](/img/iot-pipeline-1/pipeline_concept.jpg){: width="75%" height="75%" style="display: block; margin: 0 auto"}

<br />

---


### 잠깐 쏘카의 구형 단말을 돌아봅시다

서비스 초기부터 쏘카는 차량 내에 서비스를 위한 단말을 장착하여 사용하고 있었습니다. 그리고 서비스가 급격하게 성장하던 시절에도 이 단말은 그럭저럭 제 역할을 해주었죠.

하지만 서비스를 시작하던 시점의 기술이 가진 한계성 때문에 단말은 잠재적인 문제를 갖고 있었습니다. 그 중 큰 몇가지를 꼽아보자면,

- SMS의 특성상 단말에 명령이 도달하는데 시간이 많이 소요 됩니다. 게다가 이 시간은 모든 지역에서 동일하지 않고 서울에서 멀어질수록 오래걸리는 경향이 있습니다.
- 데이터 수집을 HTTP로 하다보니 이를 위한 웹서버가 필요합니다.
- 데이터 전달 요청이 급증하면서 서버에 부하가 걸리면 데이터 수집에 지연이 발생합니다.

![](/img/iot-pipeline-1/old_device_arch.jpg){: width="75%" height="75%" style="display: block; margin: 0 auto"}

<br />

---

### 이렇게 된 이상 새로운 단말을 만든다!

그래서, 서비스 초창기에 없던 많은 기술적 대안과 진보를 활용하기 위해 새로운 단말을 만들기로 결정하였습니다.

새로운 단말을 만들때 기존 단말이 갖고 있던 한계점과 문제를 보완하기 위해 아래와 같은 기술 목표를 수립하였습니다.

- HTTP를 통한 단말 데이터 수집을 더이상 하지 않는다.
- SMS를 통한 차량 제어를 더이상 하지 않는다.

해서, 위와같은 요구사항에 맞춰 신규 단말은 [MQTT](https://ko.wikipedia.org/wiki/MQTT) 프로토콜 기반의 통신 방식을 채택하였습니다. MQTT 자체가 제한적인 IoT 기기에서 대용량의 데이터를 전송하기 위한 프로토콜이므로 기존 방식 대비 훨씬 적합한 프로토콜이라 하겠습니다.

MQTT를 지원하는 메시지 브로커는

- [mosquitto](https://mosquitto.org/)
- [VernaMQ](https://vernemq.com/)
- [HiveMQ](https://www.hivemq.com/)
- [RabbitMQ](https://www.rabbitmq.com/)

정도가 있습니다.

하지만, 위에 나열되어 있는 MQTT 브로커들을 직접 운영한다고 하면 아래와 같은 고민에 부딪히게 됩니다.

- 메시지 브로커들을 직접 관리해야 하므로, 시스템 운영 부하가 생깁니다.
- 대부분의 브로커들이 스스로에 대한 모니터링 방법은 제공하지만 어떤 단말이 연결중인지에 대한 정보까지는 제공해주지 않습니다.
- 서비스 안정성을 위해서 클러스터링 할수 있어야 하지만 이것을 지원해주지 않는 브로커도 많습니다.
- 보안 설정이 차량 제어라는 특수성을 고려할때 부족한 부분이 있습니다.

그리하여 이 고민을 해결하기 위해 관리되는 메시지 브로커 서비스를 사용하기로 결정하였고, [AWS IoT Core](https://aws.amazon.com/ko/iot-core/)를 사용하기로 결정하였습니다.

<br />

---

### 새술은 새부대에. AWS IoT Core.

AWS IoT Core를 쓰면 여러가지 이점을 얻을수 있습니다.

- 관리되는 서비스이므로 운영에 있어 수기로 작업해줘야 하는 부분이 줄어듭니다.
- IoT 단말들을 정책기반으로 운영할 수 있습니다.
- 어떤 단말들이 연결되었는지 확인이 가능합니다.
- 메시지 브로커에 대한 모니터링 역시 손쉽게 가능합니다.

![](/img/iot-pipeline-1/new_device_arch.jpg){: width="75%" height="75%" style="display: block; margin: 0 auto"}

AWS IoT Core를 쓸때 서비스를 운영하는 입장에서 가장 주요한 부분은 "관리되는 서비스"라는 점입니다. 카셰어링 서비스의 특성상 시간이나 시즌마다 데이터량이 달라지는 이슈를 피할수 없습니다. 그리고 서비스에 부하가 걸리더라도 항상 단말은 명령 수신과 상태 보고에 있어 준비된 상태를 유지해야 합니다. 브로커의 상태가 불안정하면 단말 자체에 문제가 없어도 서비스의 운영 안정성에 영향을 미치므로 관리되는 서비스가 주는 이점은 강력하다 하겠습니다.

---

### AWS IoT Core 쓰면 끝?

AWS IoT Core를 활용하여 데이터 수집을 하는것에서 더 나아가 각 부서(도메인)의 관점에 따른 유연한 데이터 활용이 가능해야 합니다. 과거에는 단말 데이터를 전부 DB에 저장하여 활용하는 방식을 사용했지만, 이 방식은 DB 부하를 불러온다는 단점을 가지고 있습니다.

모비딕 팀에서는 이 단점을 극복하기 위해 AWS IoT Core로부터 수집한 데이터를 [Amazon MSK](https://aws.amazon.com/ko/msk/)를 활용하여 흘려보내고 있습니다. 이러한 구조가 갖는 이점은 데이터활용을 위해 DB에 데이터를 굳이 저장할 필요가 없으며, MSK를 구독하는 컨슈머를 만들어 각 비즈니스 영역에 맞는 데이터 셋을 만들수 있다는 것입니다.

![](/img/iot-pipeline-1/aws_iot_arch.jpg){: width="75%" height="75%" style="display: block; margin: 0 auto"}

여기까지 쏘카에서 사용중인 단말들을 더 빠르게 제어하고 더 많은 데이터를 더 안정적으로 수집하기 위한 구조를 만들어 낸 과정을 설명 드렸습니다.

다음에는 실제 Amazon MSK를 구축할때 마주쳤던 문제들을 해결하는 과정에서 얻어진 글로 찾아뵙겠습니다.

<br />
