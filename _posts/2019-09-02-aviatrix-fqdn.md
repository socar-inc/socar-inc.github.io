---
layout: post
title: "AWS VPC에서 FQDN Outbound Control"
subtitle: "Aviatrix Gateway를 이용한 FQDN Outbound Control"
date: 2019-09-02 20:00:00 +0900
category: security
background: '/assets/images/andrea-enriquez-cousino-4hBCxfrlpoM-unsplash.jpg'
author: issac
comments: true
tags:
    - outbound
    - aviatrix
    - fqdn
---

<div class="photo-copyright">
Photo by <a href="https://unsplash.com/@andreoiide?utm_medium=referral&amp;utm_campaign=photographer-credit&amp;utm_content=creditBadge">Andrea Enríquez Cousiño</a>
</div>

### AWS VPC에서 FQDN Outbound Control

**On-premise** 환경에서 현재 회사의 성장세를 따라가기 어렵다고 판단하고, 1년 전부터 Cloud 환경으로 <u>마이그레이션</u>을 진행하고 있습니다. 현재는 중요 서비스의 `90%` 이상이 Cloud 환경으로 마이그레이션 되었으며, 그 과정에서 인프라를 구성하는 많은 구성 요소가 변경, 대체 되었습니다. 또한, 여러 <u>보안 요구사항을 만족시키기 위해서</u> 추가적인 시스템 도입에 대해서 고민하였고, 그 과정을 공유하고자 합니다.

---

#### Outbound FQDN filtering을 하려는 이유
- **멀웨어 2차 확산 방지** (멀웨어에 감염된 경우 Outbound FQDN Filtering을 통해 멀웨어 [C&C](https://ko.wikipedia.org/wiki/C%26C_(%EC%95%85%EC%84%B1_%EC%86%8C%ED%94%84%ED%8A%B8%EC%9B%A8%EC%96%B4)) 서버에 연결하지 못하게 하고, <u>악성 코드가 컴퓨터의 데이터를 외부로 전송</u>하려고 하면 대상에 연결하지 못하게 할 수 있습니다.)
- Outbound 트래픽에 대한 무단 활동 감지
- google.com 등 **IP Range**가 넓고, [CDN](https://en.wikipedia.org/wiki/Content_delivery_network) 서비스 같은 <u>지속 해서 변하는 IP</u> 대응
- AWS - Security Groups 기준 Inbound or outbound rules per security group은 60개로 [제한](https://docs.aws.amazon.com/vpc/latest/userguide/amazon-vpc-limits.html) 됩니다. Inbound 관점에서 인터넷에서 고객 서비스는 특이사항이 아닐 수 있으나, Outbound 관점에서는 updates.ubuntu.com(IP 15), Github(IP 12) 등 타사의 업데이트 및 API를 생각하면 <u>60개의 IP 제한은 충분하지 않다</u>는 것을 알 수 있습니다.

#### Cloud 환경에서 Outbound 트래픽에 대한 관리를 어떻게 할까?
- Cloud platform에서 <u>TCP/IP Outbound</u>에 대해서는 로그 및 관리를 다양한 Management Service로 지원하고 있지만, Outbound 트래픽 중에 [**FQDN**](https://en.wikipedia.org/wiki/Fully_qualified_domain_name)에 대한 지원은 **Management Service 만으로 대처가 어렵**다고 판단하여 쏘카에 맞는 요구사항을 아래와 같이 정리했습니다.

```markdown
* Outbound - FQDN filtering +@(TCP/IP)
* HTTP/HTTPS +@((Wildcard)*.domain.com)
* Multiple Accounts and Clouds
* 확장성, 관리 (Infrastructure as code, Update)
* 안정성 (다양한 레퍼런스, Monitoring, HA)
* 편의성
* 비용
```

- 위와 같은 요구사항을 정리했습니다. ~~있을까요?.~~   
그 이후에는 오픈소스([Squid](https://aws.amazon.com/ko/blogs/security/how-to-add-dns-filtering-to-your-nat-instance-with-squid/)), 유료 솔루션([paloalto](https://aws.amazon.com/marketplace/pp/B00PJ2V04O?qid=1567422902264&sr=0-2&ref_=srh_res_product_title), [Cisco vMX](https://aws.amazon.com/marketplace/pp/B01N49IN0S?qid=1567422994913&sr=0-16&ref_=srh_res_product_title) 등)을 검토 하였고, 그 중 국내에서는 레퍼런스를 찾아보기 쉽지 않지만 [`Aviatrix`](https://aws.amazon.com/marketplace/pp/B079T2HGWG?qid=1567423038998&sr=0-1&ref_=srh_res_product_title) 솔루션을 선정하였습니다. 선정 과정에서 확인해본 결과 <u>해외 레퍼런스의 경우에는 NASA, Netflix, Hyatt 등</u>이 있었으며, 요구 사항에서 필요로 하는 **Service(FQDN, TransitGW, VPN)에 대한** 비용 발생 및 [IaC](https://en.wikipedia.org/wiki/Infrastructure_as_code) 배포, 구성이 타 유료 솔루션보다 이점이 있다는 것을 감안하여 Aviatrix 솔루션을 선정하게 되었습니다. 선정 과정에서 도움이 된 링크를 공유합니다.

    * [AWS 기반 Aviatrix FQDN Egress Filtering](https://aws.amazon.com/quickstart/architecture/aviatrix-fqdn-egress-filtering/?nc1=h_ls)
    * [Controlling Outbound VPC Traffic](https://www.aviatrix.com/solutions/egress-security.php)
    * [AWS Answers - Controlling VPC Egress Traffic](https://aws.amazon.com/answers/networking/controlling-vpc-egress-traffic/)

---

#### Aviatrix 솔루션 구축을 통한 요구사항 검토 (AWS)

Aviatrix 솔루션 테스트를 위해, AWS Marketplace에서 Aviatrix 솔루션을 선택 후 **Free Trial** 이용이 가능한 Custom 유형을 선택하여 테스트 초기 환경을 구성합니다. <u>AWS 인프라(EC2 등) 사용 금액이 발생하기 때문에 EC2 Type은 최소 스펙</u>을 선택하여 테스트를 진행합니다.
- [AWS-Marketplace (Aviatrix Secure Networking Platform - Custom)](https://aws.amazon.com/marketplace/pp/B0155GB0MA?ref_=aws-mp-console-subscription-detail)

설치 방법은 두가지를 제공합니다.
![1](/img/posts_aviatrix/cloudformation-image.png){: width="100%" height="100%"}

1. Amazon Machine Image
2. CloudFormation Template

원활한 테스트를 위해 <u>CloudFormation Template</u>을 선택하여 진행합니다. Amazon Machine Image를 통한 구성 시에는 role, network 구성 등의 추가 설정이 필요합니다. 상세 설치 과정은 Aviatrix에서 Guide를 제공하고 있습니다. <u>(CloudFormation Template 의 IAM role, policy 등은 아래에 별도 분석하겠습니다.)</u>
- [Aviatrix CloudFormation Template Guide](https://docs.aviatrix.com/StartUpGuides/aviatrix-cloud-controller-startup-guide.html)

CloudFormation 배포가 완료된 상태에서 output 카테고리에서 AviatrixControllerEIP 정보를 확인합니다. EIP에 대한 구성을 하지 않을 경우에는 AviatrixControllerPrivateIP 정보를 확인하여 웹 접속 주소로 사용됩니다. **PrivateIP는 초기 임시 패스워드로도 구성**됩니다.
```markdown
https://AviatrixControllerEIP
```

![2](/img/posts_aviatrix/cloudformation-output.png){: width="100%" height="100%"}

- AviatrixControllerEIP 접속 이후 관리자 아이디 및 버전을 설정합니다.  
- 위의 설정을 완료할 경우에는 초기 로그인화면에서 Onboarding 화면이 노출됩니다.  
- 해당 부분에서 이용하고 있는 Cloud platform을 지정하고 추가적인 설정을 진행합니다.

모든 설정을 완료한 이후에 제공되는 Aviatrix의 사용자 페이지 입니다.
![3](/img/posts_aviatrix/aviatrix-1.png){: width="100%" height="100%"}
- EC2 의 Outbound 트래픽을 로깅 및 관리하기 위해서 우선적으로 Gateway의 설정이 필요해서, Aviatrix 사용자 카테고리에서 Gateway 설정으로 이동합니다.

New Gateway 설정을 진행합니다.
![10](/img/posts_aviatrix/aviatrix-gw-add.png){: width="100%" height="100%"}
- "OK"를 클릭할 경우 퍼센트에 대한 상태 이미지가 노출됩니다. 이후 작업은 직접 설치를 진행하지 않아도 자동으로 설치 및 Gateway 서버에 대한 환경이 구성됩니다. `(HA 구성의 경우에는 다양한 방법을 제공하고 있어 아래 링크를 추가해 드립니다, Aviatrix 깊게 들여다보기(HA, Egress FQDN Discovery) 내용은 블로그 하단에 분류하였습니다.)`
    * [`Gateway`](https://docs.aviatrix.com/HowTos/gateway.html)
    * [`Gateway-HA`](https://docs.aviatrix.com/HowTos/gateway.html#gateway-single-az-ha)
    * [`HA-옵션`](https://docs.aviatrix.com/Solutions/gateway_ha.html)

Private Subnet의 Route Table을 생성해, 외부로 나가는 모든 트래픽이 Aviatrix-GW를 통해서 나가도록 설정을 진행합니다. `"0.0.0.0/0"` 에 대한 Target을 Aviatrix-GW <u>ENI</u>로 지정합니다. 자동 등록을 원할 경우 Aviatrix Controller 사용자 웹페이지에서 gateway > Aviatrix-GW: Edit > Source NAT를 통한 설정이 가능합니다.
![11](/img/posts_aviatrix/route-table-avi.png){: width="100%" height="100%"}

FQDN 로깅 및 관리를 위해, Aviatrix Controller 사용자 웹페이지에서 **security > Egress Control > Egress FQDN Filter**를 설정합니다.

1. Egress FQDN Filter 생성 White List/Black List > Black 지정
2. 이후 **실 서버 환경** 도입의 경우 Egress FQDN View Log를 확인하고, 사용하고 있는 FQDN을 보안성에 맞도록 분리하여, Egress FQDN Filter를 White List기반으로 변경할 수 있는 환경을 사전에 대비합니다.

테스트를 위해서 FQDN Filter를 아래의 이미지와 같이 <u>White</u> 설정한 이후에, Aviatrix-GW를 바라 보고 있는 <u>private-route-table</u> 의 <u>private-Subnet 인스턴스</u>에서 다음 명령어를 실행하였습니다.
![12](/img/posts_aviatrix/fqdn-list.png){: width="100%" height="100%"}
```bash
curl -L -k -s -o /dev/null -w "%{http_code}\n" https://www.naver.com
curl -L -k -s -o /dev/null -w "%{http_code}\n" https://google.com
curl -L -k -s -o /dev/null -w "%{http_code}\n" https://www.google.com
curl -L -k -s -o /dev/null -w "%{http_code}\n" https://docs.google.com
```
- 위의 테스트에 대한 Egress FQDN View Log

```r
2019-09-01T16:40:35.886612+00:00 ip-172-31-14-85 avx-nfq: AviatrixFQDNRule[CRIT]nfq_ssl_handle_client_hello() L#274  Gateway=Aviatrix-GW S_IP=172.31.24.52 D_IP=210.89.164.90 hostname=www.naver.com state=NO_MATCH drop_reason=NOT_WHITELISTED
2019-09-01T16:40:53.442375+00:00 ip-172-31-14-85 avx-nfq: AviatrixFQDNRule[CRIT]nfq_ssl_handle_client_hello() L#274  Gateway=Aviatrix-GW S_IP=172.31.24.52 D_IP=172.217.27.78 hostname=google.com state=MATCHED
2019-09-01T16:40:53.654101+00:00 ip-172-31-14-85 avx-nfq: AviatrixFQDNRule[CRIT]nfq_ssl_handle_client_hello() L#274  Gateway=Aviatrix-GW S_IP=172.31.24.52 D_IP=216.58.197.164 hostname=www.google.com state=MATCHED
2019-09-01T16:40:58.707731+00:00 ip-172-31-14-85 avx-nfq: AviatrixFQDNRule[CRIT]check_ip() L#65  Gateway=Aviatrix-GW S_IP=172.31.24.52 D_IP=216.58.197.164 hostname=www.google.com state=MATCHED
2019-09-01T16:41:21.940692+00:00 ip-172-31-14-85 avx-nfq: AviatrixFQDNRule[CRIT]nfq_ssl_handle_client_hello() L#274  Gateway=Aviatrix-GW S_IP=172.31.24.52 D_IP=172.217.26.14 hostname=docs.google.com state=MATCHED
2019-09-01T16:41:22.323298+00:00 ip-172-31-14-85 avx-nfq: AviatrixFQDNRule[CRIT]nfq_ssl_handle_client_hello() L#274  Gateway=Aviatrix-GW S_IP=172.31.24.52 D_IP=172.217.25.77 hostname=accounts.google.com state=MATCHED
```
- 위 로그를 통해 **state**를 확인하고 후속 조치가 가능합니다. Aviatrix 솔루션에 대한 다양한 로그 관리 및 시각화 등이 가능하기 때문에, 별도 추가로 확인하고 싶으신 내용은 아래의 링크에서 확인해 주시기 바랍니다.
    * [Aviatrix-Logging](https://docs.aviatrix.com/HowTos/AviatrixLogging.html)

- 위 내용에서 <u>google.com</u> FQDN에 대해서 [호스트 명](https://en.wikipedia.org/wiki/Fully_qualified_domain_name)을 지정하지 않을 때에는 `"*"`로 적용됩니다. 별도로 <u>*.google.com</u>을 등록하여도 가능합니다.

#### **`AWS Account with Aviatrix Gateway Architecture`**

![13](/img/posts_aviatrix/fqdn-architecture.png){: width="100%" height="100%"}

* Instance Outbound Flow & Aviatrix Controller Gateway HA Flow

<div class="mermaid">
graph LR;
  CLIENT[시작: Instance] -->|1.Private  Route Table| ag(Aviatrix Gateway)
    ag -->|2. Public Route Table| igw(Internet Gateway)
    igw -->|3| int[Internet]
    ac(시작: Aviatrix Controller) --> internet2[Internet]
    internet2[Internet]-->|Health Check| ag
    internet2[Internet]-->|Health Check| agha(Aviatrix Gateway HA)
    ag-->|Failover| agha(Aviatrix Gateway HA)
    agha-->|Failover| ag
</div>

* <u>Private Subnet</u>에서의 Outbound 발생 시 자체 설정한 <u>Route table</u>을 참조
* Route table에서 `"0.0.0.0/0"` 통신을 Aviatrix Gateway의 [`ENI`](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html)로 전달
* Aviatrix Gateway에 설정되어있는 <u>Aviatrix Controller</u> 정책에 따라 Outbound 트래픽 체크 이후에, Aviatrix Gateway에 Internet gateway로 전달
* Internet Gateway을 요청한 통신을 외부(Internet)로 전달
* <u>Aviatrix Controller</u>의 경우 Internet을 통해 Aviatrix Gateway Health Check
* Aviatrix Gateway 장애 발생시 Gateway HA의 <u>ENI</u>로 <u>Private Subnet의 Route table에 **"0.0.0.0/0"**</u> 업데이트

---

#### `Aviatrix 깊게 들여다보기`

##### **1. [HA(High Availability)](https://en.wikipedia.org/wiki/High_availability)**
HA 구성은 모든 인프라의 **기본**으로 Aviatrix를 사용할 경우에도 아래와 같은 간단한 작업으로 적용이 가능합니다.

* Gateway > Edit > Gateway Single AZ HA "Enable"
* Gateway > Edit > Gateway for High Availability Peering
    * HA 구성을 위해 <u>운영 중인</u> Gateway Subnet과 다른 [AZ](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html)의 <u>Public-Subnet</u>을 선택합니다.

![20](/img/posts_aviatrix/gateway-ha-public-b.png){: width="100%" height="100%"}

* HA 구성을 위한 설정은 마무리하였으며, 정상적인 **Failover**가 되는지 확인을 위해 아래 구성을 진행하였습니다.

```markdown
* Private-Subnet에서 운영되는 인스턴스에 대한 실시간 모니터링 설정
* Failover 기능 테스트를 위해 Aviatrix-GW 인스턴스를 AWS Console에서 강제 종료(STOP)
* Private-Subnet에서 운영되는 인스턴스에 대한 실시간 모니터링 지표 확인
```

* <u>Failover</u> 기능을 테스트하기 전의 Aviatrix Gateway 상태 이미지입니다.

![21](/img/posts_aviatrix/gateway-status.png){: width="100%" height="100%"}
* <u>Failover</u> 기능을 테스트하기 위해 Gateway를 강제 종료한 Aviatrix Gateway 상태 이미지입니다.

![22](/img/posts_aviatrix/gateway-status-2.png){: width="100%" height="100%"}

* **`테스트 결과:`** Private-Subnet에서 운영되고 있는 인스턴스에 모니터링 에이전트(Telegraf)를 설정하여, ICMP 프로토콜을 이용한 <u>1s</u> 기준으로 **Packet Loss** 현상이 발생 되지 않았습니다. 웹사이트 기준에서는 Gateway가 Failover 되는 시점을 인지할 수 없는 정도입니다.

![23](/img/posts_aviatrix/monitoring.png){: width="100%" height="100%"}

* `Aviatrix-Gateway HA Flow`

<div class="mermaid">
graph TB
    subgraph Public_A
Controller[Controller]
end
    Controller -x |1.Health Check 실패|Gateway
    Controller -x |2.AWS-Role을 이용한 table update|Private_Routetable
    Private_Routetable -x |3.traffic|Gateway_HA
    Controller-->|Health Check |Gateway
    Controller-->|Health Check|Gateway_HA
    subgraph Public
    subgraph Public_B
Gateway[Gateway]
end
    Gateway-->Public_Routetable
    subgraph Public_C
Gateway_HA[Gateway_HA]
end
end
    Gateway_HA-->Public_Routetable
    subgraph Private
    subgraph Private_A
Instance-A[Instance-A]
end
    Instance-A-->Private_Routetable
    subgraph Private_B
Instance-B[Instance-B]
end
    Instance-B-->Private_Routetable
    Private_Routetable-->Gateway
end
    Public_Routetable-->Internet
</div>

* AviatrixController -> Gateway **[Health Check]** -> <u>Fail</u>
* Gateway_HA로 네트워크 트래픽 변경
    * 문제 되는 Gateway로 설정되어 있던 Route Table 업데이트
    * 예) Private-Subnet > Route Table > **"0.0.0.0/0"** <u>Target Gateway_HA ENI</u>로 업데이트

##### **2. [Egress FQDN Discovery](https://docs.aviatrix.com/HowTos/fqdn_discovery.html)**

해당 기능은 **실 서버**에 적용하기에 앞서 실 서버에서 FQDN outbound의 <u>사용 리스트</u>를 정리하는데 유용한 기능입니다.

* Security > Egress Control > (Optional) Egress FQDN Discovery > Gateway "Start" (선택된 Gateway는 FQDN Filter에 연결이 안 되어 있어야 합니다.)

![23](/img/posts_aviatrix/fqdn-discovery-start.png){: width="100%" height="100%"}

* 테스트를 위해 Private-Subnet에서 운영되고 있는 인스턴스에서 아래 명령어를 실행합니다.

```bash
curl -L -k -s -o /dev/null -w "%{http_code}\n" https://tech.socarcorp.kr
```

* 위 HA 테스트로 인해 변경된 Aviatrix-GW-hagw에서 발생한 FQDN 내용을 확인할 수 있습니다.

![23](/img/posts_aviatrix/fqdn-discovery-status.png){: width="100%" height="100%"}

* **`테스트 결과:`** FQDN Discovery 기능을 통해 **실 서버** FQDN Outbound를 모두 사전에 확인하고, <u>필요 유무에 따라서 FQDN Filter 정책을 정의</u>하는 데 유용합니다.

##### **3. HTTPS/TLS 통신을 Gateway가 가로채서 어디로 가는지 확인할 수 있는 이유**
* 2번 내용을 보면 HTTPS/TLS 통신을 **Gateway가 어떻게?** <u>개로 채지 라는 의문점</u>이 있습니다, 해당 내용은 SNI에 대한 이해가 필요해서 SNI 내용을 정리해 드립니다.
* `SNI(Server Name Indication)`
    * 두 TCP 피어 간 암호화된 TLS 터널을 설정할 수 있고, 클라이언트는 서버에 대한 IP 주소 만 알고 있어도 TLS Handshake를 수행 할 수 있지만, <u>서버가 각각 고유한 TLS 인증서를 가진 여러 개의 독립 사이트를 동일한 IP 주소로 운영하는 문제를 해결하기 위해</u> SNI 확장이 TLS 프로토콜에 도입되었습니다.
    * SNI는 <u>여러 개의 독립 사이트 중에 하나의 사이트를 지정</u> 하기 위해 필요한 필드입니다.
    * 하지만, TLS Handshake에서 **암호화 통신**을 <u>하기 전인</u> **ClientHello** 과정에 적용됩니다.

* `TLS Handshake`

<div class="mermaid">
sequenceDiagram
opt 암호화 X
    Client ->> Server: ClientHello
    Server ->> Client: ServerHello
    end
opt 암호화
    Server ->> Client: ServerCertificate
    Server ->> Client: ServerHelloDone
    Client ->> Server: KeyExhange/ChangeCipherSpec/Finished
    Server ->> Client: ChangeCipherSpec/Finished
    end
    Server -> Client: Application Date
</div>

* `TLS Handshake > ClientHello`

<div class="mermaid">
sequenceDiagram
    Client ->> Gateway: TLS ClientHello, SNI=www.socar.kr, Client`s Diffie-Hellman Key
opt FQDN Filter
    Client --> Gateway: SNI Check
Note right of Client: FQDN Filter 성공
    Gateway ->> Server: TLS ClientHello, SNI=www.socar.kr, Client`s Diffie-Hellman Key
end
opt FQDN Filter
    Client --> Gateway: SNI Check
Note right of Client: FQDN Filter 실패
    Gateway -->> Client: TLS ClientHello, SNI=www.socar.kr, Client`s Diffie-Hellman Key
end
</div>

* Gateway가 SNI 필드를 확인하는 방법은 **암호화가 되지 않은 평문**으로 전송하기 때문입니다.
* 사용자 입장에서는 SNI는 접속하려는 사이트 주소가 **암호화되지 않은 평문**으로 전송되기 때문에, <u>타인</u>이 중간에 트래픽을 가로채서 <u>사용자가 조회하는 사이트</u>를 확인 할 수 있습니다.
* TLS 1.3 최종안이 조율되는 시기에는 **Encrypted SNI**로 인해서 Gateway가 SNI 필드를 확인하는 방법이 불가능 할 수도 있었지만, **TLS 1.3 최종안** 에서는 <u>필수</u>가 아닌, <u>확장</u> 기능으로써 추가되었습니다.

##### **4. Aviatrix-CloudFormation Template의 role, policy 이해하기**

* CloudFormation Template은 어떤 내용을 가지고 있을까?
* 왜 Gateway 서버가 자동으로 설치 되었을까?
* 왜 Private-Subnet Route Table 은 자동으로 업데이트가 되었을까?
* 멀티 Account 구성은 어떻게 가능할까?

위에 대한 내용은 CloudFormation Template의 **role**, **policy** 관계를 보면 알 수 있습니다.
아래의 소스는 CloudFormation Template의 일부 내용입니다.

- EC2 생성 과정에서 aviatrix-role-ec2를 등록하는 내용입니다.

```json
        "IAMRoleParam": {
            "Description": "Determine if IAM roles aviatrix-role-ec2 and aviatrix-role-app should be created.",
            "Default": "New",
            "Type": "String",
            "AllowedValues": [
                "aviatrix-role-ec2",
                "New"
            ]
        },
```
- EC2에 등록하기 위한 role-ec2를 생성하는 내용입니다.

```json
        "AviatrixRoleEC2": {
            "Type": "AWS::IAM::Role",
            "Condition": "AviatrixIAMRoleNotExist",
            "Properties": {
                "RoleName": "aviatrix-role-ec2",
                "AssumeRolePolicyDocument": {
                    "Version": "2012-10-17",
                    "Statement": [{
                        "Effect": "Allow",
                        "Principal": {
                            "Service": [
                                "ec2.amazonaws.com"
                            ]
                        },
                        "Action": [
                            "sts:AssumeRole"
                        ]
                    }]
                },
                "Path": "/"
            }
        },
```
- 여기서 주목할 점이 있습니다, role-app을 사용할 수 있는 [Principal](https://docs.aws.amazon.com/ko_kr/IAM/latest/UserGuide/reference_policies_elements_principal.html)에 “Ref”: “AWS::AccountId” 등록을 통해서 가능하도록 설정되는 것을 알 수 있습니다.

```json
        "AviatrixRoleApp": {
            "Type": "AWS::IAM::Role",
            "Condition": "AviatrixIAMRoleNotExist",
            "Properties": {
                "RoleName": "aviatrix-role-app",
                "AssumeRolePolicyDocument": {
                    "Version": "2012-10-17",
                    "Statement": [{
                        "Effect": "Allow",
                        "Principal": {
                            "AWS": [{
                                "Fn::Join": [
                                    "", [
                                        "arn:aws:iam::",
                                        {
                                            "Ref": "AWS::AccountId"
                                        },
                                        ":root"
                                    ]
                                ]
                            }]
                        },
                        "Action": [
                            "sts:AssumeRole"
                        ]
                    }]
                },
                "Path": "/"
            }
        },
```
- 여기서 주목할 점은 <u>arn:aws:iam::*:role/aviatrix-*</u> 설정에 있습니다. 해당 `"*"`설정으로 role-ec2의 policy는 arn:aws:iam::**{{AccountId}}**:role/aviatrix-role-app의 policy를 할당 받아서 사용할 수 있게됩니다.

```json
        "CreateAviatrixAssumeRolePolicy": {
            "Type": "AWS::IAM::ManagedPolicy",
            "Condition": "AviatrixIAMRoleNotExist",
            "Properties": {
                "ManagedPolicyName": "aviatrix-assume-role-policy",
                "Description": "Policy for creating aviatrix-assume-role-policy",
                "Path": "/",
                "PolicyDocument": {
                    "Version": "2012-10-17",
                    "Statement": [{
                            "Effect": "Allow",
                            "Action": [
                                "sts:AssumeRole"
                            ],
                            "Resource": "arn:aws:iam::*:role/aviatrix-*"
                        },
                        {
                            "Effect": "Allow",
                            "Action": [
                                "aws-marketplace:MeterUsage"
                            ],
                            "Resource": "*"
                        }
                    ]
                },
                "Roles": [{
                    "Ref": "AviatrixRoleEC2"
                }]
            }
        },
```
- role-app policy의 경우에는 아래 추가된 이미지처럼 <u>많은 Action</u>이 정의되어있습니다.

```json
        "CreateAviatrixAppPolicy": {
            "Type": "AWS::IAM::ManagedPolicy",
            "Condition": "AviatrixIAMRoleNotExist",
            "Properties": {
                "ManagedPolicyName": "aviatrix-app-policy",
                "Description": "Policy for creating aviatrix-app-policy",
                "Path": "/",
                "PolicyDocument": {
                    "Version": "2012-10-17",
                    "Statement": [{
                            "Effect": "Allow",
                            "Action": [..........................
```
![4](/img/posts_aviatrix/role-check.png){: width="100%" height="100%"}

CloudFormation Template 으로 구성된 **AWS 인프라의 이미지**를 통해 정리할 경우, 아래와 같은 구성이 설정됩니다.

* Aviatrix Controller Instance 에 role-ec2 설정

![5](/img/posts_aviatrix/role-ec2.png){: width="100%" height="100%"}
* role-ec2 에 대한 policy 는 아래와 같이 설정되며, 해당 설정에서 <u>**STS 설정**</u>을 주의 깊게 이해합니다.

![6](/img/posts_aviatrix/role-ec2-policy.png){: width="100%" height="100%"}

* 위의 이미지를 JSON 형태로 확인하면 이해가 더 쉽습니다.

```json
        {
            "Action": [
                "sts:AssumeRole"
            ],
            "Resource": "arn:aws:iam::*:role/aviatrix-*",
            "Effect": "Allow"
        },
```

* Aviatrix Controller 웹 페이지의 **Onboarding 카테고리**에서 Cloud platform에 맞게 설정을 하게 되면, AWS의 경우 <u>role-ec2</u>가 <u>role-app</u>의 policy을 **위임** 받아서 사용할 수 있는 상태가 됩니다. role-ec2는 Aviatrix Controller Instance에 등록 되어있는 role 이기 때문에, Aviatrix Controller 웹페이지에서 role-app에 적용 되어있는 policy 에 대한 이용이 가능합니다. <u>멀티 Account의 경우에는 멀티 Account에 설정 되어 있는 role-app에 대한 AccountId Trust 등록을 진행하여, 멀티 Account의 role-app을 Controller Account의 role-app이 공유 받아 사용하는 방법으로 동일</u>합니다.

![7](/img/posts_aviatrix/role-app-trust.png){: width="100%" height="100%"}

#### **`Multiple AWS Accounts with Role Switchin Aviatrix Architecture`**

![8](/img/posts_aviatrix/role-ec2-app-muac.png){: width="100%" height="100%"}

---

### 정리
* 자세한 설명을 하기 위해 많은 이미지가 추가 되었지만, 실질적으로는 AWS 마켓플레이스에서 라이선스 구입 이후에 진행되는 절차가 간단하며 사용자가 직접 <u>수동</u>으로 작업해야하는 내용이 **거의없습니다.**
* Aviatrix의 경우에는 기존의 Cisco 및 paloalto 와는 다른 Cloud 환경에 맞게 개발이 되었다는 것을 쉽게 느낄 수 있었습니다. **Role의 활용** 및 위에서는 자세하게 다루지 않았지만, <u>"EXPORT TO TERRAFORM"</u> 카테고리 부분에서 리소스 형식에 맞는 *.tf 파일들을 다운로드 받아서 **IaC** 환경에 활용이 가능합니다.
* 테스트 과정에서는 단일 Account를 활용하였지만, 쏘카에서는 **Transit GW**를 이용한 **트래픽 중앙** 관리를 통해서 운영하고 있기 때문에 On-premise 적용 등 다양한 아키텍처 구성이 가능합니다.
* 매니저 Cloud Platform에서 모두 운영이 가능하기 때문에 이후 확장성 및 특정 Cloud Platform에 [`lock-in`](https://en.wikipedia.org/wiki/Vendor_lock-in) 되지 않는다는 장점이 있습니다.
* 그 밖에 **사용한 만큼만 지급**하는 Cloud 방식에 맞는 <u>라이선스 정책</u>도 비용적인 부분에서는 많은 장점이 있다고 생각했습니다.
* **많은 장점** 가운데 **지속가능성**에 대한 문제! TLS 1.3 최종안이 적용된 지 얼마 되지 않은 상황이지만, Gateway가 SNI 필드를 통해 FQDN Filter 과정이 이후 <u>SNI 암호화 이후 FQDN Filter를 어떻게 진행할 것인가에 대한 문제점</u>이 있습니다.

---

### 계획중인 Aviatrix를 이용한 다양한 환경 구축
* Aviatrix 로깅, 분석(시각화/자동화)
* DMZ 구축 (+VDI)
* Remote Work의 다양한 설계 (+VPN, +VDI)
* Inbound 트래픽에 대한 세부적인 로깅 및 분석(시각화)
* VM to VM 간의 트래픽 관리