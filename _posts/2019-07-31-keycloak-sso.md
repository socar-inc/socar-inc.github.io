---
layout: post
title: "Keycloak를 이용한 SSO 구축"
date: 2019-07-31 17:20:00 +0900
category: security
tags:
    - keycloak
    - sso
---

## 사내 SSO(Single Sign-On) 구축하기
  사내 SSO를 구축을 진행한 내용을 정리한 내용입니다.
  사원은 300+ 규모이며, 회사에서 서비스중인 시스템의 백오피스의 인증을 통합하기 위한 목적으로 시작하였으나 추가적인 요구조건이 추가되어 최종적인 요구사항은 이렇게 정리 되었습니다.

```
  1. Web 사이트 인증.(Spring 베이스 사이트)
  2. Wifi 인증.
  3. SSH 인증(+ Sudo 인증)
  4. 2차 인증 (OTP) 지원
  5. 유연한 권한 설정(Authorization)
```

  여러 유료 솔루션(([Gluu](https://www.gluu.org/), [Google Identity Platform](https://developers.google.com/identity/) 등)도 검토 하였으나 사내 여건상 설치형 + 오픈소스로 무료로 사용 가능한 [Keycloak](https://www.keycloak.org/)으로 결정 됨.
  
  * Windows Active Directory 등도 검토 대상이었으나, Windows 서버 관리에 대한 부담(+ 안익숙함 / 왠지 모를 싫음ㅎㅎ)으로 Pass, OpenLDAP은 내부에 좋지 않은 경험을 가진 분이 계셔서 Pass..

-----
## Keycloak ?
```
  - open source + 설치형.
  - OpenID / SAML 지원
  - 기본 제공 API 이외에 필요시 커스텀 API 추가 가능.
  - RedHat에서 지원하는 프로젝트.
  - keycloak-gatekeeper 등을 이용하면 Web 사이트 권한 처리도 용이.
```

-----
## Keycloak으로 요구사항의 내용을 어떻게 구현 하면 될까?
  1. Web 사이트 인증
    - OpenID / SAML 프로토콜 지원 및 Java client 지원.
    - reverse proxy로 동작하는 [gatekeepr](https://github.com/keycloak/keycloak-gatekeeper)를 이용해 Web Application에서는 인증 / 권한 코드를 작성하지 않고 처리.
    - ~~Java Client 또는 API를 제공하고 있으므로 Web Application에서 직접 처리도 가능.~~
  2. Wifi 인증
    - 사내에 Cisco 라우터를 사용하고 있고, RADIUS 프로토콜을 지원하므로 [RADIUS 서버](https://ko.wikipedia.org/wiki/RADIUS)를 구축하면 가능.
    - [FreeRADIUS](https://freeradius.org)의 [Python 모듈](https://wiki.freeradius.org/modules/Rlm_python)에서 keycloak API를 호출해서 인증을 처리 하면 가능.
    - ~~[Tiny Radius](https://github.com/ctran/TinyRadius)라는 java로 radius 프로토콜을 구현한 라이브러리를 이용해 RADIUS 서버 구현이 가능.~~
  3. SSH 인증(+ Sudo 인증)
    - [sshd_config](https://linux.die.net/man/5/sshd_config)설정에서 UsePAM 옵션을 활성화 시키고 [pam_radius_auth.so](https://github.com/FreeRADIUS/pam_radius)를 사용해 RADIUS 서버를 통해서 인증 가능.
    - ~~[sshd_config](https://linux.die.net/man/5/sshd_config)설정에서 [AuthorizedKeysCommand](https://man.openbsd.org/sshd_config.5#AuthorizedKeysCommand)에 사용할 프로그램을 구현~~
    - ~~Keycloak와 연동 되는 Linux PAM 모듈을 직접 개발. Uber에서 golang으로 작성한 [pam-ussh](https://github.com/uber/pam-ussh)라는 PAM 모듈이 공개 되어 있어 참고해서 개발 가능~~
  4. 2차 인증 (OTP) 지원
    - 지원됨.
    - Wifi 인증시에는 사용하지 않을 예정이며, SSH(+ sudo) 인증시에는 [Access-Challenge + Reply-Message](https://www.iana.org/assignments/radius-types/radius-types.xhtml)를 회신하면 [pam_radius_auth.so](https://github.com/FreeRADIUS/pam_radius)에서 사용자에게 추가 입력을 받을 수 있음.
  5. 유연한 권한 설정(Authorization)
    - keycloak의 권한 설정은 상당히(~~과하게~~) 유연함.
  
-----
### ~~줄이 그어진~~항목들에 대해서
- **`시행착오를 겪고 drop 된 항목들입니다`** 만... 용도에 따라서는 시도해 볼 수 있는 방법들 입니다.
- Wifi 인증을 위해서는 RADIUS서버가 [EAP](https://ko.wikipedia.org/wiki/%ED%99%95%EC%9E%A5_%EA%B0%80%EB%8A%A5_%EC%9D%B8%EC%A6%9D_%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C)를 지원해야 하지만 Tiny Radius의 경우 지원하지 않습니다. `하지만, SSH(+ sudo) 인증을 위해 pam_radius_auth.so만 사용할 경우 이 방법도 사용 가능 합니다.`
- SSH 인증을 위해 `AuthorizedKeysCommand` 설정을 이용하는 방법은 sudo 인증에는 적용이 불가능하고, keycloak의 사용자 정보에 인증을 위한 Public Key를 사용자 Attribute에 저장해야 합니다.(keycloak의 User Attribute를 기본 옵션으로 설치시 varchar(255)라서 DB altering 작업이 필요합니다.)
- Keycloak 용의 PAM 작성은 golang를 알면 [pam-ussh](https://github.com/uber/pam-ussh)을 참고해서 작성 해볼만 합니다만, [ChallengeResponseAuthentication](https://man.openbsd.org/sshd_config.5#ChallengeResponseAuthentication)을 사용 할 경우 PAM 모듈에서 API 호출을 하면 PAM 모듈이 멈춰 버리는 현상 발생합니다. [SSH와 PAM 연동은 알려진 이슈](https://www.dtucker.net/pam/)가 좀 있습니다. 이슈와 연관된 문제인지는 확실치 않지만 pam-ussh를 fork해서 구현했더니 네트워크 통신 혹은 sleep 함수를 호출 하기만 해도 PAM 모듈이 멈추 현상이 발생했습니다. `ChallengeResponseAuthentication` 옵션을 끄고 `UsePAM + PasswordAuthentication` 조합을 활성화하면 커스텀 PAM 모듈에서 Keycloak API를 호출해도 멈추지 않습니다. 하지만 이 경우 사용자에게 OTP 입력을 위한 추가 입력 요청을 할 수 없습니다. 결론은 `SSH인증시 ID + PW / sudo 인증시 ID + PW + OTP로 인증 처리 가능`합니다.