---
layout: post
title: "Map의 확장 인터페이스 이야기"
subtitle: "NavigableMap"
date: 2021-08-17 09:00:00 +0900
category: api
background: ''
author: ray
comments: true
tags:
  - java collections
  - kotlin collections
  - standard api
---

어떤 항목 중에서 임의의 선택을 하되 가중치를 두어야 한다면 어떻게 구현할 수 있을까요?
예를 들어 빨강, 파랑, 초록 중 하나를 임의대로 추출하되 각각 5,3,2 만큼 가중치를 두고 싶습니다.

뜬금없이 가중치 문제로 시작했는데, 이 글에서 제시하는 답은 아래에 있습니다.
일단 각자 아는 방법대로 구현해 보고 제 방법과 비교해 보시기 바랍니다.

# Map 인터페이스와 세 가지 기본 구현체: HashMap, LinkedHashMap, TreeMap

요즈음에는 인터페이스와 구현체를 구분해서 변수를 선언하는 방법이 거의 자리를 잡은 듯 합니다.
다른 언어는 모르겠지만 자바로 제한한다면 예전에는 선언과 구현체 타입을 같게 지정하는 경우가 많았습니다.

초기에 인터페이스 없이 달랑 구현체만 있던 `Hashtable`이야[^1] 어쩔 수 없이

```java
Hashtable hash = new Hashtable();
```

처럼 했을 수 있지만 컬렉션 프레임워크가 생긴 1.2 시절에도 이런 스타일은 흔히 볼 수 있었습니다.

```java
HashMap map = new HashMap();
```

그러다가 업계의 개발 지식이 전반적으로 높아진 덕분인지 설계와 구현을 분리한 선언을 하는 방법이 거의 필수처럼 자리잡기 시작한 듯 합니다.
이제는 앞선 맵 할당을 다음과 같이 하는 경우가 대부분입니다.

```java
Map map = new HashMap();
```

우리 관심사는 map으로서 기능, 그러니까 어떤 키로 대응하는(map) 값을 찾는 기능이지 맵의 구현체가 실제로 어떠한지는 관심 대상이 아닙니다[^2].
그런데 때로는 map의 키들이 어떤 순서를 유지하는지도 중요한 경우가 있습니다.
이런 때 사용할 수 있도록 준비해 놓은 구현체가 `LinkedHashMap`, `TreeMap` 입니다.
이런 경우에도 인터페이스를 사용한 선언에 구현체를 바꿔 주기만 하면 이를 사용하는 다른 곳에서는 특별히 구현을 바꾸지 않더라도 다른 map 구현체의 특징을 활용할 수 있습니다.

`HashMap`은 저장하는 키에 특별한 순서를 보장하지 않습니다.
사실 대부분 map을 사용하는 목적 상 키 순서가 중요한 경우는 별로 없습니다.
키가 어떤 규칙에 따라 정렬이 되기를 원한다면 `TreeMap`을 사용하면 됩니다.
map에 저장하는 순서에 따라 정렬이 되기를 원하면 `LinkedHashMap`을 사용하면 됩니다.
보통은 이렇게 키의 정렬 순서를 특정하는 목적으로 세 가지 구현체를 상황에 맞게 사용하는 경우가 대부분입니다.
그리고 많은 사람들이 관심있는 동시성 처리와 관련한 `ConcurrentHashMap` 같은 특수한 환경 아래 동작하는 구현체 이야기는 쉽게 찾아 볼 수 있는데, 이 글에서는 조금 다른 이야기를 해 보려고 합니다.

## Map 기능을 확장한 몇 가지 서브 인터페이스

java api doc(v9)를 보면 `Map`의 서브 인터페이스는 11개 씩이나 됩니다.
이 중에서 “map” 하고 관계 있어 보이는 대상은 7개입니다: `ConcurrentMap`, `ConcurrentNavigableMap`, `NavigableMap`, `ObservableMap`, `ObservableMapValue`, `SortedMap`, `WritableMapValue`

7가지 뿐만 아니라 11가지 인터페이스 모두를 다루었으면 좋겠지만 이 글에서는 이 중에서 `NavigableMap`이 무엇인지, `NavigableMap`의 구현체는 무엇인지를 이야기합니다.

# 오늘의 주제: NavigableMap

`NavigableMap`을 설명하는 내용부터 약간 살펴 보겠습니다.
([https://docs.oracle.com/javase/9/docs/api/java/util/NavigableMap.html](https://docs.oracle.com/javase/9/docs/api/java/util/NavigableMap.html))

문서를 보면, 첫 줄에

> A `SortedMap` extended with navigation methods returning the closest matches for given search targets.

이라고 했습니다.
이 문장이 `NavigableMap`의 특징을 설명하는 전부입니다.
이어서 다음 문장을 읽어 보겠습니다.

> Methods `lowerEntry(K)`, `floorEntry(K)`, `ceilingEntry(K)`, and `higherEntry(K)` return `Map.Entry` objects associated with keys respectively less than, less than or equal, greater than or equal, and greater than a given key, returning null if there is no such key.

이 문장에서 어떻게 사용을 하는지 잘 설명하고 있습니다.
그런데 이 내용만 가지고 당장 어디에 써야겠다고 생각이 떠오르기는 쉽지 않습니다.
사실 저도 그러했습니다.
`NavigableMap`이 무엇인지는 오래 전에 공부했지만 특별히 쓸 만한 곳은 없었습니다.
그러다가 최근 어떤 문제를 해결하려고 고민하던 중 문득 이 자료구조가 떠올랐고, 테스트 코드를 몇 줄 구현해 보면서 다행히 생각대로 잘 풀렸습니다.
공부한지 한참 지난 후 이제서야 쏘카에서 처음으로 실제 필요한 곳에 사용할 기회가 생긴 셈이지요.

조금 더 API 문서를 훑어 보겠습니다.

이 “인터페이스"는 `SortedMap` 인터페이스를 상속했습니다.

```java
public interface NavigableMap<K,V> extends SortedMap<K,V>
```

`SortedMap`은 `Map`을 상속했습니다.
그렇다면 `NavigableMap` 이전에 `SortedMap`을 설명해야 할 수도 있겠지만, `NavigableMap`을 설명하면서 자연스럽게 이야기를 하게 되지 않을까 생각합니다.

“인터페이스"를 강조했습니다.
구현체가 아닌 인터페이스입니다.

인터페이스니까 어떤 동작을 제공하는지 계속해서 API문서를 살펴 보겠습니다.
사실 앞서 보았던 두 번째 문장이 거의 전부입니다.

jdk 버전 9 기준 `NavigableMap` 고유 메서드는 21개 이지만 이 중에서도 개인적으로 `NavigableMap`의 특징을 잘 드러낸다고 생각하는 `floorEntry()`, `ceilingEntry()`, `lowerEntry()`, `higherEntry()` 네 가지를 설명하겠습니다.

## NavigableMap 인터페이스 구현체

앞에도 강조했지만 `NavigableMap`은 인터페이스입니다.
그렇다면 구현체는 무엇일까요?

그냥 `TreeMap`을 사용하면 됩니다.

```java
Map m1 = new TreeMap();
NavigableMap m2 = new TreeMap();
```

약간 혼란을 느끼는 분이 있을 수도 있는데, 그래서 앞에 “인터페이스"를 강조했습니다.
같은 `TreeMap` 구현체를 어떤 인터페이스로 사용하느냐에 따라 관점이 달라집니다.
어쩌면 이 이야기가 더욱 중요합니다.
우리의 관심사가 단순히 (키가 정렬된) `Map`인지, 제시한 요소와 방향에 가장 가까운 키를 찾으려는지에 따라 같은 `TreeMap` 구현체라 해도 사용하는 api가 다릅니다.

`Map` 인터페이스로 들여다 본 `TreeMap`이라면 25개의 메서드(인스턴스 메서드 기준)만 사용할 수 있으나 `NavigableMap`은 여기에 더해 21개의 고유한 메서드를 더 사용할 수 있습니다[^4].

당연하게도 이러한 선언은 불가능합니다.

```java
NavigableMap m = new HashMap();
NavigableMap m = new LinkedHashMap();
```

`HashMap`, `LinkedHashMap`은 `NavigableMap` 인터페이스를 상속하지 않은 구현체입니다.

`Map`의 메서드는 25개나 되지만 사실상 `get(key)`, `put(key, value)` 말고는 사용하는 일이 거의 없습니다.
비슷하게 NavigableMap의 21개 메서드 중 4가지 메서드의 특징만 살펴보면 NavigableMap을 충분히 사용할 수 있으리라고 생각합니다[^3].

## 간단한 사용 예제 1: 숫자 범위에 따른 값 할당

매우 간단한 예제를 살펴 보겠습니다[^5].

(1부터 100까지) 숫자 범위에 따라 다른 방을 배정하려고 합니다.
if - else 구문을 사용하면 이 정도 되겠지요?

```kotlin
return if (num > 75) { // num <= 100
    "봄"
} else if (num > 50 && num <= 75) {
    "여름";
} else if (num > 25 && num <= 50) {
    "가을";
} else  { // num > 0
    "겨울";
} 
```

나쁘지 않습니다.
아마도 여러분은 더 좋은 구현 방법을 알고 있으리라 생각합니다.

`NavigableMap`으로 표현하면 어떨까요?
지금까지 했던 이야기를 바탕으로 이미 구현 방법을 떠올린 분도 계실 수 있다고 생각합니다.

```kotlin
val m: NavigableMap<Int, String> = TreeMap()
m[100] = "봄"
m[75]  = "여름"
m[50]  = "가을"
m[25]  = "겨울"

return = m.ceilingEntry(num).getValue()
```

대략 이 정도 코드인데 전반적인 느낌을 이해하기는 어렵지 않으리라 생각합니다[^6].
ceiling 뜻 그대로, 전달한 인자보다 큰 쪽으로 가장 인접한 키값을 찾습니다.

> Returns the least key greater than or equal to the given key, or null if there is no such key.

만약 `num = 69` 였다면 69에 해당하는 키는 없지만 69보다 크면서 가장 가까운(작은) 키인 75를 선택합니다.
즉, (75, "여름")이 해당하는 엔트리입니다.

`floorEntry()`는 반대로 전달한 인자보다 작은 쪽으로 가장 인접한 값을 찾겠지요?
물론 위/아래로 범위를 넘어서는 값을 주면 찾을 수 있는 값이 없습니다.
api 문서를 보면 이 경우 `null`을 반환한다고 명시했습니다.
위의 예제에서는 `m.ceilingEntry(101)`이나 `m.floorEntry(0)`이 `null`입니다.
이런 경우 상/하한 경계를 찾을 수 있는 장치로 `higherEntry()`, `lowerEntry()`가 있습니다.

`NavigableMap`이 가능하려면 어떤 전제 조건이 필요합니다.
바로 "키를 어떤 순서대로 정렬해 놓아야 한다"겠지요.
`NavigableMap`의 상위 인터페이스가 `SortedMap`이기도 하지만 구현체인 `TreeMap`이 이러한 전제 조건을 만족하는 구현체입니다.
다른 `Map` 구현체를 만들기 보다는 `TreeMap`에 floor/ceil 등의 구현을 추가하는 편이 훨씬 합리적이었으리라 생각합니다.

그렇다면 애초에 `TreeMap`으로 선언했다면 키가 정렬된 `Map`으로서 기능 뿐만 아니라 `SortedMap`, `NavigableMap`의 모든 기능을 다 쓸 수 있었을 텐데 왜 `TreeMap`으로 쓰지 않을까요? 이 내용은 글의 주제를 벗어나므로 여기서 다루지는 않겠지만 글 후반부에 약간 언급하겠습니다.

`TreeMap`을 그저 키를 정렬한 맵 정도로 알고 있었다면 지금부터는 `SortedMap`과 `NavigableMap`이라는 다른 모습도 있다는 사실을 알게 되셨으리라 생각합니다.

조금 더 쓸모있는 예제를 살펴 보겠습니다.

## 간단한 사용 예제 2: 가중치 문제 풀이

글 처음에 보여드린 문제 풀이입니다.
여러분은 어떤 방법으로 풀어 보셨나요? 해결 방법은 다양하지만 `NavigableMap`을 활용하면 비교적 간단하게 풀 수 있습니다.
비율이 2:3:5 이므로 키를 각각 2, 5(2+3), 10(2+3+5)으로 정합니다.
약간은 계산이 필요하네요.

조금 더 이해하기 쉽도록 코드 위에 주석을 활용해 간단한 그림으로 표현했습니다.

```kotlin
// | <-------5-------- | <---3---- | <-2-- |
// |         R         |     G     |   B   |
// 10  9   8   7   6   5   4   3   2   1  (0)

val map: NavigableMap<Int, String> = TreeMap()
map[10] = "빨강"
map[5]  = "파랑"
map[2]  = "초록"
```

그리고 `Random`을 써서 이 맵을 탐색합니다.

```kotlin
val color = map.ceilEntry(Random.nextInt(1, 10)).getValue()
```

이 문제 같은 경우에는 간격이 중요합니다.
간격만큼 랜덤으로 선택한 값이 걸릴 확률, 즉 가중치를 결정합니다.
선택한 값에 따라 경계값으로 설정한 키를 찾아가는 구조입니다.

* 1, 2 가 나온다면: "파랑",
* 3, 4, 5 라면: "초록",
* 6, 7, 8, 9, 10 은: "빨강"

을 반환하겠지요?
문제에서 요구한 5:3:2 가중치로 항목을 선택합니다.

매우 직관적이면서도 간결한 구현입니다.

## NavigableMap 전제 조건: SortedMap

만약 `NavigableMap`을 구현한다고 가정한다면 키 항목 구성은 어떤 전제 조건이 필요할까요?
`NavigableMap`에서 제시하는 키를 찾는 방법을 다시 살펴보면, 어떤 값을 주고 이 값 보다 크거나 작은 방향으로 가장 가까이 위치한 키를 찾습니다.
즉, 키 요소를 어떤 기준에 따라 정렬해 놓지 않으면 안 됩니다.
그래서 `NavigableMap`의 상위 인터페이스로서 `SortedMap`은 매우 자연스런 모습입니다.
java에서는 `SortedMap`이나 `NavigableMap` 구현체가 따로 존재하지 않고 `TreeMap` 하나로 이 두 가지 인터페이스를 모두 만족하도록 구현해 놓았습니다.

`NavigableMap`은 `Map`, `SortedMap`을 확장(상속)한 인터페이스라서 `Map`과 `SortedMap`에서 제공하는 api(메서드)를 모두 사용할 수 있겠지요.
`SortedMap`으로 선언하든 `NavigableMap`으로 선언하든 어차피 `TreeMap`을 할당해야 할 테니 구현체 차이는 없겠지만 그래도 사용 목적에 따라 적절한 인터페이스를 사용해야 합니다.
`SortedMap`이 필요한 구현인데 굳이 `NavigableMap`으로 선언해서 동료에게 혼란을 초래할 필요는 없겠지요.
같은 `TreeMap`을 할당하더라도 정말로 필요한 목적에 따라 인터페이스를 적절히 구분해 사용했을 때 이후 코드를 보게 될 동료들이 조금이나마 모호한 상황을 훨씬 적게 겪을 수 있습니다.
그저 정렬된 키셋이 필요한 경우라면 `Map`인터페이스로 충분합니다.
정렬된 키를 기준으로 이런저런 부분 맵이 필요하다면 `SortedMap` 인터페이스가 적절하고 키 탐색이 필요하면 `NavigableMap`을 사용합니다.
그냥 `TreeMap`으로 선언했다면 코드를 자세히 들여다보지 않는 이상 어떤 목적으로 사용하려 했는지 알기가 어렵겠지요.
그나마 주석을 달아 놓았다면 조금은 덜 혼란스러울 수도 있겠습니다.

`SortedMap`도 흥미로운 주제이니 각자 살펴 보시기 바랍니다.
저는 아직 `SortedMap`을 써야 할 만한 문제를 겪지는 않았습니다.
언젠가는 이를 써서 해결해야 할 문제를 마주하게 될 날도 오겠지요.

# 정리하며

map은 상당히 많이 사용하는 자료 구조입니다.
대부분 `HashMap` 정도면 충분하지만 때로는 키 정렬 순서 때문에 `LinkedHashMap`, `TreeMap`을 사용하는 경우가 종종 있습니다.
비슷한 `HashMap` 계열에서도 동시성 문제 때문에 `ConcurrentHashMap`을 사용하는 경우도 있고 메모리 문제 때문에 `WeakHashMap`을 사용하는 경우도 있지만 상황에 따라 구현체를 바꾸었을 뿐 `Map` 인터페이스 범주를 벗어나지 않는 선에서 사용하는 경우가 대부분입니다.

내부 구현의 차이가 있지만 단순한 키 - 값 대응만 사용하는 `Map` 을 넘어서 `Map`의 확장 인터페이스인  `NavigableMap` 관점으로 `TreeMap` 구현체를 어떻게 사용할 수 있는지 간략히 살펴 보았습니다.

다음에는 `NavigableMap`으로 조금 더 깊이 있는 활용 사례를 선보이는 시간을 갖도록 하겠습니다.

[^1]: 엄밀히 이야기하면 `Hashtable`은 추상 클래스인 `Dictionary`를 상속한 구현체입니다. 자바 설계 당시 인터페이스 개념은 추상 클래스보다 늦게 도입한 개념입니다.

[^2]: 제네릭은 이 글에서 다루는 주제가 아니기도 하고 다룰 필요도 없어서 과감히 생략합니다.

[^3]: 제가 만든 애플리케이션에서도 4개의 메서드만 사용합니다.

[^4]: `SortedMap` 까지 생각하면 조금 차이는 있으나 넘어가겠습니다

[^5]: 여기부터 편의 상 코틀린 코드를 사용합니다.

[^6]: num 입력값을 1~100으로 제한하지 않으면 이 구현에는 문제가 있지만, 조건을 제한하여 실행했다고 가정한다면 문제는 없습니다.
