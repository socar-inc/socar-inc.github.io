---
layout: post
title: "On-premise to Cloud 마이그레이션 여정 (Security: FQDN)"
subtitle: "Aviatrix 솔루션을 이용한 Outbound FQDN 관리하기"
date: 2019-09-02 20:00:00 +0900
category: security
background: '/assets/images/andrea-enriquez-cousino-4hBCxfrlpoM-unsplash.jpg'
author: issac
comments: true
tags:
    - outbound
    - aviatrix
---

<div class="photo-copyright">
Photo by <a href="https://unsplash.com/@andreoiide?utm_medium=referral&amp;utm_campaign=photographer-credit&amp;utm_content=creditBadge">Andrea Enríquez Cousiño</a>
</div>

### On-premise to Cloud 마이그레이션 여정 (Security: FQDN)

On-premise 환경에서는 현재 회사의 성장세를 따라가기 어렵다고 판단하고, 1년 전부터 Cloud 환경으로 마이그레이션을 진행 하고 있습니다. 현재는 중요 서비스의 90% 이상이 Cloud 환경으로 마이그레이션이 진행 되었으며, 그 과정에서 Cloud 환경에 맞도록 많은 부분이 축소, 보완 되었습니다. 그 과정에서 보안성은 여러 기준으로 보완 되었으며 그 과정에서 여러 고민들 중에 공감 할 수 있는 내용을 공유하고자 합니다.

---

#### Cloud 환경에서 Outbound 트래픽 에 대한 관리를 어떻게 할까?
- Cloud platform에서 TCP/IP Outbound 에 대해서는 로그 및 관리가 다양한 Management Service로 지원하고 있지만, Outbound 트래픽 중에 [FQDN](https://en.wikipedia.org/wiki/Fully_qualified_domain_name) 에 대한 지원은 Cloud platform에서 Management Service로 대처가 어렵다고 판단하고 쏘카에 맞는 요구사항을 아래와 같이 정리 했습니다. 

```markdown
* 내부 요구사항 (Outbound - FQDN) +@(TCP/IP)
* 내부 요구사항 (Multiple Accounts and Clouds)
* 편의성 (Automation)
* 확장성, 관리 (infrastructure as code, update)
* 안정성 (다양한 레퍼런스, Monitoring, HA)
* 비용
```

- 위와 같은 요구사항을 정리하고 난 이후에 든 생각은 하나였습니다. `있을까?`  
그 이후에는 오픈소스([Squid](https://aws.amazon.com/ko/blogs/security/how-to-add-dns-filtering-to-your-nat-instance-with-squid/)), 유료 솔루션([paloalto](https://aws.amazon.com/marketplace/pp/B00PJ2V04O?qid=1567422902264&sr=0-2&ref_=srh_res_product_title), [Cisco vMX](https://aws.amazon.com/marketplace/pp/B01N49IN0S?qid=1567422994913&sr=0-16&ref_=srh_res_product_title) 등)을 검토 하여, 선정 과정에서 국내에서는 레퍼런스가 없는 [`Aviatrix`](https://aws.amazon.com/marketplace/pp/B079T2HGWG?qid=1567423038998&sr=0-1&ref_=srh_res_product_title) 솔루션을 선정하였습니다. 해외 레퍼런스의 경우에는 NASA, Netflix, hyatt 등 을 가지고 있었으며 다른 유료 솔루션들과는 다르게 필요한 Service(FQDN, TransitGW, VPN)에 대한 비용 발생 및 [IaC](https://en.wikipedia.org/wiki/Infrastructure_as_code) 배포, 구성이 Aviatrix 솔루션을 선정하게 되었습니다.

---

#### Aviatrix 솔루션 구축을 통한 요구사항 검토 (AWS)

Aviatrix 솔루션 테스트를 위해 AWS Marketplace 에서 Aviatrix 솔루션을 선택하여 Free Trial 이용이 가능한 Custom 유형을 선택 해서 테스트 할 수 있는 환경의 초기를 구성합니다. AWS 인프라(EC2 등)의 금액은 발생 되기 때문에 EC2 Type 선택 에서는 최소 스팩으로 테스트를 진행 합니다.
- [AWS-Marketplace (Aviatrix Secure Networking Platform - Custom)](https://aws.amazon.com/marketplace/pp/B0155GB0MA?ref_=aws-mp-console-subscription-detail)

설치 방법은 두가지를 제공합니다.
![1](/img/posts_aviatrix/cloudformation-image.png){: width="725" height="400"}{: .center}{: .center}

1. Amazon Machine Image
2. CloudFormation Template

원활한 테스트를 위해서 CloudFormation Template을 선택하여 진행합니다. Amazon Machine Image을 통한 구성 시에는 role, network 구성 등의 추가 설정이 필요합니다. 상세 설치 과정은 Aviatrix에서 Guide를 제공하고 있습니다.`(빠른 테스트 환경 구축을 위해 CloudFormation Template의 role, policy 등 설정은 아래에 별도로 기재해 드리겠습니다.)`
- [Aviatrix CloudFormation Template Guide](https://docs.aviatrix.com/StartUpGuides/aviatrix-cloud-controller-startup-guide.html)

CloudFormation 배포가 완료된 상태에서 output 카테고리에서 AviatrixControllerEIP 정보를 확인합니다. EIP에 대한 구성을 하지 않을 경우에는 AviatrixControllerPrivateIP 정보를 확인하여 웹 접속 주소로 사용 됩니다. `PrivateIP는 초기 임시 패스워드로도 구성` 됩니다.
```markdown
https://AviatrixControllerEIP
```

![2](/img/posts_aviatrix/cloudformation-output.png){: width="725" height="400"}{: .center}{: .center}

- AviatrixControllerEIP 접속 이후 관리자 아이디 및 버전을 설정 합니다.  
- 위의 설정을 완료할 경우에는 초기 로그인화면에서 Onboarding 화면이 노출됩니다.  
- 해당 부분에서 이용하고 있는 Cloud platform을 지정하고 추가적인 설정을 진행 합니다.

모든 설정을 완료한 이후에 제공되는 Aviatrix의 사용자 페이지 입니다.
![3](/img/posts_aviatrix/aviatrix-1.png){: width="725" height="400"}{: .center}{: .center}
- 위의 많은 카테고리 중에 FQDN을 로깅하고, 관리할 수 있는 카테고리가 보이네요. EC2 의 Outbound 에 대한 트래픽을 로깅 및 관리하기 위해서 우선적으로 Gateway의 설정이 필요하여, Aviatrix 사용자 카테고리에서 Gateway 설정으로 이동 합니다.

New Gateway 설정을 진행 합니다.
![10](/img/posts_aviatrix/aviatrix-gw-add.png){: width="725" height="400"}{: .center}{: .center}
- "OK"을 클릭할 경우에는 퍼센트에 대한 상태 이미지가 노출됩니다. 이후 작업은 직접 설치를 진행하지 않아도 자동으로 설치 및 Gateway 서버에 대한 환경이 구성 됩니다. `(HA구성의 경우에는 다양한 방법을 제공하고 있으며, 위의 이미지에서 체크하지 않은 많은 기능 들은 이번 블로그 에서는 다루지 않기 때문에 아래에 링크를 참조해 드립니다.)`
    * [`Gateway-HA`](https://docs.aviatrix.com/HowTos/gateway.html#gateway-single-az-ha)
    * [`HA-옵션`](https://docs.aviatrix.com/Solutions/gateway_ha.html)
    * [`Gateway`](https://docs.aviatrix.com/HowTos/gateway.html)

Private Subnet의 Route Table을 생성하여, 외부로 나가는 모든 트래픽이 Aviatrix-GW을 통해서 나가도록 설정을 진행 합니다. `"0.0.0.0/0"` 에 대한 Target을 Aviatrix-GW `ENI`을 지정 합니다. 자동 등록을 원할 경우 Aviatrix Controller 사용자 웹페이지에서 gateway > Aviatrix-GW: Edit > Source NAT 을 통한 설정 가능 합니다.
![11](/img/posts_aviatrix/route-table-avi.png){: width="725" height="400"}{: .center}{: .center}

FQDN 로깅 및 관리를 위하여, Aviatrix Controller 사용자 웹페이지에서 `security > Egress Control > Egress FQDN Filter` 부분을 설정합니다.

1. Egress FQDN Filter 생성 White List/Black List > Black 지정
2. 이후 `Prod환경` 도입의 경우에는 Egress FQDN View Log 을 확인하고, 사용하고 있는 FQDN을 보안성에 맞도록 분리하여, Egress FQDN Filter을 White List기반으로 변경할 수 있는 환경을 사전에 대비 합니다.

테스트를 위해서 FQDN Filter을 아래의 이미지와 같이 `White` 설정 이후에, Aviatrix-GW을 바라 보고 있는 `private-route-table`의 `private-Subnet`의 `인스턴스`에서 다음 명령어를 실행 하였습니다.
![12](/img/posts_aviatrix/fqdn-list.png){: width="725" height="400"}{: .center}{: .center}
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
- 위의 로그를 통해 `state`을 확인하고 후속 조치가 가능합니다. Aviatrix 솔루션에 대한 다양한 로그 관리 및 시각화 등이 가능하기 때문에 별도로 추가로 확인하고 싶으신 내용은 아래의 링크에서 확인을 하여 주시기 바랍니다.
    * [Aviatrix-Logging](https://docs.aviatrix.com/HowTos/AviatrixLogging.html)

- 위의 부분에서 `google.com` FQDN에 대해서 [호스트 명](https://en.wikipedia.org/wiki/Fully_qualified_domain_name)을 지정 하지 않을 경우에는 `"*"`로 적용 됩니다. 별도로 `*.google.com`을 등록하여도 가능 합니다.

#### **`AWS Account with Aviatrix Gateway Architecture`**

![13](/img/posts_aviatrix/fqdn-architecture.png){: width="725" height="800"}{: .center}{: .center}
1. `Private Subnet` 에서의 Outbound 발생 시 자체 설정한 `route table`을 참조
2. route table 에서 `"0.0.0.0/0"` 트래픽을 Aviatrix Gateway의 `ENI`로 전달
3. Aviatrix Gateway에 설정되어있는 `Aviatrix Controller` 정책에 따라 Outbound 트래픽 체크 이후에, Aviatrix Gateway에 Internet gateway로 전달
4. Internet Gateway을 요청한 내부 트래픽을 외부로 전달

---

#### `CloudFormation Template의 role, policy`
```markdown
* CloudFormation Template은 어떤 내용을 가지고 있을까?
* 왜 Gateway 서버가 자동으로 설치 되었을까?
* 멀티 Account 구성은 어떻게 가능할까?
```
위에 대한 내용은 CloudFormation Template의 `role`, `policy` 관계를 보면 알 수 있습니다.
아래의 소스는 CloudFormation Template의 일부 내용 입니다.

- EC2 생성 과정에서 aviatrix-role-ec2를 등록하는 내용 입니다.

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
- EC2에 등록하기 위한 role-ec2를 생성하는 내용 입니다.

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
- 여기서 주목할 점이 있습니다, role-app을 사용할 수 Service에 "Ref": "AWS::AccountId" 등록을 통해서 가능하도록 설정되는 것을 알수 있습니다.

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
- 여기서 주목할 점은 role/aviatrix-* 설정에 있습니다. 해당 설정을 통해서 role-ec2의 policy설정을 통해 arn:aws:iam::123456789:role/aviatrix-role-app의 policy을 할당 받아서 사용할 수 있게 됩니다.

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
- role-app policy 의 경우에는 아래 추가된 이미지 처럼 많은 Action이 정의 되어있습니다.

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
![4](/img/posts_aviatrix/role-check.png){: width="725" height="400"}{: .center}{: .center}

CloudFormation Template으 로 구성된 `AWS 인프라의 이미지`를 통해 정리할 경우 아래와 같은 구성이 설정 됩니다.

1. Aviatrix Controller Instance 에 role-ec2 설정

![5](/img/posts_aviatrix/role-ec2.png){: width="725" height="300"}{: .center}{: .center}

2. role-ec2 에 대한 policy 는 아래와 같이 설정되며, 해당 설정에서 `STS 설정을 주의 깊게 이헤 합니다`. 해당 부분을 JSON 형태로 확인하면 이해가 더 쉽게 된다.
```json
        {
            "Action": [
                "sts:AssumeRole"
            ],
            "Resource": "arn:aws:iam::*:role/aviatrix-*",
            "Effect": "Allow"
        },
```
![6](/img/posts_aviatrix/role-ec2-policy.png){: width="725" height="300"}{: .center}{: .center}

3. Aviatrix Controller 웹페이지 `Onboarding 카테고리`에서 Cloud platform에 맞게 설정을 하게 되면 AWS의 경우 `role-app`을 `role-ec2`가 `role-app`의 `policy`을 `위임` 받아서 사용할 수 있는 상태가 된다. role-ec2는 Aviatrix Controller Instance 에 등록 되어있는 role 이기 때문에, Aviatrix Controller 웹페이지에서 role-app에 적용 되어있는 policy 에 대한 이용이 가능하다, `멀티 Account`의 경우에는 `멀티 Account`에 설정 되어 있는 `role-app`에 대한 `AccountId Trust 등록`을 진행하여, `멀티 Account`의 `role-app`을 `Controller Account`의 `role-app`가 `공유` 받아 사용하는 방법으로 `동일` 합니다.

![7](/img/posts_aviatrix/role-app-trust.png){: width="725" height="300"}{: .center}{: .center}

#### **`Multiple AWS Accounts with Role Switchin Aviatrix Architecture`**

![8](/img/posts_aviatrix/role-ec2-app-muac.png){: width="725" height="400"}{: .center}{: .center}

---

### 정리
* 자세한 설명을 하기 위해서, 많은 이미지가 추가 되었지만 실질적으로는 AWS 마켓플레이스에서 라이센스 구입 이후에, 진행되는 절차가 간단하고 사용자가 직접 `수동으로 작업`할 내용자체가 `없습니다`.
* Aviatrix의 경우에는 기존의 Cisco 및 paloalto와는 다른 Cloud 환경에 맞게 개발이 되었다는 것을 쉽게 느낄 수 있었습니다. `Role`의 `활용` 및 위에서는 자세하게 다루지 않았지만, `"EXPORT TO TERRAFORM"` 카테고리 부분에서 리소스 형식에 맞는 *.tf 파일들을 다운로드 받아서 `IaC` 환경에 활용이 가능합니다.
* 테스트 과정에서는 단일 Account을 활용 하였지만, 실질적으로 쏘카 에서는 `Transit GW`을 이용한 `트래픽 집중 관리`를 통해서 운영하고 있기때문에 On-premise 적용 등 다양한 아키텍처 등이 가능 합니다.
* 매니저 Cloud Platform에서 모두 운영이 가능하기 때문에 이후 확장성 및 특정 Cloud Platform에 [`lock-in`](https://en.wikipedia.org/wiki/Vendor_lock-in) 되지 않는다는 장점이 있습니다.
* 그 밖에 `사용한 만큼만 지불`하는 Cloud 방식에 맞는 `라이센스 정책`도 비용적인 부분에서는 많은 장점이 있다고 생각 했습니다.

---

### 계획중인 Aviatrix를 이용한 다양한 환경 구축 테스트
* Aviatrix 로깅, 분석(시각화/자동화)
* DMZ 구축 (+VDI)
* Remote Work의 다양한 설계 (+VPN, +VDI)
* Inbound 트래픽에 대한 세부적인 로깅 및 분석(시각화)
* VM to VM 간의 트래픽 관리