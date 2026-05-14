---
title: Kotlin Pair와 Triple
author: dnya0
date: 2025-10-11 17:30:00 +0900
categories: [Backend, Kotlin]
tag: [Kotlin]
math: true
mermaid: true
image:
  path: ../assets/img/posts-image/kotlin.png
  alt: Kotlin Image
---

## 🥑 들어가며

그동안 코틀린을 사용하면서 Pair를 많이 사용해왔다. 최근 또 Pair를 사용하려다가 다른 사람들도 많이 사용하는지 궁금하였기 때문이다. 그러던 중 주목할만한 글을 찾았다. [Prefer Data Classes Over Pairs](https://proandroiddev.com/prefer-data-classes-over-pairs-42b8a39e5e37)라는 글인데 클린코드 관점에서 코틀린의 Pair와 Triple에 대해 작성한 글이었다. 꽤 좋은 내용이라 잊지 않기위해 기록해두려 한다.

<br>

## 🚨 지양해야될 클래스: Pair, Triple

> “Don’t ever, for any reason, use a Pair or Triple, for any reason, ever, no matter what, no matter where, or who you are with, or where you are going, or where you’ve been. Ever, for any reason. Whatsoever.”<br>— Michael Scott<br>— Matt Robertson<br><br>어떤 이유로든 어떤 이유로든, 어떤 이유로든, 어디서든, 누구와 함께 있든, 어디로 가든, 어디에 가든 `Pair`나 `Triple`을 사용하지 마세요. 어떤 이유로든 항상. 어쨌든.


### 예시

해당 글에 따르면 `Pair`와 `Tripe`은 lazy data classes라 한다. `lazy`는 좋은 종류가 아니다. 아래의 심플 예시 코드를 보자.

```kotlin
fun getCoffee(): Pair<Roast, Size>
```

이런 코드가 있다고 가정했을 때 사용된다면 아래와 같이 사용될 것이다.

```kotlin
val coffee = getCoffee()
print(“A ${coffee.second}, ${coffee.first} roast coffee.")
```

이 상황에서 "크림을 위한 공간을 남겨두기"라는 요구사항이 들어왔을 때 `Pair`를 사용하는 코드라면 아래와 같이 수정될 것이다.

```kotlin
fun getCoffee(): Triple<Roast, Size, Boolean>

...

val coffee = getCoffee()
print("A ${coffee.second}, ${coffee.first} roast coffee with ${if (coffee.third) "" else "no "}room for cream.")
```

이는 유지보수의 관점에서 최악의 선택이다. 다른 개발자가 저 코드를 수정해야 한다면, 저 코드가 하는 일이 무엇인지 파악하기 위해 `getCoffee()`가 있는 파일까지 들어와야 한다!

```kotlin
data class Coffee(
    val roast: Roast,
    val size: Size,
    val leaveRoom: Boolean
)

...

fun getCoffee(): Coffee

...

val coffee = getCoffee()
print("A ${coffee.size}, ${coffee.roast} roast coffee with ${if (coffee.leaveRoom) "" else "no "}room for cream.")
```

`Pair`와 `Triple`을 사용하는 대신 data class를 만들어 전달하면 유지보수와 확장에 좋은 코드를 작성할 수 있다. 원래의 경우라면

```kotlin
val ounces = coffee.second.ounces - if (coffee.third) 2 else 0
```

`coffee.third`가 무엇인지 알 수 없었지만

```kotlin
val ounces = coffee.size.ounces - if (coffee.leaveRoom) 2 else 0
```

현재는 한눈에 알아볼 수 있다.

### 클린 코드

마지막으로 해당 글에선 클린 코드에 대한 내용을 적으며 `Pair`와 `Triple`을 사용하는 행위가 안티패턴임을 강조하고 있다.

> Pairs and Triples are anti-patterns that should be avoided in preference for data classes.

다른 이들이 읽을 것을 생각하여 내가 **작가**라는 마인드로 개발해야 한다. 깨끗하고, 유지보수에 용이한 코드를 작성해야하며, 쓰기에 더 어려워지더라도 한눈에 파악되는 코드를 작성하자는 내용을 적으며 Pairs를 사용하지 말 것을 한번 더 강조한다.

<br>

이 글은 그동안 내가 작성한 코드에 대해 고민해보게 만든다. 데드라인이 다가오면 구현에 급급해져 일단 구현부터 끝내자는 생각으로 개발을 하게 되었는데, 이는 매우 좋지 않은 습관이다. 이렇게 작성하는 코드는 리팩토링도 쉽지 않다. 개발을 할 때마다 내가 좋은 코드를 작성하고 있는지, 좋은 설계를 하고 있는지 항상 고민하지만 항상 부족한 것 같다.