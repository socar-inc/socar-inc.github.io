---
layout: post
title: "팀내 공통코드 관리 변천사"
subtitle: "공통코드 관리 설계, 공통코드 테이블 꼭 필요 한가요? (feat. gradle plugin & kotlin psi)"
date: 2021-05-27 09:25:00 +0900
category: dev
background: '/assets/images/yan-ots-FF14FKgecyM-unsplash.jpg'
author: dorma
comments: true
tags:
    - 공통코드관리
---

# 공통코드를 어떻게 관리 및 사용 할 것인가?

<br>
저희팀은 쏘카 R&D 본부에서 다양한 백오피스 개발을 담당하고 있습니다. 백오피스 개발에 사용되는 기술 스택은 다음과 같습니다.

```markdown
- Database: MySQL
- Server Framework: SpringBoot
- Frontend Framework: vue.js
- Language: kotlin / javascript(일부는 typescript)
```

개발을 하다보면 상태를 표현하는 값이나 변경 빈도가 낮은 분류(Category)등을 공통코드로 관리하게 됩니다.
프론트엔드(vue.js)에서 일반적으로 `select`나 `raido`를 표시할때 데이터로 사용하고 서버(kotlin)에서는 비지니스 로직을 작성할때 사용하게 되고 DB에도 저장되게 됩니다.

공통코드는 변경 빈도가 높지는 않지만 `DB - 서버 - 프론트엔드`에 걸쳐 넓은 범위에서 다양하게 사용되므로 변경이 일어나면 프로젝트를 구성하는 거의 모든 계층이 영향을 받으므로 적절하게 관리하고 사용되어야 합니다.

이글에서는 저희팀에서 신규 프로젝트 설계 과정에서 공통코드를 관리를 위해 어떤 시도를 했고 지금까지 어떻게 보완해 왔는지 정리해 보려고 합니다.

<br>

## 들어가기 앞서...
사실 공통코드 관리를 어떻게 하면 되느냐?에 대한 결론부터 말씀드리면 `정답은 없다`라고 생각합니다.

비슷한 고민을 하시는 분들에게 저희팀의 시행착오가 참고가 될 수 있을까 해서 저희팀이 공통코드 처리를 위해 어떤 고민을 하고 어떤 방법으로 불편한 점을 해소했는지에 대한 기록을 적어볼까 합니다.

<br>

---

## (1차) 일단 구조를 결정하고 만들어봤습니다.
<br>

![공통코드-1차](/assets/images/common-code1.png)

### 팀에서 처음한 내용들
1. 코드를 어떻게 분류 할 것인가?
   * `부모코드-자식코드`로 분류
2. 코드를 어떻게 저장할 것인가?
   * `DB`에 `부모코드(code_group) / 자식코드(code) 2개의 테이블`을 만들어서 저장.
3. 코드를 어떻게 사용할 것인가?
   * `DB`에 저장시에는 `문자열`로 저장
   * `kotlin`에서 사용할때는 `enum`을 정의해 두고 사용
   * `javascript`에서 사용할때는 서버에서 API로 필요한 코드를 조회해서 사용.
3. 각 사용처의 코드를 어떻게 sync 할것인가?
   * `DB` - 기준 데이터
   * `서버(kotlin)` - DB에 입력된 값을 기준으로 kotlin에 enum을 생성하는 유틸(`codeGenerator`)을 만들어 사용
   * `frontend(javascript)` - 서버에서 필요한 코드를 (DB에서) 조회할 수 있는 API를 만들고 필요시 요청해서 사용.
   * `각 개발자` - DB에 입력하기 위한 Insert 문을 팀 공통 문서(notion)에 관리함.
5. 코드의 이름 규칙은 어떻게 할 것인가?
   * `부모코드`는 이름과 함께 `5자리의 PREFIX`를 부여
   * `자식코드`는 `PREFIX_XXX` 형태의 이름을 사용
   * 대문자 사용.
<br>
<br>

### 결정의 배경
1. [MySQL의 enum 단점](https://velog.io/@leejh3224/%EB%B2%88%EC%97%AD-MySQL%EC%9D%98-ENUM-%ED%83%80%EC%9E%85%EC%9D%84-%EC%82%AC%EC%9A%A9%ED%95%98%EC%A7%80-%EB%A7%90%EC%95%84%EC%95%BC-%ED%95%A0-8%EA%B0%80%EC%A7%80-%EC%9D%B4%EC%9C%A0)이 많으니 사용하지 말고 `문자열`로 씁시다.
   * enum의 순서 변경이 있을 경우 테이블이 잠기고 시간이 데이터의 양에 따라 기하급수적으로 늘어날 수 있습니다.
   * 숫자로 할 경우 DB에서 직접 조회시 코드의 의미를 추측 할 수 없습니다.
2. 코딩시 공통코드를 문자열로 쓰진 않았으면 했습니다.
   * `kotlin` 쪽은 DB에 있는 공통코드를 읽어서 [kotlinpoet](https://square.github.io/kotlinpoet/)을 활용해서 enum 클래스들을 생성하기로 했습니다. (`codeGenerator`)
   * `javascript` 쪽은 서버에서 코드 조회를 위한 API를 만들어 두고 조회해서 사용하기로 했습니다. 하지만 실제 코드 값을 바로 사용할 때는 값을 바로 문자열로 사용 하기로 했습니다. 
     * `javascript`에서도 kotlin의 enum처럼 상수로 선언해 두고 사용하고 싶었으나 신규 프로젝트를 진행하는 과정에서 이루어진 결정이라 javascript에서 사용성은 일부 포기하였습니다.
3. 공통코드 목록을 구글시트에 정리해 두는 방법도 고민하였으나 개발 초기에 공통코드가 공유 되어야 하는 대상이 개발자뿐이라 그냥 insert 문을 notion에 붙여두고 관리하기로 했습니다.
<br>
<br>

### 실제 사용 예시
* 개발자간 공유를 위한 insert 쿼리
  * notion 문서에 올려 두고 공통코드를 변경한 개발자가 직적 해당 sql을 수정 후 팀내에 코드가 변경된 사실을 공유 하는 방식으로 관리.

```sql
# 코드그룹 생성
INSERT INTO `code_group` (`name`, `prefix`, `description`)
VALUES
    ('SETTLEMENT_TYPE', 'STLTP_', '비용 발생 유형');
# 코드 생성
INSERT INTO `code` (`group`, `name`, `label`, `description`,`order`)
VALUES
    ('SETTLEMENT_TYPE', 'STLTP_AUTO', '자동', '', 0),
    ('SETTLEMENT_TYPE', 'STLTP_MANUAL', '수동', '', 1);
```

* 생성된 kotlin enum 코드 <a name="generated_kotlin_code"></a>
  * 공통코드를 변경한 개발자가 generator를 돌려서 kotlin 코드를 생성 후 git에 commit.

```kotlin
object Codes {
    // ... other codes ...
    enum class SettlementType(val label: String, val value: String) {
        AUTO("자동", "STLTP_AUTO"),

        MANUAL("수동", "STLTP_MANUAL");

        companion object {
            fun getByValue(value: String): SettlementType? = values().find { it.value == value }
        }
    }
    // ... other codes ...
}
```

* vue.js에서 사용

```js
// 공통코드 가져오기
const codes = await CodeApi.getByGroups(['SETTLEMENT_TYPE']]);

// 공통코드 값 비교
this.type === 'STLTP_AUTO';

// 공통코드 => 라벨 변환
getCodeLabel = (codes, value) => {
  const code = find(codes, { name: value }) || find(codes, { id: value });
  if (!code) return null;
  return code.label;
};
```
<br>



### 이런게 불편해요.
1. 공통코드 변경시마다 신경써야 할 것들이 너무 많습니다.
   * 추가할 코드를 insert 문으로 작성 해 DB에 입력 및 notion 문서 갱신
   * `codeGenerator`를 돌려서 kotlin 코드 생성
   * kotlin 코드를 git에 commit
   * 코드 변경된 사항을 팀원에게 공유.
   * 다른 팀원들은 로컬 DB에 추가된 코드 추가 & git pull
2. `1.`의 절차가 너무 복잡해서 에러를 만나는 경우가 많습니다.
   * kotlin enum 코드와 DB의 값이 서로 달라서 에러 발생.
     * A개발자가 공통코드 변경이 필요한 개발건을 진행하고 그 코드가 머지된 이후 다른 B개발자가 pull을 받은 후 이어서 다른 개발을 진행하려고 할때 로컬 DB에는 아직 신규 코드가 추가 되지 않아 매번 에러를 확인 후 notion에 있는 변경된 insert 쿼리를 로컬 DB에 실행 하게됩니다.
3. `javascript 공통코드 값 비교` 할때 문자열로 쓰는게 맘에 안듭니다.
   * 코드 컨벤션을 깔끔하게 유지하고 싶어하는 팀원들이 많아기도하고 코딩시에 해당 코드의 실제값을 DB나 kotlin 쪽 코드에서 찾아서 복사해서 써야 하는 것이 불편함.
   * 간혹 복사하지 않고 직접 타이핑시 오타로 인한 버그도 발생.
4. javascript에서 필요할때마다 코드를 API로 가져오는데 너무 자주 필요합니다.. (코드 조회를 위한 API 요청이 너무 많습니다.)
   * 서버 / 프론트엔드 둘다 팀내에서 개발하는데 굳이 이렇게 해야하나요?

<br>

---

## (2차) 불편함을 없애 봅시다.(DB를 빼버리자!)
<br>

![공통코드-2차](/assets/images/common-code2.png)

### 공통코드 변경시마다 DB와 코드를 sync 시키는 작업을 없애봅시다.
* DB에서 직접 데이터를 확인할때 공통코드 테이블을 join해서 `공통코드 -> 라벨`로 변경해서 쿼리 하는 경우가 생각보다 없었습니다.
* 위 케이스를 제외하면 DB를 사용하는 것은 프론트엔드를 위해 서버에서 API로 코드를 내려줄때 DB를 조회해서 내리는 곳 밖에 없었습니다.
* 그렇다면! DB에서 코드 테이블을 삭제해버리고 `codeGenerator로 DB에서 생성한 kotlin code`를 메인 데이터로 사용하는 방법을 시도해 볼 수 있을거 같았습니다.
<br>
<br>

### 코드 조회 API를 DB없이 어떻게 만들면 될까요?
* kotlin 코드를 `reflection`해서 필요한 코드를 추출해서 기존 DB에서 조회해서 반환하던 response와 동일한 값을 내려줍니다.
  * `실제 사용예시`에 있는 `object Codes {}`에 확장함수(`getCodes`)를 하나 붙여줍니다.
  * 대/소문자나 camelCase / snake_case 변환이 필요한 부분은 [google guava](https://github.com/google/guava)의 `CaseFormat`를 활용했습니다.
* 코드 예시(`GitHub Gist`): [reflection을 이용한 공통코드 조회](https://gist.github.com/socar-dorma/161c58fda3b848184c62ae287ca59e4b)
<br>
<br>

### 어떤 문제가 해소되었나요?
* 이제 공통코드를 DB에 넣지 않아도 되고, notion에 insert 쿼리를 관리하지 않아도 됩니다.
* 공통코드를 변경한 사실을 팀내에 따로 전파하지 않아도 git pull만 받으면 됩니다.
* 단, 개발자들은 좋지만... `DB에서 직접 데이터를 보는 분(DBA라거나..) 입장에선 반갑지 않을 수 있습니다.`(;;;)
<br>
<br>

---

## (3차) javascript에서 쓸 코드도 생성해 봅시다.
* **덧. 글이 너무 길어질거 같아 `gradle plugin 만드는 방법 및 사용방법`은 생략된 부분이 많습니다. 대략적인 작업 흐름을 알 수 있는 정도로 작성하였습니다.**

<br>

![공통코드-3차](/assets/images/common-code3.png)

### DB가 사라졌으니 kotlin 코드를 분석해서 javascript 코드를 생성해 봅시다.
* 그런데 팀내에 프로젝트가 한개가 아니니 여러 프로젝트에서 공통으로 쓸 수 있었으면 합니다.
* gradle plugin 형태로 만들어서 각 프로젝트에서 사용 하고자 합니다.
* gradle plugin에서 프로젝트에 있는 kotlin 파일을 접근하려니 reflection으로 접근 하기가 애매합니다.
* gradle plugin이 실행되는 시점에 프로젝트 코드를 `import` 하거나 할 수는 없으니 gradle plugin 입장에선 공통코드가 선언된 파일의 경로을 입력받고 파일을 열어서 kotlin 코드를 파싱해서 쓸 수 밖에 없습니다.
<br>
<br>

### kotlin 코드를 파싱해서 javascript에서 사용할 공통코드 생성하는 gradle plugin을 만듭니다.
* reflection은 이미 class가 로딩된 이후에 [kotlin의 KClass](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect/-k-class/)나 [java의 Class](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html)를 사용하지만 Kotlin PSI의 경우는 전혀 다른 클래스들을 활용하고 별도의 dependency도 필요합니다.

```groovy
implementation "org.jetbrains.kotlin:kotlin-compiler:1.5.0"
implementation "org.jetbrains.kotlin:kotlin-compiler-embeddable:1.5.0"
```
* [(이제 직접 수정하고 있는) 생성된 kotlin enum 코드](#generated_kotlin_code)에 있는 kotlin code를 파싱하는 코드의 대략적인 구조입니다. (**실제 실행되는 코드에서 일부를 발췌한 코드라 바로 실행은 안됩니다.**)
  * javascript / typescript 코드 생성은 문법에 맞게 문자열을 생성 후 파일로 저장했습니다.
  * 코드 예시(`GitHub Gist`): [kotlin psi으로 공통코드 파싱하기](https://gist.github.com/socar-dorma/306453fafc0383869f62adf31cfaba0c)
<br>
<br>

### 만들어진 gradle plugin 이렇게 동작합니다.
* gradle plugin을 적용하고 아래 설정을 추가하면 gradle task(`generate<설정 이름(아래 설정기준management)>Code` 유형의 이름으로 생성됨)가 자동으로 추가됩니다.
  * 각 프로젝트의 [(이제 직접 수정하고 있는) 생성된 kotlin enum 코드](#generated_kotlin_code)의 절대경로와 javascript 파일을 생성할 절대경로를 전달합니다.

```groovy
codeJavascriptGenerator {
    management {
        codeFile = "${project.rootDir}/subprojects/core/src/main/kotlin/kr/socar/plan/Codes.kt"
        outDir = "${project.rootDir}/subprojects/frontend/management/src/api"
    }
}
```

* 공통코드가 수정되면 `./gradlew :<project>:generateManagementCode`를 실행해 주면 javascript에서 사용 할 공통코드 파일이 생성(갱신)됩니다.
<br>
<br>

### 생성된 `typescript` 코드
* kotlin에서 `Codes object`를 사용하는것과 최대한 동일하게 사용 할 수 있도록 코드를 생성했습니다.
* 코드 예시(`GitHub Gist`): [생성된 codes.ts 파일](https://gist.github.com/socar-dorma/a0f2c79be7ffc2cff556c02be80500f0)
<br>
<br>

### javascript(typescript)에서 생성된 공통코드파일 사용은 이렇게 하고있습니다.

```js
// 공통코드 가져오기
import Codes from `/codes`;

// 공통코드 값 비교
this.type === Codes.settlementType.AUTO.name;

// 공통코드 => 라벨 변환
const value = 'STLTP_AUTO';
const label = find(Codes.settlementType.values, { name: value }).label;
```
<br>

---

## 마무리하며...
* 공통코드는 `DB - 서버 - 프론트엔드` 모든 계층에서 폭 넓게 사용되며 추가나 수정이 필요한 경우도 많기때문에 적절하게 관리하고 사용하지 않으면 유지보수나 기능을 추가할때 큰 문제가 되기도 합니다. 그러므로 공통코드는 각 계층에서 사용하는 프로그래밍 언어 각각에서 상수로 정의해서 사용해야 하고 각 계층간에 동기화도 쉬워야 한다고 생각합니다.
* 서두에 적었듯이 `공통코드 관리방법에 정답없다`고 생각합니다.(저희팀도 (4차)수정을 감행 할 수도있습니다.)
* 저희팀은 이런 과정을 거쳐서 이렇게 사용하고 있다라는 경험을 공유드리는 것뿐 당연히 이 방법도 정답은 아닙니다.
* **`DB - 서버 - 프론트엔드`에 걸쳐서 공통코드를 어떤식으로 관리할지 고민하시는 분들에게 조금의 참고가 되었으면 좋겠습니다.**
* `gradle plugin 만드는 과정`은 이 글 주제에서는 중요도가 낮다고 생각되어서 대략적인 내용만 적었습니다.
* `kotlin PSI 사용법`도 생략이 많이 되었는데 사실 이건 저도 딱 구현에 필요한 것만 찾아가며 사용한 정도라 설명이 부족한 부분이 많은 점 양해 바랍니다.