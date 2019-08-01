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