---
layout: post
title: "Keycloak를 이용한 SSO 구축(web + wifi + ssh)"
date: 2019-07-31 17:20:00 +0900
category: security
tags:
    - keycloak
    - sso
---

## 사내 SSO(Single Sign-On) 구축하기
  이번에 쏘카에서 사내 보안 강화를 위해 SSO를 구축 하였습니다.
  쏘팸과 파트너등이 업무에 사용하는 관리시스템(web) 및 wifi, ssh 접속 인증을 1개의 ID로 관리 하도록 하는 것이 목적입니다.
  
  정리된 요구 사항은 다음과 같습니다. 

```markdown
  1. Web 사이트 인증.(Spring 베이스 사이트)
  2. Wifi 인증
  3. SSH 인증(+ Sudo 인증)
  4. 2차 인증 (OTP) 지원
  5. 유연한 권한 설정(Authorization)
```

  여러 유료 솔루션(([Gluu](https://www.gluu.org/), [Google Identity Platform](https://developers.google.com/identity/) 등)도 검토 하였으나 설치형 + 오픈소스로 무료로 사용 가능한 [`Keycloak`](https://www.keycloak.org/)으로 구축으로 방향을 잡았습니다.
  
  Wifi 인증([FreeRADIUS에서 LDAP 모듈 지원](https://freeradius.org/modules/?s=ldap&mod=rlm_ldap))이나 ssh인증([pam_ldap](https://www.tldp.org/HOWTO/archived/LDAP-Implementation-HOWTO/pamnss.html))의 경우 OpenLDAP과 연동해서 처리 하는 방법에 대한 자료가 더 많지만, Web기반으로 운용 가능한 시스템을 더 선호해서 Keycloak을 선정하게 되었습니다. ~~험한길을 가게 되었습니다.~~

-----
## [Keycloak](https://www.keycloak.org/)이란?

```markdown
  - open source + 설치형
  - OpenID / SAML 지원
  - Restful API 지원 및 커스텀 API 추가 가능
  - RedHat에서 지원하는 프로젝트
  - keycloak-gatekeeper 등을 이용하면 Web 사이트 권한 처리도 용이
```

  - github: https://github.com/keycloak/keycloak
  - [RedHat SSO](https://access.redhat.com/products/red-hat-single-sign-on)의 오픈소스 버전

-----
## Keycloak으로 요구사항의 내용을 어떻게 구현 할까?

###### 1. Web 사이트 인증
  - OpenID / SAML 프로토콜 지원 및 Java client 지원
  - reverse proxy로 동작하는 [gatekeepr](https://github.com/keycloak/keycloak-gatekeeper)를 이용해 Web Application에서는 인증 / 권한 코드를 작성하지 않고 처리 가능

###### 2. Wifi 인증
  - 사내에 Cisco 라우터를 사용하고 있고, RADIUS 프로토콜을 지원하므로 [RADIUS 서버](https://ko.wikipedia.org/wiki/RADIUS)를 구축하면 가능
  - [FreeRADIUS](https://freeradius.org)의 [Python 모듈](https://wiki.freeradius.org/modules/Rlm_python)에서 keycloak API를 호출해서 인증을 처리 하면 가능

###### 3. SSH 인증(+ Sudo 인증)
  - [sshd_config](https://linux.die.net/man/5/sshd_config)설정에서 UsePAM 옵션을 활성화 시키고 [pam_radius_auth.so](https://github.com/FreeRADIUS/pam_radius)를 사용해 RADIUS 서버를 통해서 인증 가능

###### 4. 2차 인증 (OTP) 지원
  - 지원됨.
  - Wifi 인증시에는 사용하지 않을 예정이며, SSH(+ sudo) 인증시에는 [Access-Challenge + Reply-Message](https://www.iana.org/assignments/radius-types/radius-types.xhtml)를 회신하면 [pam_radius_auth.so](https://github.com/FreeRADIUS/pam_radius)에서 사용자에게 추가 입력을 받을 수 있음

###### 5. 유연한 권한 설정(Authorization)
  - keycloak의 권한 설정은 상당히(~~과하게~~) 유연함. ([참고](https://www.keycloak.org/docs/4.8/authorization_services/))

-----

## 설치 계획 및 구성
- Keycloak은 `Kubernetes 클러스터`에 설치
- FreeRADIUS는 `별도 서버에 docker-compose`로 설치
  - RADIUS 프로토콜은 `UDP` Port2개를 사용합니다.(default: 1812, 1813)
  - ~~`UDP`는 AWS에서 Load Balancer를 사용할 수 없습니다.~~ [셋팅 완료 후 .. 얼마 전부터 지원시작](https://aws.amazon.com/ko/blogs/aws/new-udp-load-balancing-for-network-load-balancer/)
- 대략적인 구성은 아래와 같습니다
<div class="mermaid">
graph TD;
  subgraph Wifi
    CISCO_ROUTER[Cisco 무선공유기]
  end

  subgraph Linux-Servers
    PAM_RADIUS[SSH 접속]
  end

  subgraph Kubernetes-Cluster
    KEYCLOAK[Keycloak]
    WEB_APP[웹서비스]
  end
  
  subgraph Other-Server
    FREE_RADIUS[FreeRADIUS]
  end

  CISCO_ROUTER-->|RADIUS Protocol|FREE_RADIUS
  FREE_RADIUS-->|HTTP API|KEYCLOAK
  WEB_APP-->|HTTP API|KEYCLOAK
  PAM_RADIUS -->|RADIUS Protocol|FREE_RADIUS
  USER -->|관리시스템 접속|WEB_APP
  USER -->|Wifi 접속|CISCO_ROUTER
  USER -->|SSH 접속|PAM_RADIUS
</div>

-----
## Kubernetes(AWS EKS)에 Keycloak 설치
  - **[helm](https://helm.sh/)이 설치되어 있어야 합니다.**
  - Keycloak는 [Helm Chart](https://github.com/codecentric/helm-charts/tree/master/charts/keycloak)로 제공 되고 있습니다.
  - 설치전에 사전에 설정되어야 할 내용들 입니다.
    - [nginx-ingress](https://github.com/helm/charts/tree/master/stable/nginx-ingress) 설치
    - AWS의 Route53에 `*.mydomain.com`을 nginx-ingress 설치 후 생성된 LB로 연결.
      - Route53에 `*.mydomain.com`를 사용하면 등록되지 않은 subdomain은 모두 이곳으로 연결되게 됩니다.
    - RDS로 MySQL 사용.
  - helm install시에는 [command line에서 매개변수로 설정](https://github.com/codecentric/helm-charts/tree/master/charts/keycloak#configuration)을 할수도 있지만 변경해야 될 값들이 많으므로 `values.yaml`을 수정해서 사용합니다.
  - 아래는 설치한 환경에 맞게 설정하기 위해 `values.yaml`에서 수정해야 할 할 부분 입니다.

```yaml
...

keycloak:
...
  # 초기 admin ID
  username: keycloak

  # 초기 admin password
  # 파일을 저장하거나 git 등으로 관리 할 경우 비밀번호 노출 위험이 있음.
  # 설치 이후 바로 암호 변경 권장.
  password: "keycloak1234"

  # 위에 'password'에 암호를 직접 입력하지 않고 kubernetes의 secret를 사용할 경우 패스워드가 저장된 Secret의 이름.
  existingSecret: ""

  # 위 'existingSecret'에 지정한 Secret 중 사용할 값이 저장된 key 값.
  existingSecretKey: password

...

  ingress:
    # ingress 사용 여부
    enabled: true
    path: /

    annotations: {}
      # ssl 오프로딩 사용을 위한 설정
      nginx.ingress.kubernetes.io/ssl-redirect: "false"

    labels: {}
    # key: value

    hosts:
      # 사용할 domain
      - keycloak.mydomain.com

...

  persistence:
    # If true, the Postgres chart is deployed
    # mysql을 사용하기 위해 false로 수정.
    deployPostgres: false

    # DB 종류 ("postgres", "mysql", "mariadb", "h2") 중 하나
    dbVendor: mysql

    # mysql 접속 정보가 저장된 kubernetes secret 이름.
    existingSecret: "my-db-secret"

    # 위 'existingSecret'에 지정한 Secret 중 사용할 값이 저장된 key 값.
    existingSecretKey: password

    # DB 접속 정보
    dbName: db-name
    dbHost: db-host
    dbPort: 3306
    dbUser: db-user

...
```
  - 설치 합니다.

```bash
helm install --name keycloak -f values.yaml codecentric/keycloak
```

  - `권장사항` 설치 후 위 `keycloak.mydomain.com/auth/admin`로 접속 후 `values.yaml`의 username / password에 설정한 비밀번호로 로그인 후 패스워드를 변경 합니다.
    - values.yaml 파일을 저장해두거나 git에 올려 두거나 할 경우 노출 위험이 있습니다.
  - 상세 설정은 keycloak의 공식 문서를 참조
    - https://www.keycloak.org/documentation.html

-----
## RADIUS 서버 설치 및 Keycloak 연동 설정

###### 1. [EAP(확장 가능 인증 프로토콜)](https://ko.wikipedia.org/wiki/%ED%99%95%EC%9E%A5_%EA%B0%80%EB%8A%A5_%EC%9D%B8%EC%A6%9D_%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C)
  - Wifi 접속 인증으로 사용하기 위해서는 RADIUS 서버가 `EAP`를 지원해야 합니다.
    - 단말에서 사용자가 입력한 ID/PW를 RADIUS 서버에서 복호화해서 평문으로 추출 할수 있어야 Keycloak에 로그인 요청을 할 수 있습니다.
    - EAP 규격중 `EAP-TTLS`를 사용하고 2차인증으로 `PAP`를 사용합니다. PEAP, CHAP 등의 방식은 hash를 이용해 handshake하는 방식이라 RADIUS 서버에서 사용자가 입력한 Password를 알아 낼수 없습니다.
  - `Wifi 접속 인증이 필요없고, SSH 접속인증을 Keycloak과 연동하기 RADIUS 서버를 사용하는 경우 EAP 관련 설정은 불필요합니다.`

###### 2.구성
  - [Docker Hub에 공개되어 있는 이미지](https://hub.docker.com/r/freeradius/freeradius-server/)를 기반으로 python 사용이 가능하도록 수정한 이미지를 생성해 사용합니다.
  - FreeRADIUS는 다양한 모듈을 제공합니다. 그중에 [Python 모듈(rlm_python)](https://wiki.freeradius.org/modules/Rlm_python)을 이용해 keycloak과 연동합니다. 
    - ~~많은 모듈을 제공하는데 Keycloak 모듈은 없네요 ㅠ_ㅠ~~
    - [Perl 모듈(rlm_perl)](https://wiki.freeradius.org/modules/Rlm_perl)도 있습니다.(전 Perl을 배워 본적이 없어서...)
    - REST 모듈(rlm_rest)로도 할 수 있을거 같은데 keycloak 연동시에 API 요청을 여러번 해야 해서 이걸로 하면 처리가 힘들 수 있을거 같아서 python 모듈로 방향을 잡았습니다.
  - 처리 흐름은 아래와 같습니다.

<div class="mermaid">
graph LR;
  CLIENT[시작: Client] -->|1. RADIUS 요청| FREE_RADIUS(FreeRADIUS)
  subgraph Radius-Server
    FREE_RADIUS -->|2| PYTHON_MODULE(Python모듈)
  end
    PYTHON_MODULE -->|3. 권한확인 요청| KEYCLOAK[Keycloak]
    KEYCLOAK -->|4. 권한확인 응답| PYTHON_MODULE
    PYTHON_MODULE -->|5| FREE_RADIUS
    FREE_RADIUS -->|6. RADIUS 응답| CLIENT
</div>

###### 3. 설치시 주의사항(Kubernetes 사용시)
  - **Radius는 `UDP` Port2개를 사용합니다.(default: 1812, 1813)**
  - ~~`UDP`는 AWS에서 Load Balancer를 사용할 수 없습니다.~~ [얼마 전부터 지원시작](https://aws.amazon.com/ko/blogs/aws/new-udp-load-balancing-for-network-load-balancer/)
  - Kubernetes에 설치시 ingress의 유형에 따라 UDP를 지원하지 않을 수 있습니다.
  - 그냥 NodePort로 노출 시키고, Route53에서 Kubernetes에 사용된 각 Node(EC2 Instance)의 Public IP로 Round Robin으로 분산 시켜도 됩니다.[(참고)](https://docs.aws.amazon.com/ko_kr/Route53/latest/DeveloperGuide/routing-policy.html)
  - Kubernetes에서 NodePort를 사용 할 경우 사용가능한 Port Range는 `30000-32767` 입니다.(Radius의 기본 Port번호 사용 불가)

###### 4. FreeRADIUS 설정 및 Docker Image 생성
  - [Docker Hub에 공개되어 있는 이미지](https://hub.docker.com/r/freeradius/freeradius-server/)를 그대로 이용하면 좋지만... 기본 이미지는 `python`을 지원하지 않습니다. 
  - Python 모듈에서 암복호화 기능도 사용할 예정이므로 [pycrypto](https://pypi.org/project/pycrypto/)도 필요합니다.
  - FreeRADIUS 설정 파일들은 굳이 Docker Image에 넣지 않고 실행시 mount 해도 상관없습니다만, 저는 Docker Image에 설정파일을 수정해서 포함 시켰습니다.
  - FreeRADIUS에서 사용 할 인증서 파일도 생성해서 이미지에 포함시킵니다.(이것도.. 실행시 mount 해도 무방합니다.)

###### 5. FreeRADIUS를 기반으로 커스텀 Docker Image 만들기
**step1. 설정 디렉토리 가져오기.**
  - 먼저 [Docker Hub에 공개되어 있는 이미지](https://hub.docker.com/r/freeradius/freeradius-server/)를 실행시킨 후 설정 디렉토리(`/etc/freeradius`)를 추출 합니다.
  - 추출한 설정 파일을 적절히 수정하고, 수정한 파일을 적용해서 Docker Image를 새롭게 생성할 예정입니다.

```bash
$ docker run -d --name freeradius-server freeradius/freeradius-server
$ docker cp freeradius-server:/etc/freeradius freeradius-config
$ docker kill freeradius-server
```

  - 설정 디렉토리 구조(수정 불필요 파일은 생략..)

```
.
├── certs
│   ├── Makefile
│   ├── README
│   ├── ca.cnf
│   ├── client.cnf
│   ├── dh
│   ├── inner-server.cnf
│   ├── passwords.mk
│   └── server.cnf
├── clients.conf
├── mods-available
│   ├── ...
│   └── python
├── mods-config
│   └── python
│       ├── example.py
│       └── radiusd.py
├── mods-enabled
│   ├── ...
│   └── utf8
├── radiusd.conf
├── sites-enabled
│   ├── default
│   └── inner-tunnel
├── templates.conf
├── trigger.conf
└── users
```

**step 2. 인증서 생성(Wifi 인증)**
  - `Wifi 인증`에 `EAP-TLS`, `EAP-TTLS` 등을 사용해야 할 경우 필요합니다. 아닐 경우 다음 단게로...
  - 추출한 설정 디렉토리중 [`certs/README`](https://github.com/redBorder/freeradius/blob/master/raddb/certs/README)를 참조해서 진행합니다.
  - 추가로 [이곳](https://wiki.alpinelinux.org/wiki/FreeRadius_EAP-TLS_configuration)도 참조하시면 좋습니다.
  - 먼저 기존 파일을 제거합니다. (`아래 작업들은 cert 디렉토리에서 수행합니다.`)
  
  ```bash
  $ rm -f *.pem *.der *.csr *.crt *.key *.p12 serial* index.txt*
  ```
  
  - `ca.cnf`, `server.cnf`, `client.cnf`를 열고 수정합니다.(저는 아래 항목들을 수정했습니다.)
    - `default_days`: 인증서 유효기간
    - `countryName ~ commonName`: 인증서에 포함시킬 정보
    - `input_password & output_password`: 인증서 암호

  ```bash
  $ make ca.pem
  $ make ca.der
  $ make server.pem
  $ make server.csr
  $ make client.pem
  $ openssl dhparam -check -text -5 4096 -out dh   # 오래 걸릴 수 있음... 몇분~몇십분
  ```

**step 3. FreeRADIUS 기본 설정**
  - `...` 부분은 기본 설정 유지한 부분.
  - client.conf
    - `SHARED_SECRET` 실행시 환경변수로 지정합니다.

```properties
client localhost {
  # ...
	secret = $ENV{SHARED_SECRET}
  # ...
}
```

  - sites-enabled/default
    - 기본 리퀘스트 처리를 위한 설정.
    - eap 처리 설정 + authorize와 authenticate 단계에서 python 모듈을 사용하도록 추가
    - authorize 단계에서 실행된 python script에서 `Auth-Type=KEYCLOAK`를 설정 할 예정.

```properties
server default {

    authorize {
        # ...
        eap {
            ok = return
            updated = return
        }
        # ...
        python
    }

    authenticate {
        # ...
        Auth-Type KEYCLOAK {
            python
        }
        # ...
        eap
    }
    
# ...
}

```

  - sites-enabled/inner-tunnel
    - `mods-enabled/eap`에서 EAP-TTLS의 `virtual-server`로 inner-tunnel이 설정됨.
    - 들어온 RADIUS요청이 EAP-TTLS일 경우 `sites-enabled/default`에서 eap 관련 처리가 되고 inner-tunnel로 전달 됨.

```properties
server inner-tunnel {
    authorize {
        # ...
        pap
        python
    }

    authenticate {
        Auth-Type KEYCLOAK {
            python
        }
        # ...
        eap
    }
}
```

  - mods-enabled/eap
    - `default_eap_type`은 `ttls(EAP-TTLS)`로 지정.
    - tls, ttls 관련 설정을 빼고 다른 EAP Type 들은 주석처리.
    - `private_key_password`는 인증서 생성시에 만들때 쓴 암호 입력.

```properties
eap {
  # ...
	default_eap_type = ttls
	ignore_unknown_eap_types = yes

	#md5 {
	#}

	#pwd {
	#}

	#leap {
	#}

	#gtc {
	#}

	tls-config tls-common {
		private_key_password = <private_key_password>
      # ...
	}


	tls {
    # ...
		tls = tls-common
    # ...
	}

	ttls {
    # ...
		tls = tls-common
    # ...
		virtual_server = "inner-tunnel"
    # ...
	}

	#peap {
	#}

	#mschapv2 {
	#}

	#fast {
	#}
}
```

  - mods-enabled/python
    - python 모듈 실행 설정
    - `python_path`에 `:`로 구분해서 참조 할 모듈의 path를 지정.
    - command line에서 python을 실행하는 경우 기본 python module 및 추가 설치된 module 들의 path가 기본 설정된 채로 실행되지만 FreeRADIUS에서 python을 실행해 줄때는 그렇지 않으므로 스크립트에서 사용 할(import 하는..) 모듈 들이 있는 path를 추가해 주어야함.
    - Dockerfile에서 이미지 생성시 추가하는 모듈들의 docker image 내 경로를 확인해서 추가해 주어야함.
    - 아래는 `pycrypto`가 설치된 경로를 추가한 설정임. (아래 Docker file 참조)

```properties
python {
	python_path="/usr/lib/python2.7/:/usr/local/lib/python2.7/dist-packages:${modconfdir}/python/:/usr/lib/python2.7/lib-dynload/"
	module = keycloak
	pass_all_vps = no
	pass_all_vps_dict = yes

  # ...

	mod_authorize = ${.module}
	func_authorize = authorize

	mod_authenticate = ${.module}
	func_authenticate = authenticate

  # ... 
}
```

  - mods-config/python/keycloak.py
    - 원래 해당 디렉토리에는 `radiusd.py` / `example.py` 두개 파일이 있습니다.
    - `radiusd.py`는 반환값 상수 및 로그 출력을 위한 함수가 선언되어 있습니다.(자동생성된 파일로 수정해도 반영되지 않습니다.)
    - `example.py` -> `keycloak.py`로 변경(위 mods-enabled/python 파일에서도 미리 변경해두었음.) 
    - 각 단계별 함수에 넘어오는 `p`는 `tuple`이며, 일반적으로 `(('User-Name', 'dorma'), ('User-Password', 'password'), ('NAS-Identifier', 'client-identifier'), ...)`으로 전달 됩니다.
    - 인증 방식이 `PAP`가 아닐 경우 `User-Password`는 전달되지 않습니다.
    - `authorize`단계에서는 username / password가 있는 경우 `Auth-Type=KEYCLOAK`으로 설정하며, `authenticate`단계에서 실제 인증을 처리. 
      - sites-enabled 설정의 authenticate에서 Auth-Type 분기처리가 되어있어서 username / password가 없으면 authenticate 함수는 호출되지 않음.
    - 실제 인증처리 로직은 [keycloak API](https://www.keycloak.org/docs-api/6.0/rest-api/index.html) 참고 하셔서 구현하시면 됩니다.
      - 꼭 keycloak과 연동 안하고 python으로 어떤 처리든 하고 결과만 반환하면 되니 원하는 서버 혹은 DB 베이스로 인증처리를 하시면 됩니다.

```python
#! /usr/bin/env python2

import radiusd
...

def authorize(p):
  username = getUserName(p)
  password = getUserPassword(p)
  if username and password:
    # tuple로 반환시 `인증결과, response, FreeRADIUS 내부 처리 변수` 순으로 반환 가능
    return (
      radiusd.RLM_MODULE_UPDATED,
      (),
      (('Auth-Type', "KEYCLOAK"),)
    )
  else :
    # 처리 결과 반환
    return radius.RLM_MODULE_REJECT

def authenticate(p):
  username = getUserName(p)
  password = getUserPassword(p)
  nasId = getNasIdentifier(p)
  
  # ...<인증 처리>...

  if authResult :
    return radiusd.RLM_MODULE_OK
  else :
    return radiusd.RLM_MODULE_REJECT

...

def getNasIdentifier(p) :
  for key, val in p :
    if key.upper() == "NAS-Identifier".upper():
      return val
  return ""

def getUserName(p) :
  for key, val in p :
    if key.upper() == "User-Name".upper():
      return val
  return ""

def getUserPassword(p) :
  for key, val in p :
    if key.upper() == "User-Password".upper():
      return val
  return ""
 ```

**step 4. Docker Image 생성**
  - [Docker Hub에 공개되어 있는 이미지](https://hub.docker.com/r/freeradius/freeradius-server/)를 베이스.
    1. python 설치
    2. python 추가 모듈 설치(아래 예시는 [pycrypto](https://pypi.org/project/pycrypto/), pycrypto는 설치시 빌드가 필요해서 build-essential도 설치했다가 설치 후 제거)
    3. 위에서 수정한 설정파일들을 설정 디렉토리(`/etc/raddb`) 아래에 덮어씀.

```docker
FROM freeradius/freeradius-server:3.0.17

RUN \
  apt-get update --force-yes -y && \
  apt-get install -o Dpkg::Options::="--force-confold" --force-yes -y python python-dev curl build-essential

RUN curl -o pycrypto-2.6.1.tar.gz https://ftp.dlitz.net/pub/dlitz/crypto/pycrypto/pycrypto-2.6.1.tar.gz
RUN tar -xzf pycrypto-2.6.1.tar.gz
RUN rm -rf pycrypto-2.6.1.tar.gz
WORKDIR /pycrypto-2.6.1
RUN python setup.py install
WORKDIR /
RUN rm -rf pycrypto-2.6.1

RUN apt-get remove --force-yes -y --auto-remove build-essential
RUN apt-get purge --force-yes -y --auto-remove build-essential

COPY ./freeradius-config/radiusd.conf /etc/raddb/radiusd.conf
COPY ./freeradius-config/clients.conf /etc/raddb/clients.conf
COPY ./freeradius-config/sites-enabled/default /etc/raddb/sites-enabled/default
COPY ./freeradius-config/sites-enabled/inner-tunnel /etc/raddb/sites-enabled/inner-tunnel
COPY ./freeradius-config/mods-enabled/eap /etc/raddb/mods-enabled/eap
COPY ./freeradius-config/mods-enabled/python /etc/raddb/mods-enabled/python
COPY ./freeradius-config/certs /etc/raddb/certs
COPY ./freeradius-config/mods-config/python /etc/raddb/mods-config/python

WORKDIR /etc/raddb/mods-config/python
RUN rm -rf *.pyc

WORKDIR /
```

  - docker build & push

```bash
$ docker build -t socar/radius-server:0.9.1 .
$ docker push -t socar/radius-server:0.9.1
```

-----
## 설치
  - `docker-compose.yml` 예시

```yaml
version: "3"

services:
  radius-server:
    container_name: free-radius-server
    image: socarr/radius-server:0.9.1
    env_file:
      - ./config.env
    ports:
      - 31812:1812/udp
      - 31813:1813/udp
    volumes: []
      #- ./logs:/var/log/freeradius
```

  - `config.env` 예시

```bash
SHARED_SECRET=<my-shared-secret>
```

  - 설치 (docker-compose.yml, config.env가 있는 경로에서 실행)

```bash
$ docker-compose up -d
```