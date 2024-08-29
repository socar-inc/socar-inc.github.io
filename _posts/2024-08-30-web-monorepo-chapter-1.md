---
layout: post
title:  "쏘카 프론트엔드 모노레포 - Part1. 프로젝트 구성하기"
subtitle: 모노레포 환경에서 프로젝트를 구성하는 방법에 대해 알아봅시다.
date: 2024-08-29 12:00:00 +0900
category: fe
background : "/img/web-monorepo-chapter-1/web-monorepo-bg.jpg"
author: riaco
comments: true
tags:
    - Web
    - FE
    - monorepo
---


# 목차

1. [**시작하며**](#1-시작하며)
2. [**규모가 커지면서 생기는 문제점들**](#2-규모가-커지면서-생기는-문제점들)
3. [**일관성을 위한 모노레포 도입과 고민**](#3-일관성을-위한-모노레포-도입과-고민)
4. [**프로젝트 세팅 비용 단축해보기**](#4-프로젝트-세팅-비용-단축하기)
    
     
    4.1 [기존 멀티레포 환경에서의 프로젝트 세팅](#41-기존-멀티레포-환경에서의-프로젝트-세팅)

    4.2 [모노레포 환경 구성 세팅하기](#42-모노레포-환경-구성-세팅하기)

    4.3 [Code Generator를 활용한 프로젝트 세팅](#43-code-generator를-활용한-프로젝트-세팅)

    4.4 [결론](#44-결론)

5. [**마치며**](#5-마치며)



# 1. 시작하며
 
<aside style="text-align: center; color: #646f7c;">
💡 해당 글은 아래 분들에게 도움이 됩니다

- 모노레포 환경으로 이관/구성하려는 프론트엔드 개발자
- 프로젝트 환경 세팅 자동화를 하여 효율성을 높이고 싶은 개발자
</aside>

안녕하세요 쏘카에서 CommonResource 팀에 소속되어 있는 웹 프론트엔드 개발자 리아코(최민석) 입니다. 

이미 여러 회사에서도 프로젝트를 관리하기 위한 방법으로 모노레포를 도입한 곳이 많을 것 입니다. 쏘카 웹 프론트엔드 규모가 점점 커지면서 (24년 6월 기준) 27명의 프론트엔드 인원과 수십 개의 프로젝트를 효율적으로 관리하기 위해 모노레포를 도입하게 되었습니다. 기존 멀티레포 환경에서 모노레포 환경으로 이관하게 되면서 사용하던 기술 스택에 대한 기준 설정과 프로젝트 일관성을 보장하기 위한 방법을 고민하고 개선했습니다. 이 글을 통해 개선한 경험을 공유해 모노레포 프로젝트 관리 효율을 높이는데 도움이 되면 좋겠습니다.


# 2. 규모가 커지면서 생기는 문제점들

<!-- ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/6f9b88d0-c326-4fee-8f8c-a3d27ae75aab/ee36d591-4f76-43b6-996d-5e90447b7ffe/Untitled.png)

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/6f9b88d0-c326-4fee-8f8c-a3d27ae75aab/a34e7dd1-0ad0-4e36-9f3c-427559d966d7/image.png)

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/6f9b88d0-c326-4fee-8f8c-a3d27ae75aab/01c1d564-8ee1-4a34-acff-fdc9a5006538/image.png)

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/6f9b88d0-c326-4fee-8f8c-a3d27ae75aab/e634f4d1-f753-4a50-b2ef-40d2f1e0250b/image.png) -->

점점 프론트엔드 팀의 규모가 커지고 진행되는 프로젝트가 많아지면서 유지 보수와 코드 중복에 대한 고민이 깊어졌습니다. 특히 디자인 시스템에 정의되지 않은 컴포넌트들이 각 프로젝트마다 중복 구현되는 문제가 두드러졌습니다.

위 이미지에서 볼 수 있듯이, `BottomSheet`라는 유사한 기능의 컴포넌트가 여러 프로젝트에 걸쳐 중복 구현되어 있습니다. 이는 코드 파편화를 야기했고, 일관성 있는 사용자 경험을 제공하는 데 어려움 뿐만 아니라 유지보수에도 어려움을 겪었습니다.

프론트엔드 팀 인원이 30명 가까이 늘어나면서 이러한 문제는 더욱 심화되었고, 이러한 문제들을 해결하기 위해 저희 쏘카 웹 그룹 팀은 모노레포 도입을 결정했습니다.

# 3. 일관성을 위한 모노레포 도입과 고민

모노레포는 이미 많은 회사에서 프로젝트 관리의 일관성을 위해 도입하고 있는 방식입니다. 모노레포를 통해 다음과 같은 이점을 얻고자 했습니다.

```jsx
1. 공통 컴포넌트의 중앙화된 관리
2. 코드 재사용성 증가
3. 일관된 개발 환경 및 툴링
4. 의존성 관리 간소화
```

모노레포를 관리하기 위한 여러 도구들(lerna, nx 등)이 있지만, 저희는 여러 성능과 사용 편의성을 고려하여 turborepo를 기반으로 모노레포 프로젝트를 구성했습니다.

모노레포 도입을 통해 30명이 넘는 프론트엔드 인원과 수 십개의 프로젝트를 관리하여도 동일한 기술 스택과 프로젝트 환경 등, 일관성이라는 장점으로 하나의 사람이 관리하는 것과 같이 관리할 수 있게 되었습니다.

다만 멀티레포에서 사용하던 방식들에 대한 변경 점과 모노레포 프로젝트 관리를 위한 정책들과 도구들이 추가되어야 했습니다. 저희는 먼저 프로젝트 세팅에 대한 고민을 시작했습니다.

# 4. 프로젝트 세팅 비용 단축하기

멀티레포 환경에서 모노레포로 이관하면서 고민했던 문제들과 프로젝트 세팅을 위한 Code Generator에 대해 소개합니다

## 4.1. 기존 멀티레포 환경에서의 프로젝트 세팅

쏘카 웹 프론트엔드 프로젝트 템플릿은 [Github Repository Template](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-template-repository) 기능으로 프로젝트 환경을 2개로 나누어서 사용 중입니다.
<!-- 
![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/6f9b88d0-c326-4fee-8f8c-a3d27ae75aab/74fe5f09-b489-44ed-bf66-b84b69ae76d9/Untitled.png) -->

Multi-Repo 방식의 프로젝트 관리를 하면서 해당 템플릿을 받아오는 스캐폴딩 개발 방식을 도입했습니다.

<aside>
💡   스캐폴딩(Scaffolding) 코드란?
- 새로운 프로젝트나 모듈을 시작할 때 초기 구조와 설정을 자동으로 생성해주는 도구 및 과정

</aside>

이러한 Github Repository Template 기능을 활용하여 다음과 같은 이 점을 얻을 수 있었습니다.

<aside>
💡   1.  멀티레포 방식의 효율적인 프로젝트 세팅 방법

1. 프로젝트 간 일관성 유지
</aside>

이를 통해 개발 프로세스의 효율성을 높이고 코드 품질의 일관성을 유지할 수 있었습니다. 

그러나 모노레포 환경으로 이전하면서 Repository Template을 더 이상 활용할 수 없게 되었습니다. 이를 대체하기 위해 저희는 Code Generator를 구현했고 이에 대해 다음 섹션에서 자세히 설명하겠습니다.


## 4.2. 모노레포 환경 구성 세팅하기

쏘카 프론트엔드 모노레포는 [turborepo](https://turbo.build/repo) 기반으로 모노레포 환경을 구성했습니다. 멀티 레포와 다르게 하나의 Repository로 프로젝트들을 관리하게 되면서 새로운 스캐폴딩 방식 중 하나인 `Code Generator`를 도입하게 되었습니다.

<aside>
💡 Code Generator란?

코드 제너레이터는 프로그래밍 코드를 자동 생성하는 도구나 소프트웨어를 의미합니다. 개발자가 직접 작성하는 대신 특정 규칙 및 템플릿 명세를 기반으로 소스코드를 자동으로 생성하는 프로그램입니다.

</aside>

쏘카 프론트엔드 모노레포 환경에서 생성해야 하는 선정한 제너레이팅되어야 하는 패키지들을 나열해보았습니다.

<!-- ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/6f9b88d0-c326-4fee-8f8c-a3d27ae75aab/50d41432-c60b-4084-b499-330279052922/Untitled.png) -->

**App**

웹 그룹에서 사용하고 있는 next 기반의 프로젝트 세팅 및 프로젝트 구조를 담고 있습니다. 
AWS Amplify 환경 구성으로 세팅 된 CI/CD Workflow, 테스트 환경, 코어라이브러리가 세팅 된 템플릿을 제공합니다.

**socar-design-system**

웹 그룹에서 사용하고 있는 디자인시스템 컴포넌트를 구성할 때 사용합니다. 컴포넌트 코드에 대한 컨벤션과 스타일 파일을 제공합니다.

**utils**

웹 그룹에서 사용하고 있는 공통으로 사용하는 유틸 함수 패키지를 구성할 때 사용합니다. 유틸 기능에 대한 주석 컨벤션과 코드 컨벤션을 제공합니다.

위와 같은 패키지들은 모노레포에서 사용하고 있는 컨벤션이 적용 된 템플릿을 쓸 수 있도록 제공하고 있습니다.


## 4.3. Code Generator를 활용한 프로젝트 세팅

쏘카 모노레포 환경은 위에서 언급한 것과 같이 turborepo로 이루어져 있습니다.  해당 도구에서는 모노레포 패키지를 보다 효율적으로 활용할 수 있는 제너레이팅 스크립트를 제공합니다.

turborepo에서는 제너레이트 도구로 JavaScript로 정의할 수 있는 프롬프트 라이브러리인 [Plop](https://plopjs.com/)을 활용합니다. 이를 통해 새로운 컨트롤러, 구성 요소가 변경되어도 손 쉽게 유지 보수 할 수 있는 스크립트 코드를 작성할 수 있습니다.

다음은 Code Gen 구성을 위한 Plop Config에 대한 예제입니다. 기본적으로 Plop 문법을 따르고 있으며 Generator 할 수 있는 Plop Action들을 정의해 사용할 수 있습니다.

```jsx
// turbo/generators/config.ts
import { type PlopTypes } from '@turbo/gen'
import {
  hooksPlopConfig,
  nextAppPlopConfig,
  socarDesignSystemPlopConfig,
  utilsPlopConfig,
} from '../plops'
import { toUpperCase } from './helpers'

export default function generator(plop: PlopTypes.NodePlopAPI) {
  plop.setGenerator('next', nextAppPlopConfig)
  plop.setGenerator('socar-design-system', socarDesignSystemPlopConfig)
  plop.setGenerator('utils', utilsPlopConfig)
  plop.setGenerator('hooks', hooksPlopConfig)

  plop.setHelper('uppercase', toUpperCase)
}
```

위와 같이 PlopConfig를 역할에 맞도록 파일을 생성하여 유지 관리하고 있습니다.

Plop의 장점으로는 제공하는 add, modify와 같은 정의 된 예약어가 아닌, 확장 된 설정을 할 수 있습니다. 저희는 커스텀 헬퍼 함수와 액션을 추가하여 생성되는 파일에 대한 `ESLint`의 `fix` 옵션을 적용한 프로세스를 설정했습니다. 

이를 통해 추가 구현한  `socar-design-system` 생성 스크립트입니다.

```jsx
import { type PlopTypes } from '@turbo/gen'
import path from 'path'
import { lintAndFixFiles } from '../utils'

const SOCAR_DESIGN_SYSTEM_SRC_PATH = 'packages/socar-design-system/src' as const
export const socarDesignSystemPlopConfig: PlopTypes.PlopGeneratorConfig = {
  description: "'packages/socar-design-system' 공통 UI 컴포넌트를 생성합니다.'",
  prompts: [
    {
      type: 'input',
      name: 'file',
      message:
        '작업 할 컴포넌트 이름을 입력해주세요. (ex: button 입력 시 Button 컴포넌트가 생성됩니다.)',
	    ...
    },
  ],
  actions: [
    {
      type: 'add',
      path: `${SOCAR_DESIGN_SYSTEM_SRC_PATH}/components/{{ properCase file }}/styles`,
      templateFile: ...,
    },
    {
      type: 'add',
      path: `${SOCAR_DESIGN_SYSTEM_SRC_PATH}/components/{{ properCase file }}/index.tsx`,
      templateFile: ...,
    },
    {
      type: 'modify',
      pattern: /(\n\s*)$/,
      path: `${SOCAR_DESIGN_SYSTEM_SRC_PATH}/components/index.ts`,
      template: ...,
    },
	  async () => {
      const filePath = path.resolve(__dirname, "배럴 파일 경로")
      return await lintAndFixFiles(filePath)
    },
  ],
}
```


## 4.4. 결론

저희는 위와 같이 Code Generator를 도입하여 다음과 같은 이점을 얻을 수 있었습니다.

**코드 일관성 유지**

- 모든 새로운 컴포넌트나 유틸리티가 동일한 구조와 스타일로 생성되어 코드베이스 전체의 일관성을 유지할 수 있습니다.

**시간 절약**

- 반복적인 보일러플레이트 코드 작성을 자동화함으로써 개발자의 시간을 절약할 수 있습니다. 
새로운 프로젝트 생성/세팅 시간이 기존 대비 80퍼 이상 감소했습니다.

**오류 감소**

- 수동으로 파일을 생성하고 설정하는 과정에서 발생할 수 있는 인적 오류를 줄일 수 있습니다.

**쉬운 온보딩**

- 새로운 팀원들이 프로젝트 구조와 컨벤션을 빠르게 이해하고 적용할 수 있습니다.

위와 같은 개선은 개발자들의 생산성 향상과 코드 품질 개선으로 이어지고 더 나은 제품을 더 빠르게 제공할 수 있게 해주었고 긍정적인 결과를 보여주었습니다.


## 5. 마치며

모노레포 도입 이전부터 프로젝트 구성에는 기초 지식이 필요했습니다. 이러한 진입 장벽을 낮추고 개발자들의 부담을 줄이고자 우리는 앞서 소개한 도구들을 도입하게 되었습니다. 개발 팀이 성장하고 프로젝트 규모가 커짐에 따라, 이러한 개선 노력들이 큰 효과를 발휘하기 시작했습니다. 결과적으로 우리는 개발자들이 복잡한 환경 설정보다는 실제 서비스 로직에 더 집중할 수 있는 환경을 만들어가고 있습니다.

CommonResource 팀은 끊임없이 개발자 생산성 향상을 위한 방법을 모색하고 있습니다. 단순히 코드 작성의 효율성을 높이는 것을 넘어, 마치 한 사람이 개발한 것처럼 일관된 코드베이스를 유지하기 위해 다양한 거버넌스(Governance) 정책을 수립하고 있습니다. 또한 개발자들의 일상적인 작업을 돕는 여러 도구들을 지속적으로 개발하고 있습니다.

긴 글을 끝까지 읽어주셔서 감사합니다. 이 글이 여러분의 개발 프로세스 개선에 조금이나마 도움이 되었기를 바랍니다. 앞으로도 저희 팀의 경험과 인사이트를 계속해서 공유하겠습니다. 함께 성장하는 쏘카의 개발 문화를 만들어 나가는 데 여러분의 관심과 참여를 기다리겠습니다. 

다음 편에서는 **쏘카 프론트엔드 모노레포 - Part2. 코어라이브러리 버전 관리** 시리즈로 이어가겠습니다.