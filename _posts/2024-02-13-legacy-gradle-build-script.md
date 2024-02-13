---
layout: post
title:  에셋팀 레거시 개선 (1) 쏘카존 관리 시스템
subtitle: Gradle build script 개선
date: 2024-02-13 00:00:00 +0900
category: dev
background: '/img/how-to-organize-tech-blog/thumbnail.jpg' 
author: onestone
comments: true
tags:
    - dev
    - asset
    - developer
    - service engineering
---



# 목차

0. **소개**
1. **개선 목적** 
2. **그레이들 프로젝트 현황 소개** 
3. **Root project의 `build.gradle` 파일 분석**
4. **`build.gradle` 파일 `build.gradle.kts`로 마이그레이션**
5. **`build.gradle.kts` 파일 개선**
    
    5.1. `gradle.properties`에 프로퍼티 선언

    5.2. 메이븐봄을 그레이들봄으로 변경

    5.3. 스프링이 관리하는 의존성 라이브러리의 버전 명시 제거

    5.4. `buildscript {}` 블록을 `plugins {}` 블록으로 대체

    5.5. 코틀린이 관리하는 아티팩트(`kotlin-dsl`) 제거

6. **build.gradle.kts 파일 에서 `buildscript {}` 블록 제거하기**

    6.1. 마커 없는 아티팩트 `plugins {}` 블록에서 사용

    6.2. 플러그인에 마커 추가 후 배포
<br><br><br>



# 0. 소개
안녕하세요. 쏘카 서비스 엔지니어링본부 에셋(Asset)팀 백엔드 개발자 원스톤입니다. 저는 쏘카 존과 차량 도메인을 개발하고 있습니다.

구성 코드를 적절히 관리해야만 지속 성장하는 소프트웨어를 만들 수 있습니다. 그레이들 빌드 스크립트 또한 지속 성장 가능한 소프트웨어 코드의 일부입니다. 지금부터 레거시 빌드 스크립트를 개선한 사례를 소개하겠습니다.

개선 작업이 필요했던 이유를 먼저 소개하고, `Root project`의 `build.gradle` 현황 분석과 마이그레이션 및 리펙토링, 전체 그레이들 멀티모듈 프로젝트에 대한 마이그레이션 순으로 소개하겠습니다.
<br><br><br>



# 1. 개선 목적
쏘카는 코틀린 스프링을 표준 기술로 채택하여 신규 프로젝트는 그레이들 코틀린 DSL을 사용합니다. 하지만 오래전 그루비 스크립트 언어로 개발되어 복잡하고 관리되지 않고 방치된 DSL 스크립트도 존재 합니다. 프로젝트마다 괸리 방식이 다르고 표준화 되지 않아 신규 개발자가 상황을 파악하고 개발하는데 어려움을 겪었습니다. 이에 그레이들 빌드를 활용해 코틀린 DSL와 그루비 DSL을 구성하고 작은 단위로 점진적 개선을 결정했습니다. 그레이들 8.x 버전부터 코틀린 DSL이 기본 언어로 채택 되었고 스크립트 코드의 자동완성을 지원 합니다. 더불어 코틀린 DSL은 컴파일 시점에 오류 확인이 가능해, 런타임 시점에 오류를 발견해야 하는 그루비 DSL에 비해 문법 오류를 명확하게 발견할 수 있는 장점이 있습니다. 모든 개발자가 겪는 어려움이지만 쏘카도 빠르게 성장하는 비즈니스의 요구사항을 충족하기 위해 기술 부채 청산에 시간을 할애 하기 어렵습니다. 비즈니스 요구사항 개발 속도에 영향이 없도록 개선 작업은 작은 단위로 분할하여 점진적으로 진행했습니다.
<br><br><br>



# 2. 그레이들 프로젝트 현황 소개
`gradle -q projects` 명령어로 확인해 보니 그레이들 프로젝트는 복잡한 멀티모듈 프로젝트로 구성되어 있습니다.

```
------------------------------------------------------------
Root project ‘gradle'
------------------------------------------------------------

Root project ‘gradle'
+--- Project ':api'
+--- Project ':batch'
+--- Project ':client'
|    +--- Project ':client:keycloak'
|    ...
|    \--- Project ':client:slack'
+--- Project ':core'
+--- Project ':daemon'
\--- Project ':grpc'
```

 루트 프로젝트(Root project)에 `settings.gradle`과 `build.gradle`, `gradle.properties` 파일로 구성 되어 있으며, 하위 프로젝트(Project)에 단일 `build.gradle` 파일이 있습니다. 우선 루트 프로젝트의 `build.gradle` 파일부터 마이그레이션 하기로 결정 했습니다.
<br><br><br>



# 3. Root project의 `build.gradle` 파일 분석
 레거시 `build.gradle` 파일은 아래와 같습니다.

 ```groovy
 // groovy
 buildscript {
   ext {
      kotlinVersion = '1.6.10'
      springBootVersion = '2.4.5'
      socarPluginVersion = '1.0.0'
      servletVersion = "2.5"
      poiVersion = "4.1.2"
      springCloudAwsVersion = "2.2.6.RELEASE"
      ...
   }
   repository {
      gradlePluginPortal()
      mavenCentral()
      maven {
         name = 'GitHubPackages'
         ...
      }
   }
   dependencies {
      ...
      classpath('socar:kotlin-gradle-plugin:$socarPluginVersion')
      classpath('org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlinVersion')
      classpath('org.springframework.boot:spring-boot-gradle-plugin:$springBootVersion')
      ...
   }
}

allprojects {
   repository {
      gradlePluginPortal()
      mavenCentral()
         maven {
         name = 'GitHubPackages'
         ...
      }
   }
}

subprojects {
   ...
   apply plugin: 'kotlin'
   apply plugin: 'org.springframework.boot'
   apply plugin: 'io.spring.dependency-management'
   ...

   dependencyManagement {
      imports {
         mavenBom 'org.springframework.cloud:spring-cloud-aws-dependencies:$springCloudAwsVersion'
         ...
      }

      dependencies {
         dependency('org.jetbrains.kotlin:kotlin-stdlib-jdk7:${kotlinVersion}')
         dependency('org.jetbrains.kotlin:kotlin-stdlib-jdk8:${kotlinVersion}')
         dependency('com.google.guava:guava:${guavaVersion}')
         ...
      }
   }
   ...
}
 ```

- `buildscript {}` 블록에 루트 프로젝트 및 하위 프로젝트에서 사용할 ext 프로퍼티를 선언하고, 메이븐 센트럴 혹은 프라이빗 메이븐 레포지터리의 아티팩트(jar 파일)의 의존성을 선언합니다.
- `allprojects {}` 블록에 루트 프로젝트 및 하위 프로젝트 전체에서 사용할 리포지터리를 선언합니다.
- `subprojects {}` 블록에 각 서브프로젝트에서 사용할 의존성을 선언합니다.
	- `dependencyManagement {}` 확장 블록에 mavenBom을 이용해 하위 프로젝트에서 루트 프로젝트의 스프링 버전과 호환되는 서드 파티 의존성 라이브러리들의 버전을 자동 구성합니다. 하위 `import {}` 블록으로 스프링의 의존성 봄을 가져오고 `dependencies {}` 블록에서는 스프링이 관리하는 의존성 버전을 오버라이딩하여 버전을 직접 관리합니다.

---
**Tip:**

*\* `buildScript {}`, `allProject {}`, `subProject {}`는 그레이들 예약어로 블록이라 부르며, `dependencyManagement {}`의 경우 스프링에서 제공하는 그레이들의 외부 플러그인으로 확장 블록이라 부릅니다.*
<br><br><br>



# 4. build.gradle 파일 build.gradle.kts로 마이그레이션
(1) 그루비 DSL은 작은따옴표(’), 큰따옴표(”)모두 사용할 수 있지만, 코틀린 DSL은 큰따옴표만 사용할 수 있습니다. 그러므로 첫 번째로 작은따옴표(’)로 되어있는 모든 문자열을 큰따옴표(”)로 일괄 수정합니다.

`변경예시)`
```groovy
// groovy
mavenBom 'org.springframework.cloud:spring-cloud-aws-dependencies:$springCloudAwsVersion' 
```

->

```groovy
// groovy
mavenBom "org.springframework.cloud:spring-cloud-aws-dependencies:$springCloudAwsVersion"
```

---


(2) 그루비 DSL은 함수를 호출할 때 괄호()를 생략할 수 있지만 코틀린에서는 항상 괄호가 필요합니다. 두 번째로 첫 번째에서 수행한 문자열의  `"..."` 를 `("...")`로 일괄 수정합니다.

`변경예시)`
```groovy
// groovy
mavenBom "org.springframework.cloud:spring-cloud-aws-dependencies:$springCloudAwsVersion"
```

->

```groovy
// groovy
mavenBom("org.springframework.cloud:spring-cloud-aws-dependencies:$springCloudAwsVersion")
```

---


(3) 그레이들의 build sync 명령을 사용하여 빌드 파일에 문제가 없는지 확인합니다. 빌드 오류가 발생하면 다음 내용을 수행합니다.
   - 그루비 DSL은 프로퍼티스 변수를 할당할 때 할당(`=`) 연산자를 생략할 수 있지만 코틀린에서는 항상 할당 연산자가 필요합니다. 프로퍼티 할당 오류가 발생하는 부분에 할당 연산자를 추가해 줍니다.

---


(4) build.gradle파일의 `.gradle`확장자를 `.gradle.kts`확장자로 변경하여 build.gradle.kts파일로 만들어 줍니다.

---


(5) 그루비 DSL의 ext 프로퍼티를 코틀린 DSL 문법에 맞춰 extra 프로퍼티로 수정 합니다.

`변경예시)`
```groovy
// groovy
ext {
   kotlinVersion = "1.6.10"
   springBootVersion = "2.4.5"
   socarPluginVersion = "1.0.0"
   servletVersion = "2.5"
   poiVersion = "4.1.2"
   springCloudAwsVersion = "2.2.6.RELEASE"
   ...
}
```

->

```kotlin
// kotlin
extra.apply {
   set("kotlinVersion", "1.6.10")
   set("springBootVersion", "2.4.5")
   set("socarPluginVersion", "1.0.0")
   set("servletVersion", "2.5")
   set("poiVersion", "4.1.2")
   set("springCloudAwsVersion", "2.2.6.RELEASE")
   ...
}
```

---


(6) 하드코딩된 버전을 참조가 용이하도록 변수로 선언합니다.

```kotlin
// kotlin
...
buildscript {
   val kotlinVersion = "1.6.10"
   val springBootVersion = "2.4.5"
   val socarPluginVersion = "1.0.0"
   val servletVersion = "2.5"
   val poiVersion = "4.1.2"
   ...

   extra.apply {
      set("kotlinVersion", kotlinVersion)
      set("springBootVersion", springBootVersion)
      set("socarPluginVersion", socarPluginVersion)
      set("servletVersion", servletVersion)
      set("poiVersion", poiVersion)
      set("springCloudAwsVersion", "2.2.6.RELEASE")
      ...
   }

   repository {
      gradlePluginPortal()
      mavenCentral()
      maven {
         name = "GitHubPackages"
         ...
      }
   }

   dependencies {
      ...
      classpath("socar:kotlin-gradle-plugin:$socarPluginVersion")
      classpath("org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlinVersion")
      classpath("org.springframework.boot:spring-boot-gradle-plugin:$springBootVersion")
      ...
   }
}
...
```

---


(7) extra 프로퍼티는 코틀린 델리게이트문법(`by`)을 이용해 `build.gradle.kts` 파일의 전역 변수로 선언할 수 있습니다. `extra.apply {}`블록을 제거하고, `buildscript {}` 블록에서 사용하지 않는 프로퍼티는 전부 전역 변수로 선언합니다.   

```kotlin
// kotlin
...
val kotlinVersion by { "1.6.10" }
val servletVersion by { "2.5" }
val poiVersion by { "4.1.2" }
val springCloudAwsVersion by { "2.2.6.RELEASE" }
...

buildscript {
   val kotlinVersion = "1.6.10"
   val springBootVersion = "2.4.5"
   val socarPluginVersion = "1.0.0"
   ...
   repository {
      gradlePluginPortal()
      mavenCentral()
      maven {
         name = "GitHubPackages"
         ...
      }
   }
   dependencies {
      ...
      classpath("socar:kotlin-gradle-plugin:$socarPluginVersion")
      classpath("org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlinVersion")
      classpath("org.springframework.boot:spring-boot-gradle-plugin:$springBootVersion")
      ...
   }
}
...
```

**Tip:**

*\* `buildscript {}` 블록 내에서는 전역 변수를 참조할 수 없으므로 별도 버전을 `buildscript {}` 블록에 명시해야 합니다.*

---


(8) `subproject {}` 블록의 apply 문법을 코틀린 DSL 문법에 맞게 변경합니다.

`변경예시)`
```groovy
// grrovy
apply plugin: "kotlin"
apply plugin: "org.springframework.boot"
apply plugin: "io.spring.dependency-management"
```

->

```kotlin
// kotlin
apply {
   plugin("kotlin")
   plugin("org.springframework.boot")
   plugin("io.spring.dependency-management")
}
```
---


(9) `dependencyManagement`는 스프링이 제공하는 의존성 관리 확장 블록입니다. 코틀린에서는 `plugins {}` 블록을 사용하지 않는 확장 블록에는 올바른 타입을 명시해야 합니다.

`변경예시)`
```groovy
// grrovy
dependencyManagement {
   imports {
      ...
   }

   dependencies {
      ...
   }
}
```

->

```kotlin
// kotlin
import io.spring.gradle.dependencymanagement.dsl.DependencyManagementExtension
...

the<DependencyManagementExtension>().apply {
   imports {
      ...
   }

   dependencies {
      ...
   }
}
```

---


(10) 모두 완료하면, 그루비 DSL에서 코틀린 DSL로 변경이 끝납니다.

```kotlin
// kotlin
import io.spring.gradle.dependencymanagement.dsl.DependencyManagementExtension

...
val kotlinVersion by extra { "1.6.10" }
val servletVersion by extra { "2.5" }
val poiVersion by extra { "4.1.2" }
val springCloudAwsVersion by { "2.2.6.RELEASE" }
...

buildscript {
   val kotlinVersion = "1.6.10"
   val springBootVersion = "2.4.5"
   val socarPluginVersion = "1.0.0"

   repository {
      gradlePluginPortal()
      mavenCentral()
      maven {
         name = "GitHubPackages"
         ...
      }
   }
   dependencies {
      ...
      classpath("socar:kotlin-gradle-plugin:$socarPluginVersion")
      classpath("org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlinVersion")
      classpath("org.springframework.boot:spring-boot-gradle-plugin:$springBootVersion")
      ...
   }
}

allproject {
   repository {
      gradlePluginPortal()
      mavenCentral()
      maven {
         name = "GitHubPackages"
         ...
      }
   }
}

subprojects {
   apply {
      ...
      plugin("kotlin")
      plugin("org.springframework.boot")
      plugin("io.spring.dependency-management")
      ...
   }
   ...
   the<DependencyManagementExtension>().apply {
      imports {
         mavenBom("org.springframework.cloud:spring-cloud-aws-dependencies:$springCloudAwsVersion")
         ...
      }

      dependencies {
         dependency("org.jetbrains.kotlin:kotlin-stdlib:$kotlinVersion")
         dependency("org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlinVersion")
         ...
      }
   }
   ...
}
```

<br><br><br>



# 5. `build.gradle.kts` 파일 개선
위에서 마이그레이션한 루트 프로젝트의 `build.gradle.kts`는 최신 그레이들 버전의 문법을 활용해 간결하게 개선할 수 있습니다. 이번 장에서는 `build.gradle.kts`와 `gradle.properties`를 다음과 같이 수정합니다.

- `gradle.properties`에 프로퍼티 선언
- 메이븐봄을 그레이들봄으로 변경
- 스프링이 관리하는 의존성 라이브러리의 버전 명시 제거
- `buildscript {}` 블록을 `plugins {}` 블록으로 대체
- 코틀린이 관리하는 아티팩트(*`kotlin-dsl`*) 제거


## 5.1. `gradle.properties`에 프로퍼티 선언
 4.10.절, 코틀린 DSL로 변경한 `build.gradle.kts` 파일에는 해당 빌드 스크립트 내에서만 사용하는 extra 프로퍼티 뿐 아니라 다른 하위 모듈에서 사용하는 extra 프로퍼티가 포함되어 있어 파일이 전체적으로 길어지는 문제가 있습니다.

 - 먼저, `build.gradle.kts` 파일의 extra 프로퍼티들을 `gradle.properties` 파일로 이동합니다.

`build.gradle.kts 파일`
```kotlin
// kotlin
...
val kotlinVersion by extra { "1.6.10" }
val servletVersion by extra { "2.5" }
val poiVersion by extra { "4.1.2" }
val springCloudAwsVersion by extra { "2.2.6.RELEASE" }
... 
```

`gradle.properties 파일`
```
...
kotlinVersion = 1.6.10
servletVersion = 2.5
poiVersion = 4.1.2
springCloudAwsVersion = 2.2.6.RELEASE
...
```
--- 


- `build.gradle.kts` 파일에서 사용할 extra 프로퍼티를 다음과 같이 선언해 줍니다. 코틀린 DSL에서 extra 프로퍼티를 사용하려면 다음과 같이 타입을 명시해야 합니다.

`build.gradle.kts 파일`
```kotlin
// kotlin
...
val kotlinVersion: String by project.extra
val springBootVersion: String by project.extra
val springCloudAwsVersion: String by project.extra
...

subprojects {
   ...
   the<DependencyManagementExtension>().apply {
      imports {
         mavenBom("org.springframework.cloud:spring-cloud-aws-dependencies:$springCloudAwsVersion")
         ...
      }

      dependencies {
         dependency("org.jetbrains.kotlin:kotlin-stdlib:$kotlinVersion")
         dependency("org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlinVersion")
         ...
      }
   }
   ...
}
```

- `gradle.properties`에 extra 프로퍼티를 옮겨 `build.gradle.kts`에서 사용할 프로퍼티를 선언할 수 있게 개선했습니다.

---

## 5.2. 메이븐 봄을 그레이들 봄으로 변경
 메이븐 봄에 비해 그레이들 봄이 좀 더 DSL 스크립트 코드를 깔끔하게 관리할 수 있습니다. 4.10.절의 예제를 다시 사용해서 메이븐 봄을 사용하는 부분을 그레이들 봄을 사용하도록 변경해 보겠습니다.

`변경예시)`
```kotlin
// kotlin
import io.spring.gradle.dependencymanagement.dsl.DependencyManagementExtension

...
subprojects {
   ...
   the<DependencyManagementExtension>().apply {
      imports {
         mavenBom("org.springframework.cloud:spring-cloud-aws-dependencies:$springCloudAwsVersion")
         ...
      }

      dependencies {
         dependency("org.jetbrains.kotlin:kotlin-stdlib:$kotlinVersion")
         dependency("org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlinVersion")
         ...
      }
   }
   ...
}
```

->

```kotlin
// kotlin
...
subprojects {
   ...
   dependencies {
       implementation(platform("org.springframework.boot:spring-boot-dependencies:$springBootVersion"))
       implementation("org.jetbrains.kotlin:kotlin-stdlib:$kotlinVersion")
       implementation("org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlinVersion")
      ...
   }
   ...
}
```

**Tip:**

*\* bom이란 Bill Of Material의 약자로 사용할 의존성 라이브러리 버전의 명세가 기록되어 있습니다. 이를 사용하여 하위 모듈에서는 버전을 명시하지 않고 BOM에 명세된 버전을 사용하게 됩니다.*

---


## 5.3. 관리되는 의존성 라이브러리의 버전 명시 제거
아래 두 가지의 의존성 관리를 추가해주면, 하위 모듈의 버전을 명시하지 않아도 됩니다. (공식문서에 해당 라이브러리 버전으로 테스트가 완료되어 변경하지 말라는 경고가 있습니다.)

- io.spring.dependency-management
- org.springframework.boot:spring-boot-dependencies

스프링(2.5.6)이 관리하는 의존성 버전에 대한 정보는 [다음 페이지](https://docs.spring.io/spring-boot/docs/2.5.14/reference/html/dependency-versions.html#appendix.dependency-versions)에 있습니다. 스프링 AWS (2.x) 라이브러리 의존성에 대한 정보는 [깃헙의 pom.xml](https://github.com/awspring/spring-cloud-aws/blob/2.3.x/spring-cloud-aws-dependencies/pom.xml)에서 확인할 수 있습니다. 어떤 의존성 라이브러리가 관리되고 있는지 확인하고 작성하면 스크립트 코드의 가독성과 서버의 안정성이 높아집니다.

`변경예시)`
```kotlin
// kotlin
kapt("com.querydsl:querydsl-apt:$queryDslVersion:jpa")
compileOnly("org.springframework.boot:spring-boot-starterweb:$springBootStarterWebVersion")
compileOnly("org.springframework.boot:spring-boot-starter-data-redis:$springBootStarterDataRedisVersion")
compileOnly("org.springframework.cloud:spring-cloud-starter-aws:$springCloudStarterAWSVersion")
```

->

```kotlin
// kotlin
kapt(group = "com.querydsl", name = "querydsl-apt", classifier = "jpa")
compileOnly("org.springframework.boot:spring-boot-starterweb")
compileOnly("org.springframework.boot:spring-boot-starter-data-redis")
compileOnly("org.springframework.cloud:spring-cloud-starter-aws")
```

---

## 5.4. `buildscript {}` 블록을 `plugins {}` 블록으로 대체
코틀린 DSL에서는 타입 안전 접근자를 사용하기 위해서 `plugins {}` 블록을 사용해야 합니다. `plugins {}` 블록에 아티팩트를 추가하기 위해서는 그레이들 플러그인 포탈에 등록되어 있거나, 플러그인 마커 아티팩트와 함께 게시되어야 합니다.

**Tip:**

*\*해당 글에서는 용어의 혼동을 방지하기 위해 바이너리 아티팩트, 플러그인, 의존성 모듈을 모두 아티팩트로 통일합니다. 의존성 라이브러리란 코드에서 가져다 쓰는 것들로 한정합니다.*

---


- 그레이들 코어 플러그인에 대한 아티펙트는 좀 더 간결하게 사용가능합니다. 

```kotlin
// kotlin
id("java")
```

->

```kotlin
// kotlin
java
```

- 코틀린 DSL을 사용할 경우, org.jetbrains.kotlin:kotlin-test를 kotlin(”test”)과 같이 코틀린 아티팩트에 대한 줄임말를 사용할 수 있습니다.

```kotlin
// kotlin
id("org.jetbrains.kotlin.kapt") // https://plugins.gradle.org/plugin/org.jetbrains.kotlin.kapt
id("org.jetbrains.kotlin.jvm") // https://plugins.gradle.org/plugin/org.jetbrains.kotlin.jvm
id("org.jetbrains.kotlin.plugin.spring") // https://plugins.gradle.org/plugin/org.jetbrains.kotlin.plugin.spring
```

->

```kotlin
// kotlin
kotlin("kapt")
kotlin("jvm")
kotlin("plugin.spring")
```


- 이렇게 하여 다음과 같이 `buildscript {}` 블록 아래 `plugins {}` 블록을 추가했습니다.

```kotlin
// kotlin
import io.spring.gradle.dependencymanagement.dsl.DependencyManagementExtension
...
buildscript {
   val kotlinVersion = "1.6.10"
   val springBootVersion = "2.4.5"
   val socarPluginVersion = "1.0.0"

   repository {
      gradlePluginPortal()
      mavenCentral()
      maven {
         name = "GitHubPackages"
         ...
      }
   }
   dependencies {
      classpath("socar:kotlin-gradle-plugin:$socarPluginVersion")
   }
}

plugins {
   val kotlinVersion = "1.8.10"

   java
   kotlin("kapt") version kotlinVersion
   kotlin("jvm") version kotlinVersion
   kotlin("plugin.spring") version kotlinVersion
   kotlin("plugin.allopen") version kotlinVersion
   id("org.gradle.kotlin.kotlin-dsl") version "4.0.7"
   id("org.springframework.boot") version "2.5.14" apply false
}
...
```

`buildscript {}` 블록을 제거하지 못한 이유는 플러그인 마커 아티팩트에 대한 메타데이터가 존재하지 않는 아티팩트를 사용중이기 때문입니다. `plugins {}` 블록 또한 `buildscript {}` 블록같은 특수 블록이라 버전에대한 정보를 extra에서 불러오지 못합니다.

---

## 5.5. 코틀린이 관리하는 아티팩트(kotlin-dsl) 제거
`plugins {}` 블록 내에 문제가 될만한 아티팩트 선언이 존재합니다. `org.gradle.kotlin.kotlin-dsl`는 코틀린 버전에 의존성이 있는 아티팩트이므로 버전을 직접 명시할 경우 문제가 발생할 수 있습니다. 기본적으로 `id("org.gradle.kotlin.kotlin-dsl") version "4.0.7"` 을 `kotlin-dsl` 으로 간소화하여 표현 가능하며, 코틀린 버전에 따라 아티팩트를 가져오기 때문에 제거하여 코틀린 아티팩트가 관리하는 버전을 사용하는게 안정적입니다.
```kotlin
// kotlin
...
plugins {
   val kotlinVersion = "1.8.10"

   java
   kotlin("kapt") version kotlinVersion
   kotlin("jvm") version kotlinVersion
   kotlin("plugin.spring") version kotlinVersion
   kotlin("plugin.allopen") version kotlinVersion
   // 제거 - id("org.gradle.kotlin.kotlin-dsl") version "4.0.7"
   id("org.springframework.boot") version "2.5.14" apply false
}
...
```

아티팩트 및 라이브러리 버전은 각 생태계가 관리하는 버전을 사용하여 의존성 버전의 관리 포인트를 줄일 수 있습니다. 뿐만아니라, 각 생태계에서 버전에 맞는 테스트와 지원을 제공하기 때문에 안정적으로 사용할 수 있습니다. 각 생태계의 의존성 버전에 대한 지속적인 관심이 필요합니다.
<br><br><br>



# 6. build.gradle 파일 에서 `buildscript {}` 블록 제거하기
아직 `buildscript {}` 블록과 `plugin {}`블록을 함께 사용하여 깔끔하지 못한 상태로 남아있습니다. 이번 장에서는 `buildscript {}` 블록을 제거해 보겠습니다. 진행할 내용은 다음과 같습니다.

- 마커가 없는 아티팩트 `plugins {}` 블록에서 사용
- 아티팩트에 마커 추가 후 배포


## 6.1. 마커 없는 아티팩트 `plugins {}` 블록에서 사용
마커가 없는 아티팩트를 `plugins {}` 블록에서 사용하려면 `settings.gadle.kts` 파일을 수정해야 합니다. `pluginManagement {}` 블록을 `settings.gadle.kts` 파일 최상단에 추가하고, `resolutionStrategy {}` 와 `repositories {}`를 설정해 줍니다.

`settings.gadle.kts`
```kotlin
// kotlin
pluginManagement {
   resolutionStrategy {
      eachPlugin {
         if (requested.id.id == "kr.socar.gradle.plugin") {
            useModule("kr.socar:gradle-plugin:1.13.0")
         }
      }
   }

   repositories {
      mavenLocal()
      maven {
         url = uri("https://maven.pkg.github.com/example/maven")
         credentials {
            username = xxxxx
            password = xxxxx
         }
      }
      gradlePluginPortal()
   }
}
```
 위 코드는 `plugins {}` 블록에서 id가 "kr.socar.gradle.plugin"인 아티팩트 요청이 있다면, `repositories {}`  저장소에 `"kr.socar:gradle-plugin:1.13.0"` 라이브러리를 요청하라는 것으로 해석할 수 있습니다. 
 
 **Tip:**

*\*기본적으로 `plugins {}` 블록은 그레이들 플러그인 포탈에서만 아티팩트를 찾기 때문에 `settings.gadle.kts` 파일에 `pluginManagement {}` 블록의 `repositories {}` 블록을 설정해주어야 다른 저장소의 아티팩트를 가져올 수 있습니다.*

 ---

`build.gadle.kts`
```kotlin
// kotlin
...
// buildscript { ... } 제거

plugins {
   val kotlinVersion = "1.8.10"

   java
   kotlin("kapt") version kotlinVersion
   ...
   id("kr.socar.gradle.plugin") apply false // apply false 필수
}
...
```

`plugins {}` 블록에 마커 없는 아티팩트를 적용하여 `buildscript {}` 블록을 제거하여 깔끔하게 사용이 가능합니다.

---


## 6.2.  아티팩트에 마커 추가 후 배포
이번 절에서는 그레이들 게시와 플러그인 개발에 대한 자세한 내용은 제외하고, 개발 된 플러그인에 마커를 추가하는 방법을 다룹니다.

6.1.절 까지 완료되면, `buildscript {}` 블록을 더 이상 사용하지 않아도 되므로, 빌드 스크립트를 깔끔하게 관리할 수 있게 됐습니다. 하지만 마커 없는 아티팩트는 타입 안전 접근자를 사용하지 못해 확장 블록을 사용할 수 없는 불편한 점이 있습니다.

 `"kr.socar:gradle-plugin"` 아티팩트는 2장의 *"현재 그레이들 프로젝트 구성"* 에 `Project ':core'`에서 사용하고 있는 플러그인입니다. `:core` 프로젝트의 `build.gradle` 파일을 확인해보면 해당 아티팩트의 플러그인을 다음과 같이 `exposedModelGenerator {}` 확장 블록을 사용하여 구성하고 있습니다.

`core/build.gradle`
```groovy
// groovy
apply plugin: 'socar-gradle-plugin'

...
exposedModelGenerator {
   socarZone {
      ...
   }
   legacySocar {
      ...
   }
   legacySocarLog {
      ...
   }
}
...
```

마커가 없는 아티팩트를 사용하여 플러그인을 구성한 빌드 스크립트를 `build.gradle.kts` 로 마이그레이션 하면, 다음과 같이 확장 블록을 사용하지 못하는 코드가 됩니다.

`core/build.gradle.kts`
```kotlin
// kotlin
import kr.socar.gradle.plugin.ExposedModelGenerator

apply {
   plugin("socar-gradle-plugin")
}

...

// 이 태스크의 명칭은 "generateSocarZone"
// 클래스 타입은 ConventionTask를 상속받은 ExposedModelGenerator
tasks.register<ExposedModelGenerator>("generateSocarZone") {
   ...
}

tasks.register<ExposedModelGenerator>("generateLegacySocar") {
   ...
}

tasks.register<ExposedModelGenerator>("generateLegacySocarLog") {
   ...
}
```
해당 아티펙트는 사내에서 직접 만들어 배포하고 있기 때문에, 마커를 추가하여 아티펙트를 게시해 보겠습니다.

 아티팩트를 만드는 프로젝트로 이동한 뒤, 다음과 같이 `build.gradle` 스크립트에 마커를 추가하기 위한 아티팩트와 `gradlePlugin {}` 확장 블록을 추가합니다.

 `build.gradle`
```groovy
// groovy
...
plugins {
   id 'java-gradle-plugin'
}

...
gradlePlugin {
   plugins {
      // 플러그인명
      socarGradlePlugin {
         // plugins {} 블록에서 사용할 id
         id = 'kr.socar.gradle.plugin'
         // Plugin<Project>을 상속한 클래스명
         implementationClass = 'kr.socar.gradle.plugin.SocarGradlePlugin'
      }
   }
}
```

아래와 같이 `maven-publish`와 상호작용하여 Marker와 함께 게시되는 플러그인이 추가됩니다.
![1](/img/2024-02-13-legacy-gradle-build-script/socar_legacy_gradle_plugin_img_1.png)

해당 플러그인을 실행하여 리포지터리에 게시가 완료되면, 아래와 같이 `plugins {}` 블록에서 사용가능하게 되며 확장 블록을 사용할 수 있게 됩니다.

`core/build.gradle.kts`
```kotlin
// kotlin
plugins {
   id("kr.socar.gradle.plugin") version "0.0.0-test"
}

...

exposedModelGenerator {
   ...
}
```
<br><br><br>


# 마무리
4개의 파일을 수정하여 마이그레이션 및 리펙토링을 진행했습니다. 스크립트 속도 개선, 의존성 라이브러리의 버전 업 등의 변경점이 있을 때 빌드 스크립트가 발목 잡는 경우가 발생하지 않도록 가독성 높고 일원화된 방식의 관리가 필요하다고 생각합니다.

`gradle.properties`
```groovy
// groovy
...
kotlinVersion = 1.6.10
servletVersion = 2.5
poiVersion = 4.1.2
springCloudAwsVersion = 2.2.6.RELEASE
...
```

`settings.gradle.kts`
```kotlin
// kotlin
pluginManagement {
   repositories {
      mavenLocal()
      maven {
         url = uri("https://maven.pkg.github.com/example/maven")
         credentials {
            username = xxxxx
            password = xxxxx
         }
      }
      gradlePluginPortal()
   }
}

include("api")
include("core")
include("grpc")
...
```

`build.gradle.kts`
```kotlin
// kotlin
import io.spring.gradle.dependencymanagement.dsl.DependencyManagementExtension

...
...
val kotlinVersion: String by project.extra
val springBootVersion: String by project.extra
val springCloudAwsVersion: String by project.extra
...
plugins {
   val kotlinVersion = "1.8.10"

   java
   kotlin("kapt") version kotlinVersion
   kotlin("jvm") version kotlinVersion
   kotlin("plugin.spring") version kotlinVersion
   kotlin("plugin.allopen") version kotlinVersion
   id("org.springframework.boot") version "2.5.14" apply false
}

allproject {
   repository {
      gradlePluginPortal()
      mavenCentral()
      maven {
         name = "GitHubPackages"
         ...
      }
   }
}

subprojects {
   apply {
      ...
      plugin("kotlin")
      plugin("org.springframework.boot")
      plugin("io.spring.dependency-management")
      ...
   }
   ...
   the<DependencyManagementExtension>().apply {
      imports {
         mavenBom("org.springframework.cloud:spring-cloud-aws-dependencies:$springCloudAwsVersion")
         ...
      }

      dependencies {
         dependency("org.jetbrains.kotlin:kotlin-stdlib:$kotlinVersion")
         dependency("org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlinVersion")
         ...
      }
   }
   ...
}
```

`core/build.gradle.kts`
```kotlin
// kotlin
plugins {
   id("kr.socar.gradle.plugin") version "0.0.1-test"
}

...

exposedModelGenerator {
   ...
}
```

그레이들 스크립트 관리 노하우를 배우고 싶다면 쏘카 에셋팀에 지원해 주세요!
<br><br><br>



---
참고 자료
- [그루비에서 코틀린으로 빌드 로직 마이그레이션](https://onestone9900.github.io/docs/gradle/7.6/3.migrating_from_groovy_to_kotlin_dsl/)
- [그레이들 플러그인 사용](https://onestone9900.github.io/docs/gradle/7.6/4.using_gradle_plugins/)
- [그레이들 코틀린 DSL 입문서](https://onestone9900.github.io/docs/gradle/7.6/2.gradle_kotlin_dsl_primer/)
- [스프링 그레이들 플러그인 의존성 관리](https://onestone9900.github.io/docs/spring_boot/3.1.1/gradle_plugin/3.managing_dependencies/)
- [스프링 의존성 버전 공식 문서](https://docs.spring.io/spring-boot/docs/current/reference/html/dependency-versions.html#appendix.dependency-versions)


