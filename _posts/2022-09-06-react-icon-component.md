---
layout: post
title: "React Custom Icon Component 개발기"
subtitle: "아이콘 등록 프로세스 효율화를 통한 개발자 경험(Developer Experience) 개선"
date: 2022-09-06 09:00:00 +0900
category: dev
background: "/img/icon-component/icons.png"
author: sienna, blanc
comments: true
tags:
  - web
  - frontend
  - react
  - design_system
---

안녕하세요. 쏘카 웹 프론트엔드 개발자 시에나, 블랑입니다.

이 글에 다룰 내용은 쏘카에서 공통적으로 사용하는 **React Icon Component 개발 과정**입니다. 아이콘 컴포넌트를 어떻게 개발했으며 어떤 점들을 개선했는지에 대해서 중점적으로 다룰 예정입니다.

다음과 같은 분들이 읽어보시면 좋을 것 같습니다.

- 웹 프론트엔드 개발자, 웹 디자이너
- 웹 개발에 관심이 있으신 분
- 쏘카 웹 프론트엔드 팀이 궁금하신 분

---

## 목차

- [1. React Icon Component란?](#1.-React-Icon-Component란?)
- [2. React Icon Component 개발의 필요성](#2.-React-Icon-Component-개발의-필요성)
  - [2.1 기존 방식에 대해 알아보자](#2.1-기존-방식에-대해-알아보자)
  - [2.2 기존 방식의 문제점을 알아보자](#2.2-기존-방식의-문제점을-알아보자)
  - [2.3 이렇게 개선해 보자](#2.3-이렇게-개선해 보자)
- [3. Icon Component 개발 과정](#3.-Icon-Component-개발-과정)
  - [3.1 아이콘 파일로 사용되는 SVG를 알아보자](#3.1-아이콘-파일로-사용되는-SVG를-알아보자)
  - [3.2 다양한 Element 도형을 Path Element로 변경해 보자](#3.2-다양한-Element-도형을-Path-Element로-변경해 보자)
  - [3.3 컴포넌트를 만들어서 적용해 보자](#3.3-컴포넌트를-만들어서-적용해 보자)
  - [3.4 타입을 적용해 보자](#3.4-타입을-적용해 보자)
  - [3.5 스토리북에 적용해서 확인해 보자](#3.5-스토리북에-적용해서-확인해 보자)
- [4. Icon Component 등록 과정](#4.-Icon-Component-등록-과정)
  - [4.1 아이콘 등록을 해보자](#4.1-아이콘-등록을-해보자)
  - [4.2 아이콘 등록 방식을 자동화해 보자](#4.2-아이콘-등록-방식을-자동화-해보자)
- [5. 문제점 해결](#5.-문제점-해결)
- [6. 개선할 점](#6.-개선할-점)
- [7. 마무리](#7.-마무리)

---

## 1. React Icon Component란? <a name="1.-React-Icon-Component란?" />

React에서 Component는 재사용이 가능한 UI의 단위를 말합니다. 즉 React Icon Component는 React를 활용하여 아이콘 재사용을 용이하게 만들어주는 UI 단위를 말합니다. 

## 2. React Icon Component 개발의 필요성 <a name="2.-React-Icon-Component-개발의-필요성" />

디자인 시스템이란 재사용 가능한 UI 요소들을 미리 정의하여 일관된 디자인을 할 수 있도록 도와주는 규칙을 말합니다.

쏘카의 디자인 시스템은 디자이너, IOS 개발자, Android 개발자, 웹 프론트엔드 개발자가 협업하여 구축했습니다. 사용자가 일관된 UI/UX를 경험할 수 있도록 IOS, Android, 웹에서 모두 같은 형태의 컴포넌트들을 개발하여 사용합니다. 쏘카의 디자인 시스템에 대한 자세한 내용은 프로덕트 디자인 팀이 작성해 주신 *[<U>쏘카의 디자인 시스템 맛보기</U>](/design/2020/06/23/socar-design-system-01.html)* 를 참고해 주세요. 

이번 글에서 다룰 아이콘 또한 디자인 시스템에 정의되어 있습니다. 쏘카의 아이콘은 쏘카 앱 내 곳곳에서 찾아볼 수 있고 [쏘카 플랜](https://plan.socar.kr/)에서도 사용되고 있습니다.

<div style="display: flex; justify-content: space-between;">
    <img src="/img/icon-component/infoIcon.jpg" width="30%">
    <img src="/img/icon-component/chevronIcon.jpg" width="30%">
    <img src="/img/icon-component/socarplanIcons.jpg" width="30%">
</div>

### 2.1 기존 방식에 대해 알아보자 <a name="2.1-기존-방식에-대해-알아보자" />

저희는 기존에 아이콘을 웹폰트로 변환하여 사용하고 있었습니다. 해당 방식으로도 아이콘을 등록하고 사용하는 데에는 큰 무리가 없지만, 디자인 시스템은 여러 개발자가 공동으로 사용하기 때문에 개발자 경험(Developer Experience, DX) 또한 중요합니다. 기존의 방식은 개발자 경험 측면에서 불편한 점이 있어 아이콘 방식 개선에 대한 필요성을 느끼게 되었습니다.

구체적으로 기존의 방식에 어떤 문제점이 있었는지 말씀드리겠습니다.

1. SVG → 웹폰트 변환

   - [<b>외부 서비스</b>](https://icomoon.io/app)를 이용해서 아이콘 SVG 파일들을 웹폰트로 변환합니다.

   ![image](/img/icon-component/svgToFont.png)

2. 생성된 파일 등록

   - 웹폰트로 변환하면 폰트 파일과 CSS 파일이 생성됩니다.

   ![image](/img/icon-component/generateIconResult.png)

   ![image](/img/icon-component/iconFontFiles.png)

   - 생성된 파일들을 디자인 시스템 저장소에 넣어줍니다. 이때 아이콘 폰트에 원하는 컬러를 주입할 수 있도록 하기 위해 CSS의 color 속성을 제거해 줍니다.

   <div style="display: flex; justify-content: space-between;">
       <img style="width: 60%; display: inline-block;" src="/img/icon-component/cssBefore.png" />
       <img style="width: 35%; display: inline-block;" src="/img/icon-component/cssAfter.png" />
   </div>

3. 디자인 시스템 배포
   - 아이콘 폰트 등록이 완료되면 디자인 시스템의 배포를 진행합니다. 디자인 시스템을 사용 중인 모든 서비스들도 디자인 시스템 버전을 최신화해서 같이 배포를 진행합니다.

4. 아이콘 사용
   - `<i>` 태그에 사용하고자 하는 아이콘의 className을 주입하여 사용합니다.

     ```html
     <i className="ic18_help grey050" />
     ```


### 2.2 기존 방식의 문제점을 알아보자 <a name="2.2-기존-방식의-문제점을-알아보자" />

기존 방식을 사용하면서 겪은 문제점은 다음과 같습니다.

1. 외부 서비스에 의존적이다.

    - 위에서 설명드렸듯이 SVG 파일 -> 웹폰트 변환 과정은 외부 서비스에 전적으로 의존하고 있습니다. 해당 서비스의 기능이 변경되거나 서비스가 종료되면 기존의 방식으로는 아이콘 등록을 못 하게 될 수도 있는 위험성이 있습니다.

2. 휴먼 에러가 발생할 여지가 많다.

   - 아이콘 등록하는 전 과정을 수기로 진행하는 것이 번거롭고 어느 한 과정이라도 누락하거나 실수가 발생하면 아이콘이 제대로 등록되지 않거나 실 서비스 화면에서 아이콘이 깨져 보이게 됩니다.

3. 폰트 방식을 이용하면 발생하는 문제점이 많다.

    - 글꼴 파일을 불러오고 빌드 하는 시간이 소요됩니다.
    - 웹 접근성이 좋지 않습니다.
    - 화면을 확대하면 화질 저하가 발생합니다.
    - 텍스트 기반 CSS 규칙을 따르기 때문에 CSS 컨트롤이 어렵습니다.

### 2.3 이렇게 개선해 보자 <a name="2.3-이렇게-개선해 보자" />

위와 같은 문제점들을 겪으면서 디자인 시스템에 아이콘을 등록하는 방식 자체를 개선해야겠다고 생각했습니다. 개선의 목표는 다음과 같았습니다.

- **아이콘 등록 과정의 간소화**
- **아이콘 등록 방식의 자동화**

위의 목표를 달성하면서도 기존에 사용 중인 아이콘들에 영향을 미치지 않는 방식으로 개선을 시도했습니다.

---

## 3. Icon Component 개발 과정 <a name="3.-Icon-Component-개발-과정" />

### 3.1 아이콘 파일로 사용되는 SVG를 알아보자 <a name="3.1-아이콘-파일로-사용되는-SVG를-알아보자" />

SVG는 2차원 벡터 그래픽을 표현하는 XML 기반의 마크업 언어입니다.

벡터 그래픽이란 수학적 표현을 통해 점, 직선, 곡선을 사용하여 이미지를 만드는 그래픽을 말합니다. 흔히 알고 있는 또 다른 그래픽 파일 포맷으로는 래스터 그래픽, 즉 비트맵으로 PNG, JPEG가 있습니다. 이것은 픽셀로 구성된 이미지로 작은 컬러 사각형인 픽셀이 무수히 많이 모여 만들어지는 그래픽입니다.

![image](/img/icon-component/vector&bitmap.png)*출처: [벡터 그래픽과 래스터 그래픽의 차이 ](https://www.selfmadedesigner.com/what-are-vector-graphics/)*




벡터 그래픽과 래스터 그래픽은 해상도에서 차이점을 보입니다.
픽셀로 이루어진 래스터 그래픽은 벡터 그래픽보다 더 다양한 색상을 표현할 수 있지만 크기를 크게 조절하면 이미지 품질이 저하된다는 단점이 있습니다. 이미지 품질을 높이면 크게 조절이 가능하지만 용량이 커지면서 웹 성능의 영향을 미칠 수 있습니다.

벡터 그래픽은 크기를 크게 조절하더라도 이미지 품질이 저하되지 않고 용량도 래스터 그래픽보다 작다는 장점을 갖습니다. 이에 더하여 애니메이션이 가능하고, 수정이 용이하며, 출력이 빠르기 때문에 **아이콘 파일 포맷**으로 사용되고 있습니다.



### 3.2 다양한 Element 도형을 Path Element로 변경해 보자 <a name="3.2-다양한-Element-도형을-Path-Element로-변경해 보자" />

SVG를 이루는 기본적인 도형의 종류로는 Path, Rect, Circle, Ellipse, Line, Polyline, Polygon Element가 있습니다. 그중에서 Path Element는 **d** Attribute 하나로 정의되고, 선과 곡선, 호 등 다양한 형태를 그릴 수 있는 Element입니다. 여러 개의 직선과 곡선을 합쳐서
**복잡한 도형을 그릴 수 있다는 큰 장점**을 갖고 있습니다.

가장 먼저 Path Element의 장점을 이용해 쏘카가 가진 다양한 형태의 아이콘을 하나의 형태로 바꾸는 스크립트를 작성하였습니다. 

Path Element로 바꾸는 원리를 Rect Element로 작성된 아이콘으로 설명해 드리겠습니다.

```html
<rect x="8" y="8" width="2" height="2" fill="#374553"/>
```

해당 코드는 (8,8)가 사각형 왼쪽 꼭짓점이고 넓이, 높이가 2인 사각형을 그리게 됩니다.

Path Element로 나타내면 아래와 같은 코드가 됩니다.

```html
<path d="M8,8 H10 V10 H8 z">
```

M은 시작점, H는 수평선, V는 수직선을 뜻합니다. 마지막 z는 "Close Path"라는 의미로 다시 시작점으로 선을 그리라는 뜻입니다.
이대로 위 path를 해석해 보면 (8,8)에서 (10,8), (10,10), (8,10)을 찍고 마지막으로 z를 통해 첫 시작 점 (8,8)으로 그리면 2x2인 정사각형 아이콘이 그려집니다. 

<p align="center">
  <img src="/img/icon-component/iconExample.jpg" width="200">
</p>


이런 원리를 식으로 계산하여 Circle, Rectangle Element를 Path Element로 변환하는 스크립트 파일을 작성하였고, 이 결과물로 아이콘 이름을 Key 값으로 갖는 Path JSON 파일이 만들어집니다.

```json
  "ic18_description_dash2": {
    "width": 18,
    "height": 18,
    "path": "M8,8H10V10H8z"
  }
```

### 3.3 컴포넌트를 만들어서 적용해 보자 <a name="3.3-컴포넌트를-만들어서-적용해 보자" />

아이콘의 형태를 동일하게 만들어 하나의 JSON 파일에 모아두었으니 이걸 활용하여 컴포넌트를 만들어 적용해 보겠습니다.

3.2에서 만든 Path JSON 파일을 활용하여 컴포넌트를 만들었습니다. Icon 이름, Icon 색상, Style, ClassName 을 Parameter로 받고 SVG Path로만 이루어진 컴포넌트입니다. 


```jsx
import svgPaths from 'iconDist/svgIcons.json'

const Icon = ({icon, iconColor, style, className}) => {
  const { width, height, path } = svgPaths[icon]

  return (
    <svg
      width={width}
      height={height}
      fillRule="evenodd"
      clipRule="evenodd"
      xmlns="http://www.w3.org/2000/svg"
      style={style}
      className={className}
    >
      <path d={path} />
    </svg>
  )
}
```

Icon Component를 사용하려고 보니 IDE가 제공해 주는 자동완성이 되지 않는 문제점이 있었습니다. 
아이콘 개수와 색상이 수백 개인데 사용할 때마다 일일이 적는 것이 번거로웠습니다.
이때 생각한 것이 아이콘 이름과 색상을 타입으로 지정하면 IDE의 도움을 받을 수 있을 것이라 예상했습니다.

### 3.4 타입을 적용해 보자 <a name="3.4-타입을-적용해 보자" />

아이콘의 이름을 리터럴 타입으로 정의하기 위해서 3.2에서 만든 Path JSON 파일에서 아이콘 이름만 추출하는 작업이 필요합니다.

```json
{
 "ic12_24_chevron_right2": {
    "width": 12,
    "height": 24,
    "path": "M3.4949,4.5051L11.2399,12.2501L3.4949,19.9951L2.505,19.0051L9.261,12.2501L2.505,5.4951L3.4949,4.5051z"
  },
  "ic12_24_divider_vertical": {
    "width": 12,
    "height": 24,
    "path": "M5,5H7V19H5z"
  },
  "ic12_24_divider_vertical_small": {
    "width": 12,
    "height": 24,
    "path": "M5.3,7H6.7V17H5.3z"
  },
  "ic12_24_field_dash": {
    "width": 12,
    "height": 24,
    "path": "M2,11H10V13H2z"
  },
}
```

아이콘 이름이 Key 값으로 정의되어 있기 때문에 Key 값만 가져온다면 손쉽게 타입 정의를 할 수 있을 것이라 생각했습니다.
`Object.key()`를 활용해 아이콘 이름을 가져오는 스크립트를 작성하였고, 실행시키면 아래와 같은 파일이 만들어집니다. 


```jsx
export const ICON_NAME = {
  "gr_icon_credit": "gr_icon_credit",
  "ic12_24_chevron_right2": "ic12_24_chevron_right2",
  "ic12_24_divider_vertical": "ic12_24_divider_vertical",
  "ic12_24_divider_vertical_small": "ic12_24_divider_vertical_small",
  "ic12_24_field_dash": "ic12_24_field_dash",
} as const;

export type IconName = typeof ICON_NAME[keyof typeof ICON_NAME];
```

아이콘 이름을 리터럴 타입으로 정의했듯이 색상도 타입으로 정의하기 위해 색상 이름을 추출하는 작업이 필요합니다. 

색상 코드가 적힌 CSS 파일을 문자열 메서드를 이용하여 색상 이름만 가져와 JSON 파일을 만들어 주고, JSON 파일에 특정 부분만 읽어 색상 이름만 타입으로 정의하는 스크립트를 작성합니다. 

```json
{
  "grey005": "#fefefe",
  "grey010": "#fdfdfd",
  "grey020": "#f8f9fb",
  "grey025": "#f2f4f6",
  "grey030": "#e9ebee",
  "grey040": "#c5c8ce",
}
```
```jsx
export const ICON_COLOR = {
  "grey005": "grey005",
  "grey010": "grey010",
  "grey020": "grey020",
  "grey025": "grey025",
  "grey030": "grey030",
  "grey040": "grey040",
} as const;

export type IconColor = typeof ICON_COLOR[keyof typeof ICON_COLOR];
```

아이콘 이름과 색상을 Type으로 정의하여 적용하니 컴포넌트를 사용할 때 IDE의 자동완성이 뜨는 것을 확인할 수 있었습니다.

<p align="center">
  <img src="/img/icon-component/iconName.png" >
</p>
<p align="center">
  <img src="/img/icon-component/iconColor.png">
</p>

타입까지 적용된 최종 아이콘 컴포넌트는 다음과 같습니다. 

```jsx
import React, { CSSProperties } from 'react'
import svgPaths from 'src/iconDist/svgIcons.json'
import iconPalette from 'src/iconDist/iconPalette.json'
import { IconName, IconColor } from './type'

export interface IconProps {
  icon: IconName
  color: IconColor
  style?: CSSProperties
  className?: string
}

const Icon = ({ icon, color, style, className }: IconProps) => {
  const { width, height, path } = svgPaths[icon]

  return (
    <svg
      width={width}
      height={height}
      fill={iconPalette[color]}
      fillRule="evenodd"
      clipRule="evenodd"
      xmlns="http://www.w3.org/2000/svg"
      style={style}
      className={className}
    >
      <path d={path} />
    </svg>
  )
}

export default Icon
```

<br>

### 3.5 스토리북에 적용해서 확인해 보자 <a name="3.5-스토리북에-적용해서-확인해 보자" />

스토리북은 컴포넌트 단위의 UI 개발 환경을 지원하는 도구입니다. 개발한 아이콘 컴포넌트를 출시하기 전에 미리 디자이너, 개발자가 테스트해서 확인할 수 있습니다.
![image](/img/icon-component/storybook.png)

이렇게 아이콘과 색을 지정해서 출력해 볼 수 있으며, Docs를 확인하여 아이콘 컴포넌트의 사용 방법을 알 수 있으며, 디자이너가 원하는 아이콘이 맞는지 QA도 진행할 수 있습니다.

![image](/img/icon-component/StorybookTest.png)
![image](/img/icon-component/storybookDocs.png)

## 4. Icon Component 등록 과정 <a name="4.-Icon-Component-등록-과정" />

### 4.1 아이콘 등록을 해보자 <a name="4.1-아이콘-등록을-해보자" />
아이콘 컴포넌트를 사용하기 위해서는 SVG 파일의 아이콘이 필요합니다. 피그마에서 아이콘을 SVG 형태로 받아 지정된 폴더에 넣어줍니다.
추가한 아이콘의 형식을 통일시키기 위해 [Path Element로 전환하는 스크립트](#3.2-다양한-Element-도형을-Path-Element로-변경해 보자)를 실행합니다. 
아이콘에 색상을 넣어주기 위해 형식에 맞게 아이콘 색상 코드를 적어 지정된 폴더에 넣어줍니다. 
아이콘 컴포넌트를 IDE의 도움을 받아 사용하려면 타입 정의도 필요하기 때문에 아이콘 이름과 색상 값의 [타입을 추출해 주는 스크립트](#3.4-타입을-적용해 보자)도 실행합니다. 이렇게 하면 원하는 아이콘을 아이콘 컴포넌트를 사용하여 화면에 나타낼 수 있습니다. 

정리하면, 새로운 아이콘 파일과 색상 코드가 적힌 CSS 파일을 지정된 폴더에 넣고 스크립트만 돌리면 등록이 손쉽게 완료됩니다.



### 4.2 아이콘 등록 방식을 자동화해보자 <a name="4.2-아이콘-등록-방식을-자동화-해보자" />
아이콘을 등록할 때마다 스크립트를 실행해야 합니다. 매번 스크립트 파일을 찾아보면서 실행하기엔 불편한 점이 많아 스크립트 자동화를 진행했습니다.
Node.js의 패키지를 관리하는 NPM을 이용하여 스크립트를 명령어 한 줄로 실행할 수 있도록 package.json 파일에 `scripts`를 작성했습니다.

```json
"scripts": {
  "build-icon": "node build/convertSvgPath.js && npm run build-icon-palette && npm run build-icon-type",
}
```

`npm run build-icon`을 터미널에 입력하면 작성한 스크립트를 한 번에 실행시킬 수 있습니다. 



## 5. 문제점 해결 <a name="5.-문제점-해결" />

매번 기존에 있던 아이콘부터 새로운 아이콘까지 전부 넣고 외부 서비스를 사용하여 아이콘 폰트 파일로 만들어야 했었습니다.
아이콘 폰트 파일을 만들면 추가해야 할 파일도 여러 개고, 아이콘 색상을 모든 파일에 직접 추가해야 했었습니다. 
이 과정에서 휴먼 에러도 발생하고 리소스도 많이 필요했습니다. 

이제는 새로운 아이콘만 폴더에 추가하여 스크립트 실행만으로 손쉽게 아이콘을 등록하여 사용할 수 있게 됐습니다. 
더 이상 외부 서비스를 사용하지 않아도 되고 아이콘 색상도 손쉽게 바꿀 수 있게 됐습니다. 
스크립트 실행을 명령어 한 줄로 빌드 하여 아이콘 등록 과정을 자동화하며 사용자의 경험을 개선할 수 있었습니다.  
수동으로 작성했을 때 발생하는 휴먼 에러를 방지할 수 있다는 점에서도 의미 있는 작업이었습니다. 


## 6. 개선할 점 <a name="6.-개선할-점" />

- [SVGR](https://react-svgr.com/docs/webpack/)을 활용한 SVG Path 파싱
  
  - SVG Path를 파싱 하는 코드를 작성했지만, SVG의 형태에 따라 Path를 파싱 하지 못하는 경우가 있습니다.
    
  - SVGR을 활용해서 이런 경우를 방지할 수 있을 것으로 기대합니다.
  
  ![image](/img/icon-component/codeReview.png)

- Figma와 연동

  - 아이콘을 등록하는 방식 중 대부분을 자동화하는데 성공했지만 SVG 파일을 디렉터리에 넣는 과정은 여전히 수기로 진행해야 합니다.
    
  - 디자이너분들이 아이콘 파일을 Figma를 통해 전달해 주시기 때문에, Figma와 연동 혹은 다른 방법을 통해 모든 과정을 자동화할 예정입니다.

---

## 7. 마무리 <a name="7.-마무리" />

쏘카 웹 프론트엔드 팀에서 React Icon Component를 개발한 과정에 대해 알아보았습니다. 외부 서비스는 사용자 경험을 중요시하는 것처럼 회사 내 개발자들이 사용하는 서비스는 개발자 경험이 중요합니다.

컴포넌트는 한번 개발되면 여러 개발자들이 공동으로 사용하기 때문에 개발할 때 '개발자 경험 향상'을 중점으로 두고 진행했습니다. 팀원들이 새로운 아이콘 등록 방식과 아이콘 컴포넌트를 잘 사용하고 있는 걸 보면 목적을 잘 달성했다는 생각이 듭니다.

![image](/img/icon-component/commentLisbon.png)

![image](/img/icon-component/commentDosii.png)

> 내가 개발한 것을 통해서 다른 누군가가 편해진다는 것은 뿌듯한 일입니다.

긴 글 읽어주셔서 감사합니다. 지금까지 쏘카 웹 프론트엔드 팀의 시에나, 블랑이었습니다.
