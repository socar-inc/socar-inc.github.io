---
layout: post
title: "Android Studio 플러그인으로 코드 자동 리팩토링하기"
subtitle: "IntelliJ platform plugin을 활용한 대량의 Kotlin 코드 수정"
date: 2021-12-29 09:00:00 +0900
category: dev
background: '/assets/images/intellij-plugin/plugin.png'
author: jian
comments: true
tags:
  - kotlin
  - intellij
  - android-studio
  - plugin
---

리팩토링 작업은 한두 개의 함수를 개선하는 것으로 충분한 때도 있지만, 때로는 여러 개의 파일을 전체적으로 고치고 나서야 끝이 나는 경우도 있습니다.
복잡한 로직을 수정하는 것이 아니라 단순한 코드 변경이라고 생각해서 예상 작업량을 짧게 잡고 덤벼들었다가도, 리팩토링의 대상이 되는 코드가 곳곳에 사용되고 있다면 코드 전체를 수정하는 것에는 최초의 예상보다 많은 시간이 소요되어 난처해지는 경우도 생기곤 합니다.
또한 그 과정에서 처음에는 예상하지 못했던 특이한 케이스라던가, 수정 과정에서 실수로 빠트리는 부분, 반복되는 동일 작업으로 인한 집중력 하락 등으로 인해 일정이 밀리다 보면 '리팩토링을 시작하지 말았어야 했나?' 하는 생각이 드는 경우도 있습니다.

안녕하세요, 쏘카 안드로이드팀의 지안(전현기)입니다. 저희 안드로이드팀에서도 몇 개월 전, 처음에는 단순해 보이지만 수십에서 수백 개 파일에 걸친 변경사항을 하나하나 고치려고 하다 보면 마냥 단순하지만은 않았던 리팩토링을 수행했던 경험이 있었습니다.
그리고 자칫 길어질 뻔했던 그 반복적인 작업은 IntelliJ platform plugin을 통해서 훨씬 수월해질 수 있었습니다.

당시에 리팩토링 작업을 하며 내부 세미나를 통해 발표했던 내용을, 연말연시를 맞이하여 이 글을 통해 다시금 정리해서 공유해 보고자 합니다.

---
# 리팩토링을 마음먹게 된 계기
## View binding으로의 전환
안드로이드에서 뷰에 접근하는 방식은 [계속해서 바뀌어](https://medium.com/mobile-app-development-publication/how-android-access-view-item-the-past-to-the-future-bb003ae84527) 왔습니다.
이러한 변화 중에서 현재 가장 이슈가 되고 있는 것은 아무래도 Jetpack Compose의 [정식 출시](https://android-developers.googleblog.com/2021/07/jetpack-compose-announcement.html)겠지만, 이 글은 그보다 약간 이전에 있었던 사건에 관한 이야기입니다.

2020년 말, Kotlin Synthetics 가 Kotlin Android Extensions와 함께 [deprecated](https://github.com/JetBrains/kotlin/releases/tag/v1.4.20) 되었습니다.
다행스럽게도 저희는 그 시점에 Kotlin Synthetics를 직접 사용하지 않고 [ButterKnife](https://github.com/JakeWharton/butterknife) 기반의 [ButterKt](https://github.com/Rajin9601/ButterKt)를 수정해서 활용하고 있었기 때문에 deprecation의 직접적인 영향 없이 Kotlin 버전을 업데이트할 수 있었습니다.
그러나 ButterKnife의 readme 파일에도 적혀있듯이, view binding을 사용하라는 권고는 늘 마음 한편(과 기술 부채 목록)에 남아있었죠.

하기야 뷰에 대한 타입 추론도 잘해주고, 속도도 이전에 비해 빨라지고, 뷰에 대한 구조적인 접근도 가능한 view binding을 사용하는 것에 딱히 나쁜 점은 없었습니다.
더군다나 이렇게 [migration 가이드](https://developer.android.com/topic/libraries/view-binding/migration)도 제공하고 있고요.

가이드를 보면서 저희 코드를 기준으로 얼핏 생각해 보았을 때는 기존에 사용하던 아래와 같은 코드를
```kotlin
class SomeActivity : BaseActivity() {
  private val maybe_different_name: TextView by bindView(R.id.declared_id)

  fun someFunction() {
    ...
    maybe_different_name.text = "value"
  }
}
```
이렇게 아래처럼 변경해 주기만 하면 될 것으로 보입니다.
```kotlin
class ChangedActivity : BaseActivity() {
  fun changedFunction() {
    ...
    binding.declaredId.text = "value"
  }
}
```

그러나 이렇게 가벼운 마음으로 작업을 하다 보면 이어지는 절에서 이야기할 번거로운 부분들이 보이기 시작합니다.
혹시 이런 번거로운 부분들을 해결해 줄 더 좋은 migration 방법이 제공될까 싶어서 기다리다 보니, 어느새 이 작업의 우선순위는 다른 feature들에 의해 계속해서 밀리게 되었습니다.

하지만 역설적이게도 우선순위가 높은 feature 화면의 개발 중에는 여전히 뷰의 타입이나 XML ID 매칭으로 인한 문제가 종종 발생해서 시간을 소비하곤 했습니다.
결국 이러한 문제로 인해 개발 시간이 불필요하게 늘어나고 있다는 의견에 도달하자 view binding으로 전환하는 리팩토링을 본격적으로 시작하게 되었습니다.
다만, 무작정 작업에 돌입하기보다는 효율적인 방법에 대해서 생각해 볼 필요가 있었죠.

---
# 리팩토링 검토
## View binding 전환에 필요한 것
위에서도 언급했다시피 view binding으로 리팩토링하는 작업을 위해 요구사항을 정리하다 보면 처음에 가졌던 생각보다 번거로운 지점이 보여서 멈칫멈칫하게 되는 부분들이 있었습니다.

- `by bindView` delegate를 사용하는 변수들을 쓰고 있는 모든 usage에 대해 이름 변경이 필요하다.
- 기존 방식과는 달리 앞에 `binding.`을 붙여서 접근하도록 해야 한다.

이 부분만 놓고 보면 단순하게 regex를 사용해서 치환해도 어떻게든 가능할 것 같다는 생각이 듭니다.
그런데 이 작업만으로는 충분하지 않습니다.

- Kotlin 파일에서 `bindView`로 사용하고 있는 변수명의 케이스를 변경(`snake_case` → `lowerCamelCase`)해줘야 한다.
- 만약 XML에 있는 ID(`R.id.~`)와 다른 변수명을 사용하고 있다면 XML의 ID를 사용하도록 변경해야 한다.

이 시점에서 regex를 이용한 '단순' 치환은 어렵겠다는 생각이 듭니다.
그래도 Android Studio를 사용하고 있으니 IDE의 기능을 빌어서 refactoring → rename 기능을 시도해 볼 수는 있을 것 같습니다.
변수 하나를 변경하는데 타이핑을 빠르게 하면 5~10초 정도 걸리는 것 같으니 나쁘지는 않아 보입니다.
하지만...

- Activity, fragment, custom view를 포함한 뷰 코드 파일들이 백 개가 넘고, 각각의 파일에는 XML ID와 연결되어 있는 변수가 수십 개 있다.

이렇게 되면 하나하나 타이핑해가면서 수동으로 수정하기에는 부담스러운 분량입니다.
수정하는 과정에서 행여나 누락되는 곳이나 실수하는 곳이 있지는 않을지 걱정도 되고요.
단순한 작업이다 싶어서 시작한 일인데 이렇게까지 반복적인 작업을 오랜 시간에 걸쳐서 신경 써가며 작업해야 할까 싶은 생각이 듭니다.

그렇게 해서 자동으로 Kotlin 코드를 파싱 하여 수정하는 방법들까지도 검토해 보게 되었습니다.

## 사용해 볼 만한 방법들
그런 생각을 거쳐서 아래에 있는 다섯 가지 정도의 방법을 떠올리고 간단하게 비교를 진행했습니다.

- Android Studio의 rename 기능을 변수 하나하나에 적용해서 바꾸기
- Regex를 사용해서 찾아 바꾸기
- Kotlin compiler의 [parsing](https://github.com/JetBrains/kotlin/tree/master/compiler/psi/src/org/jetbrains/kotlin/parsing)을 사용하기
- LSP와 [비공식 Kotlin Language Server](https://github.com/fwcd/kotlin-language-server)의 [rename](https://github.com/fwcd/kotlin-language-server/pull/319)을 사용하기
- [IntelliJ Platform Plugin](https://plugins.jetbrains.com/docs/intellij/welcome.html)을 만들어서 동작시키기

이 중에서 Android Studio의 IDE 기능을 써서 renaming 하는 방식은 신뢰도가 높고 추가적인 개발이 필요 없지만, 각각의 변수들을 하나하나 수정해야 하므로 작업에 걸리는 시간이 길어진다는 단점이 있었습니다.

그리고 정규표현식으로 찾아 바꾸는 작업은 여러 변수나 파일들을 한꺼번에 고칠 수는 있었지만, reference(usage)를 명확하게 구분해내기가 어렵고 정규표현식을 작성하는 것에도 시간이 많이 소요된다는 문제가 있었습니다.

Kotlin compiler를 써서 parsing 하거나, LSP를 사용해서 수정하는 것은 해당 기능을 개발하기 위해 필요한 배경지식들이 과도하게 많이 필요했습니다.
전체 작업에 드는 시간을 고려하면 변수를 하나하나 바꾸는 데 걸리는 시간이 오히려 비슷하거나 빠를 수도 있겠다는 판단도 들었습니다.
또한 Kotlin Language Server는 아직 [공식적으로 제공되지 않고 있으며](https://discuss.kotlinlang.org/t/any-plan-for-supporting-language-server-protocol/2471), 비공식 language server에서는 [rename 기능에 문제](https://github.com/fwcd/kotlin-language-server/pull/319)가 있다는 이야기도 있어서 섣불리 시도해 보기도 어려웠고요.

반면에 IntelliJ Platform Plugin은 기존에 IDE에서도 사용해왔던 기능들을 그대로 사용할 테니 reference를 제대로 찾아서 변경해 주는 안정성이 확보되어 있다고 볼 수 있었습니다.
또한 이미 다양한 기능을 가진 플러그인들이 plugin marketplace에 올라와 있는 것을 보면 단지 이번 리팩토링뿐만 아니라 다른 기능을 추가해 볼 수도 있을 것이라는 생각도 들었습니다.
물론 개발에 들어가는 시간이 있겠지만 Kotlin compiler나 LSP를 다루는 것보다는 빠르게 진행할 수 있으리라고 보았습니다.

앞서 이야기한 몇몇 기준점을 가지고 각각의 방식을 간략하게 비교하면 아래의 표와 같이 정리할 수 있습니다.

|  | IntelliJ rename | Regex replace | Kotlin parser | Language server | IntelliJ Plugin |
|:---:|:---:|:---:|:---:|:---:|:---:|
| 대량 수정(자동화) | X | O | O | O | **O** |
| Usage 검색 기능 | O | X | O | O | **O** |
| 포매팅 유지 | O | X | ? | ? | **O** |
| 개발에 필요한 시간 | 없음 | 보통 | 많음 | 많음 | 보통 |
| 확장성 | 낮음 | 낮음 | 보통 | 보통 | **높음** |

이러한 비교를 바탕으로 개발 시간이 많이 필요하지 않으면서도 자동화가 가능한 IntelliJ Platform Plugin 방식을 선택했고, 이를 통해 view binding으로의 리팩토링을 진행해 보기로 했습니다.

---
# IntelliJ Platform Plugin
그렇다고 하더라도 자료를 찾기 어렵다면 개발 시간이 길어질 것이므로 걱정했지만, 다행스럽게도 JetBrains에서는 공식적으로 IntelliJ Platform에서 사용 가능한 [플러그인](https://lp.jetbrains.com/gradle-intellij-plugin/) 개발에 대한 [문서](https://plugins.jetbrains.com/docs/intellij/welcome.html)를 제공하고 있었습니다.
이 문서의 [Getting Started 페이지](https://plugins.jetbrains.com/docs/intellij/getting-started.html)에 따르면 아래와 같은 방식으로 플러그인을 개발하는 것을 권장하고 있습니다.

> There are three supported workflows available for building plugins. The **recommended workflow** for new projects is to use [GitHub Template](https://github.com/JetBrains/intellij-platform-plugin-template) or to use [Gradle](https://github.com/JetBrains/gradle-intellij-plugin) to create everything from scratch. The old Plugin DevKit workflow still supports existing projects.

이렇게 나열된 방식 중에서 비교적 간단하게 개발할 수 있는 방식인 [GitHub Template](https://github.com/JetBrains/intellij-platform-plugin-template)를 통해 그 안에 있는 예제 플러그인 코드로부터 개발을 시작했습니다.

## 예제 플러그인 동작 확인
[링크](https://github.com/JetBrains/intellij-platform-plugin-template)로부터 예제 템플릿 레포지토리를 클론 해와서 Android Studio로 열어보면 `Run Configurations` 중에 `Run Plugin`이라는 항목을 볼 수 있습니다.
그 항목을 선택하고 `Run` 버튼을 눌러서 이를 실행시키면 예제 플러그인이 설치되어 동작할 IntelliJ Community Edition이 자동으로 다운로드되고, 그 sandbox 인스턴스 IDE가 새로 뜨며, 그 위에서 예제 플러그인이 돌아가는 것을 확인해 볼 수 있습니다.

[설명](https://github.com/JetBrains/intellij-platform-plugin-template/tree/v1.1.0#plugin-configuration-file)에도 나와있듯이 `/src/main/resources/META-INF/plugin.xml` 파일에 `applicationService`로 지정된 `MyApplicationService.kt`, `projectService`로 지정된 `MyProjectService.kt`가 sandbox IDE의 로드 시점에 수행되며, `println`으로 출력하는 메시지가 바깥쪽 IDE의 Run 탭에 출력되는 것을 확인할 수 있었습니다.

![기본 예제 확인](/assets/images/intellij-plugin/sample.png)

## Kotlin 코드 다루기
하지만 우리가 플러그인을 통해 최종적으로 이름 변경을 하기 위해서는 Kotlin 코드를 인식하고 분석하는 기능이 필요합니다.
기존에 Android Studio를 비롯한 IntelliJ 계열 IDE에도 renaming 기능이 있기 때문에, 플러그인에도 관련 내용이 있으리라 판단했고, 아니나 다를까 [이 문서](https://plugins.jetbrains.com/docs/intellij/kotlin.html#handling-kotlin-code)에서 Kotlin 관련 내용을 찾을 수 있었습니다.

> If a plugin processes Kotlin code (e.g., providing inspections), it needs to add a dependency on the Kotlin plugin (Plugin ID `org.jetbrains.kotlin`) itself. Please refer to [Plugin Dependencies](https://plugins.jetbrains.com/docs/intellij/plugin-dependencies.html) for more information.

제시된 여러 문서를 따라가 보면 아래의 두 가지 작업으로 귀결됩니다.
우선 `plugin.xml`에 아래의 코드를 추가하고,

```xml
...
<depends>org.jetbrains.kotlin</depends>
...
```

`build.gradle.kts`이 플러그인 의존성 값을 받아오고 있는 `gradle.properties` 파일에다가 아래와 같이 `java`, `Kotlin` 플러그인 의존성을 추가해서 IntelliJ **Kotlin** Plugin 의존성을 사용하도록 하면 됩니다.
```yaml
...
platformPlugins = ..., java, Kotlin
...
```

이제 해당 변경사항을 IDE가 인지할 수 있도록 `Sync Project with Gradle Files`를 해주면 Kotlin 코드를 다룰 준비가 되었습니다.

## PSI 사용하기
실질적으로 코드를 다루는 작업은 IntelliJ platform에서 제공하는 인터페이스인 [PSI(Program Structure Interface)](https://plugins.jetbrains.com/docs/intellij/psi.html)를 통해서 진행합니다.
여기서 PSI란 [이곳](https://plugins.jetbrains.com/docs/intellij/implementing-parser-and-psi.html)에 적혀있는 것처럼 특정 언어를 다루기 쉽도록 IntelliJ platform이 파싱한 AST 요소들 위에 부가정보(문법적인 정보나, 언어 특유의 속성)들을 더한 것입니다.
저희는 위에서 적었던 `platformPlugins = ..., Kotlin`을 통해서 IntelliJ Kotlin plugin이 제공하는 Kotlin PSI를 사용할 수 있게 되었습니다.

즉, 위에서 `Kotlin` 의존성을 추가해 줌으로써 `org.jetbrains.kotlin.psi.KtClass`와 같이 `org.jetbrains.kotlin` 패키지에 있는 내용을 우리가 만드는 플러그인 코드에서 사용할 수 있고, 그래서 이제 이 플러그인에서 아래와 같은 동작을 할 수 있게 되었습니다.

PSI tree에 있는 이 Kotlin PSI element에 대해서
- 해당 element가 class인지 property인지 function인지 판별
- 이 element를 참조하고 있는 다른 element로 이동
- AST에 있는 하위 element들은 어떤 것들이 있는지 확인

등의 다양한 동작을 해볼 수 있습니다.

### 모든 KtClass 이름 출력
Kotlin PSI를 사용해서 프로젝트에 있는 모든 `.kt` 파일에 정의된 Kotlin class의 이름을 출력해 보려면 아래와 같은 함수를 만들어서 사용해 볼 수 있습니다.

```kotlin
fun printKtClassNames(project: Project) {
    project.allModules().forEach { module ->
        FilenameIndex.getAllFilesByExt(project, "kt", module.moduleContentScope)
            .mapNotNull { it.toPsiFile(project) as? KtFile }
            .flatMap { ktFile -> ktFile.collectDescendantsOfType<KtClass>() }
            .forEach { ktClass -> println(ktClass.name) }
    }
}
```

아래의 스크린샷은 기본 예제 프로젝트의 `MyProjectService.kt`파일을 수정한 뒤에 `Run Plugin`을 통해 실행된 테스트용 sandbox IDE에서 동일한 프로젝트를 열었을 때 바깥쪽 IDE에 값들이 출력되는 모습입니다.

![KtClass 이름 출력](/assets/images/intellij-plugin/ktclass.png)

Sandbox IDE에서 열린 프로젝트의 `KtClass` 이름들이 아래쪽의 콘솔 창에 찍힌 것을 확인해 볼 수 있습니다.
다만 프로젝트가 열리는 시점에는 indexing이 끝나지 않아 모듈이나 파일 목록들이 아직 구성되지 않은 상태일 수도 있기 때문에 위와 같이 `DumbService.getInstance(project).runWhenSmart()`를 사용해서 indexing이 완료된 후에 실행될 수 있도록 했습니다.

### 특정 프로퍼티 가져오기
그렇다면 `KtClass`안에 정의된, `bindView`를 사용하는 프로퍼티와 연결된 XML ID는 어떻게 가져올 수 있을까요?
클래스 안에 정의된 프로퍼티를 가져와서 그 PSI tree를 보면서 `bindView`를 사용하고 있는지, 그리고 어떤 ID를 사용하는지 확인해 보면 됩니다.
현재 활성화된 파일의 PSI tree가 어떤 식으로 구성되어 있는지 간단하게 확인해 보기 위해 [이 문서](https://plugins.jetbrains.com/docs/intellij/explore-api.html#31-use-internal-mode-and-psiviewer)에 나와 있는 것처럼 IntelliJ Plugins Marketplace에 있는 *PsiViewer* 플러그인을 사용했습니다.

해당 플러그인을 사용하면 아래와 같이 현재 커서가 있는 곳의 PSI element가 전체 트리의 어떤 위치에 있는지 파악하는 것이 가능합니다.
![PsiViewer plugin](/assets/images/intellij-plugin/psi-viewer.png)

이러한 기능을 바탕으로 PSI tree를 확인해서 우리가 수정할 대상인 `bindView` delegate를 사용하는 프로퍼티 목록을 가져올 수 있습니다.
가져오는 방법은 여러 가지가 있겠지만, 저는 아래와 같은 코드로 접근했습니다.
```kotlin
fun KtClass.getBindViewProperties() = getProperties()
    .mapNotNull { property ->
        val bindViewCall = property.delegate
            ?.expression
            ?.castSafelyTo<KtCallExpression>()
            ?.takeIf { it.referenceExpression()?.text == "bindView" }
        val firstArgument = bindViewCall?.valueArgumentList?.arguments?.first()
        val xmlId = firstArgument?.getArgumentExpression()?.lastChild?.text
        xmlId?.let { BindViewProperty(property, it) }
    }

data class BindViewProperty(val property: KtProperty, val xmlId: String)
```

### 프로퍼티의 이름을 변경하기
위쪽 단락에서 받아온 Kotlin PSI의 `KtProperty` 타입은 `PsiNamedElement`를 implement한 타입입니다.
따라서 아래와 같이 함수를 만들어서 reference를 포함한 모든 장소의 이름을 변경하고, 그 변경사항을 반영할 수 있습니다.

```kotlin
fun PsiNamedElement.renameAllReferences(project: Project, newName: String) {
    updateAndCommit(project) {
        val files = ReferencesSearch.search(this).map {
            it.handleElementRename(newName)
            it.resolve()?.containingFile
        }
        this.setName(newName)
        println("[Rename] ${this.elementType} ${this.name} -> $newName")
        files.plus(containingFile).filterNotNull()
    }
}

fun updateAndCommit(project: Project, action: () -> Iterable<PsiFile>) {
    DumbService.getInstance(project).runWhenSmart {
        WriteCommandAction.runWriteCommandAction(project) {
            val filesToCommit = action()
            filesToCommit.toSet().forEach { it.commitAndUnblockDocument() }
        }
    }
}
```

View binding에서는 snake case 대신에 lower camel case를 사용하므로, 실제 코드에서는 아래와 같이 간단한 변환 함수를 활용해서 `renameAllReferences()`를 호출해 주었습니다.

```kotlin
fun String.snakeToLowerCamelCase(): String =
    split('_').joinToString("", transform = String::capitalize).decapitalize()
```

그 밖에도 `PsiElement.astReplace`를 활용하면 이름을 변경하는 것을 넘어서 직접 AST를 조작하는 것 또한 가능합니다.
가령 아래와 같이 임의의 `PsiElement`를 white space로 변경시킬 수 있습니다.
```kotlin
psiElement.astReplace(PsiWhiteSpaceImpl(text))
```

## Action으로 등록해서 사용
앞서 말한 동작들이 프로젝트 로딩 시점마다 매번 실행되는 것은 플러그인이라는 특성상 그다지 바람직하지 않은 일입니다.
따라서 IntelliJ에서는 [action](https://plugins.jetbrains.com/docs/intellij/basic-action-system.html)을 등록할 수 있게 해 두었습니다.
`plugins.xml`에 아래와 같이 작성하고 `Run Plugin`을 돌려서 켜진 sandbox IntelliJ를 확인해 보면, 상단의 Tools 메뉴 가장 위에 action이 등록된 것을 볼 수 있습니다.
```xml
<actions>
    ...
    <action class="path.to.the.action.class" description="..." id="..." text="...">
        <add-to-group anchor="first" group-id="ToolsMenu" />
    </action>
</actions>
```

이제 `class`에 지정한 클래스로 가서 action에서 수행할 내용을 적어주면 됩니다.
저희 플러그인에서는 아래 코드처럼 커서가 있는 `KtClass`에 대해서만 리팩토링을 수행하도록 했습니다.

```kotlin
class BindViewRefactoring : AnAction() {
    override fun update(event: AnActionEvent) {
        // Set the availability based on whether a project is open
        event.presentation.isEnabledAndVisible = event.project != null
    }

    override fun actionPerformed(event: AnActionEvent) {
        val elementAtCursor = event.getData(CommonDataKeys.PSI_FILE)
            ?.findElementAt(event.getData(CommonDataKeys.CARET)?.offset ?: 0)
        val targetElement = elementAtCursor?.parentOfType<KtClass>()

        Messages.showOkCancelDialog(
            event.project,
            "리팩토링 대상: ${targetElement?.name}",
            "View Binding",
            "실행",
            "취소",
            Messages.getInformationIcon()
        ).let {
            if (it == Messages.OK && targetElement is KtClass) {
                refactorBindViewProperties(project!!, targetElement)
            }
        }
    }
}
```

---
# 마무리하며
이러한 과정을 거쳐서 작성한 플러그인 코드를 빌드 하여 Android Studio에 설치하고, 리팩토링에 빠르게 사용해 볼 수 있었습니다.
![Android Studio에 설치한 쏘카 플러그인](/assets/images/intellij-plugin/plugin.png)

또한 작성한 플러그인의 기능에는 전처리/후처리를 좀 더 편하게 할 수 있도록 위에서 언급했던 프로퍼티 변경 기능 외에도 아래와 같은 기능들을 추가했습니다.
- View binding migration 페이지에 있는 것처럼 레이아웃을 `R.layout.~` resource 대신 `...Binding` 클래스로부터 받아와서 초기화하는 코드 삽입 기능
- IntelliJ에서 제공하는 `OptimizeImportsProcessor`, `ReformatCodeProcessor` 등을 사용해서 수정한 코드를 다시 정리하는 기능

덕분에 `Activity`, `Fragment`, custom view 등 100개가 넘는 파일에 있던 `bindView` 프로퍼티들을 한꺼번에 수정할 수 있었습니다.
PR에서 코드 리뷰 과정을 거치는 도중, view binding 초기화 코드를 수정하면 좋겠다는 의견이 있어서 이를 전체적으로 반영할 때에도 하나하나 파일을 찾아가며 고칠 필요가 없던 것도 큰 이득이었습니다.

그뿐만 아니라 현재는 이 플러그인을 확장해서 live template으로 하기에는 까다로운 템플릿 코드 기능을 추가하는 등, 더 다양한 형태로 활용하고 있습니다.
이런 식으로 앞으로도 IntelliJ 플러그인을 통해서 개발자들의 소중한 개발 시간을 조금이나마 절약해 볼 수 있으면 좋겠습니다.

## P.S.
2021년 11월 말, JetBrains에서 차세대 IDE [Fleet](https://www.jetbrains.com/fleet/)을 발표했습니다.
짧은 지원자 신청 기간을 거쳐 현재는 closed preview를 진행 중인데요, Fleet에서 plugin 지원은 어떻게 진행할지, language server에 대한 정책은 어떻게 바뀔지 흥미롭습니다.
비록 플러그인을 작성하는 방식이 기존과 달라질 수도 있겠지만, 여기서 진행했던 리팩토링 자동화 경험에 약간의 변주만 더한다면 수월하게 작업할 수 있으리라 생각합니다.
