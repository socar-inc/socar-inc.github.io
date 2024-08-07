---
layout: post
title: "누가 내 쿠키를 먹었을까?"
subtitle: "iOS 웹뷰 환경에서 세션쿠키가 소실되는 이유와 해결방법을 알아봅니다."
date: 2024-07-16 07:00:00 +0900
category: dev
background: "/img/ios-webView-cookie/backgroundImage.jpg"
author: rio
comments: true
tags:
  - fe
  - frontend
  - cookie
  - iOS webView cookie
---

# 누가 내 쿠키를 먹었을까?

왜 iOS 웹뷰의 세션쿠키가 사라졌을까?

안녕하세요. 쏘카에서 프론트엔드 개발을 담당하고 있는 리우입니다.<br>
iOS 웹뷰에서만 세션쿠키가 갑자기 소실되는 현상을 마주한 적이 있나요? <br>
해당 문제가 무엇이고, 왜 발생하고, 어떻게 해결했는지 실제로 겪은 사례를 통해 설명드리겠습니다.

> - 여기서 세션쿠키는 Expires 또는 Max-Age가 없는 쿠키를 의미합니다.
> - 세션쿠키는 브라우저를 닫는 순간 삭제됩니다.

![sessionCookie](/img/ios-webView-cookie/sessionCookie.png)

<p style='text-align: center; color: #646f7c;'>개발자 도구→애플리케이션→쿠키탭</p>

해당 내용은 다음과 같은 분들이 읽으면 좋습니다.

> - 모든 프론트엔드 개발자, 웹뷰 환경에서 개발하는 네이티브 개발자
> - iOS 웹뷰에서 쿠키 관련 이슈를 겪고 있는 분

## 상황

여느 때처럼 쏘카 서비스를 개발하는데 Tech Lead께서 저희가 개발했던 서비스에서 에러가 발생한다고 말해주셨습니다.

어떤 상황에서 발생했는지 여쭤보니 쏘카 앱 내의 웹뷰 서비스를 이용한 후, 앱을 백그라운드 상태에 두고 한참 후에 다시 열었을 때 에러가 발생한다고 하셨습니다.<br>

<p align="center">
<img src="/img/ios-webView-cookie/expired.gif" width="50%" hight="50%"/>
</p>
<p style='text-align: center; color: #646f7c;'> 웹뷰 백그라운드 → 특정 시간 후 포어그라운드 → 웹뷰 리프레시 → 서버 호출 시 에러</p>
<p style='text-align: center; color: #646f7c;'>영상 제공을 허락해주신 iOS 미남 3인방 감사합니다🙇‍♂️</p>
<br>

처음에는 단순히 서버 문제로 생각해서 담당 개발자의 도움을 받아 이슈 트래킹 툴을 통해 분석했습니다.
쏘카앱에서는 웹뷰를 보여줄 때 사용자 정보를 전달하고 웹은 그 정보를 쿠키에 저장한 후 서버에 요청하고 있는데, 분석 결과 웹의 쿠키에 사용자 기종과 앱의 버전 정보가 빠진채로 요청되고 있었습니다.

> - Q: 앱의 버전 정보를 쿠키에 저장하는 이유는 무엇인가요?
> - A: 앱 버전에 따라 서버에서 다른 로직을 처리하기 위함입니다.

세션 쿠키가 왜 손실되는지 여러 가설을 세워 다양한 기종에서 테스트 했고 다음의 결론을 얻었습니다.

## 문제 원인

- **iOS 앱 자체의 메모리 정리 시점에 백그라운드에 존재하는 웹의 세션쿠키를 소실 시킴**
  - `iOS Safari에서도 동일하게 세션쿠키가 소실`되는 것을 확인함

> - iOS 모든 버전에서는 테스트해보지는 못했지만, iOS 15버전 이상에서는 발생합니다.

`즉, iOS 앱에서 메모리가 많이 사용되는 경우 iOS 앱이 메모리 정리를 함 → 이 과정에서 백그라운드에 있는 웹뷰에 저장된 세션쿠키를 소실시킴`

### 버그 리포트

관련 이슈와 동일한 버그 리포트도 첨부합니다.

- [developer.apple](https://forums.developer.apple.com/forums/thread/745912)
- [react-native-webview](https://github.com/react-native-webview/react-native-webview/issues/2228#issuecomment-2170404006)
- [iOS와 네이티브 및 WebView 세션 동기화](https://medium.com/axel-springer-tech/synchronization-of-native-and-webview-sessions-with-ios-9fe2199b44c9)

## 해결 방법

쿠키에 저장 시에 세션쿠키가 아닌 지속쿠키(Expires/Max-Age가 있는 쿠키)로 이용할 수 있도록 기간을 정의해줍니다. 이렇게 해주면 iOS가 메모리 정리되는 시점에 지속쿠키는 유지해 주기 때문에 문제없이 사용할 수 있습니다.

![cookie](/img/ios-webView-cookie/cookie.png)

<p style='text-align: center; color: #646f7c;'>해결방법은 간단하죠?</p>
<br>

<p align="center">
<img src="/img/ios-webView-cookie/maintenance.gif" width="50%" hight="50%"/>
</p>
<p style='text-align: center; color: #646f7c;'>포어그라운드 -> 웹뷰 리프레시 -> 쿠키 유지 -> 서버 호출 시 정상적으로 동작</p>

영상을 보시면 알 수 있듯이 포어그라운드로 돌아왔을 때 UI가 잠시 새로고침 됩니다. 이후 해당 페이지가 렌더링 되고 서버 요청 시 정상적으로 동작합니다. 즉 유실되던 쿠키가 잘 유지됨을 확인할 수 있습니다.

해결 후 이슈 트래킹 분석 툴을 통해 전 후 차이를 비교해봤습니다. (\*iOS 기준)
<br>
AS-IS : 하루 평균 약 246건 이슈 발생 -> TO-BE : 하루 평균 약 0.85건 발생

**하루 평균 `약 99.66%의 이슈가 감소`한 것을 알 수 있습니다.**

엄청난 효과이지 않나요? 이제 쏘카 웹 서비스를 한층 더 안정성 있게 제공할 수 있게 됐습니다!!
![issue](/img/ios-webView-cookie/issue.jpg)

## 한 줄 요약

**`iOS 앱은 백그라운드에서 메모리가 정리될 때 웹의 세션 쿠키가 손실된다. 이를 막기 위해서는 지속 쿠키로 이용되도록 기간을 정의해주면 된다.`**

## 후기

해결 방법은 간단했지만 원인을 분석하고 가설을 도출한 후 테스트 진행하는 과정에서 많은 시간이 소요되었습니다. 그러나 이 과정을 통해 문제의 근본적인 원인을 찾아내고 해결함으로써 개발자로서 한층 더 성장할 수 있었습니다. 또한, iOS 공식 문서에 해당 이슈에 대한 해결 방법을 제안해 보는 귀중한 경험도 해볼 수 있었습니다. (\*위에 첨부한 [버그 리포트](#버그-리포트) 참고)

특히, 이 문제는 쏘카의 모든 웹 서비스에 영향을 미쳤기 때문에 해결함으로써 큰 파급 효과가 있었습니다. 이로 인해 쏘카 서비스를 더 안정적으로 제공할 수 있어 뿌듯한 경험이었습니다.

마지막으로 해당 이슈는 오래전부터 존재했지만 비중이 낮아 주목받지 못했습니다. 이 문제를 발견하고 이슈화해주신 팀장님과 내부 제보자들, 문제 해결에 도움을 주신 동료들, 네이티브 개발자분들께 감사드립니다.
