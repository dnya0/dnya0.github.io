---
title: 모던 자바 인 액션 - 01. 자바 8, 9, 10, 11
author: dnya0
date:   2023-08-11 00:27:00 +0900
categories: [Book, 모던 자바 인 액션]
tag: [Study, 모던 자바 인 액션]
math: true
mermaid: true
image:
  path: https://github.com/dnya0/dnya0.github.io/assets/84761609/be791143-bd96-46a4-b9b4-dd6ed60b3ccf
  alt: book image
---

> [모던 자바 인 액션](https://www.yes24.com/Product/Goods/77125987)을 읽고 정리한 글입니다.
{: .prompt-info }

<br>

자바는 처음부터 많은 유용한 라이브러리를 포함하는 잘 설계된 객체지향 언어로 시작했다. 또한 처음부터 스레드와 락을 이용한 소소한 동시성도 지원했다. 코드를 JVM 바이트 코드로 컴파일하는 특징 때문에 자바는 인터넷 애플릿 프로그램의 주요 언어가 되었다.

그러나 프로그래머는 빅데이터라는 도전에 직면하면서 멀티코어 컴퓨터나 컴퓨팅 클러스터를 이용해서 빅데이터를 효과적으로 처리할 필요성이 커졌다. 즉, 병렬 프로세싱을 활용해야 하는 데 지금까지의 자바로는 충분히 대응할 수 없었다.

자바8은 더 다양한 프로그래밍 도구 그리고 다양한 프로그래밍 문제를 더 빠르고 정확하며 쉽게 유지보수할 수 있다는 장점을 제공한다. 자바8에 추가된 기능은 자바에 없던 완전히 새로운 기능이지만 현재 시장에서 요구하는 기능을 효과적으로 제공한다. 이 책은 병렬성을 활용하는 코드, 간결한 코드를 구현할 수 있도록 자바8에서 제공하는 기능의 모태인 세 가지 프로그래밍 개념을 자세히 설명한다.

<br>

## 세 가지 프로그래밍 개념

- 스트림 처리
- 동작 파라미터화로 메서드에 코드 전달하기
- 병렬성과 공유 가변 데이터

<br>

### 스트림 처리(Stream Processing)

**스트림**은 **한 번에 한 개씩 만들어지는 연속적인 데이터 항목들의 모임**이다. 이론적으로 프로그램은 입력 스트림에서 데이터를 한 개씩 읽어 들이며 마찬가지로 출력 스트림으로 데이터를 한 개씩 기록한다. 즉, 어떤 프로그램의 출력 스트림은 다른 프로그램의 입력 스트림이 될 수 있다.

스트림 API는 조립 라인처럼 파이프라인을 만드는 데 필요한 많은 메서드를 제공한다. 스트림 파이프라인을 이용해서 입력 부분을 여러 CPU 코어에 쉽게 할당할 수 있다는 부가적인 이득도 얻을 수 있다. 스레드라는 복잡한 작업을 사용하지 않으면서도 **공짜**로 병렬성을 얻을 수 있다. 

<br>

### 동작 파라미터화(behavior parameterization)로 메서드에 코드 전달하기

자바8 이전의 자바에서는 메서드를 다른 메서드로 전달할 방법이 없었지만 자바8에서는 메서드를 다른 다른 메서드의 인수로 넘겨주는 기능을 제공한다. 이러한 기능을 **동작 파라미터화**라고 부른다. 해당 개념이 중요한 이유는 스트림 API는 연산의 동작을 파라미터화할 수 있는 코드를 전달한다는 사상에 기초하기 때문이다. **함수형 프로그래밍** 기술을 이용한다.

<br>

### 병렬성과 공유 가변 데이터

스트림 메서드로 전달하는 코드는 다른 코드와 동시에 실행하더라도 안전하게 실행될 수 있어야 한다. 보통 다른 코드와 동시에 실행하더라도 안전하게 실행할 수 있는 코드를 만들려면 공유된 가변 데이터(shared mutable data)에 접근하지 않아야 한다. 이러한 함수를 순수(pure) 함수, 부작용 없는(side-effect-free) 함수, 상태 없는(stateless) 함수라 부른다.

<br>

## 자바 함수

프로그래밍 언어에서 함수라는 용어는 메서드 특히 정적 메서드와 같은 의미로 사용된다. 자바의 함수는 이에 더해 수학적인 함수처럼 부작용을 일으키지 않는 함수를 의미한다. 자바8에서의 함수 사용법은 일반적인 프로그래밍 언어의 함수 사용법과 아주 비슷하다. 

프로그래밍 언어의 핵심은 값을 바꾸는 것이다. 역사적으로 그리고 전통적으로 프로그래밍 언어에서는 이 값을 일급(first-class)값 또는 시민(citizens)이라 부른다. 다양한 구조체(메서드, 클래스 같은)가 값의 구조를 표현하는 데 도움이 될 수 있으나 모든 구조체를 자유롭게 전달할 수는 없다. 이렇게 전달할 수 없는 구조체는 이급 시민이다. 위에서 언급한 값은 일급 시민이나 메서드, 클래스 등은 이급 시민에 해당한다. 인스턴스화한 결과가 값으로 귀결되는 클래스를 정의할 때 메서드를 아주 유용하게 활용할 수 있지만 여전히 메서드와 클래스는 그 자체로 값이 될 수 없다. 런타임에 메서드를 전달할 수 있다면, 즉 메서드를 일급 시민으로 만들면 프로그래밍에 유용하게 활용할 수 있다.

<br>

### 메서드 참조(method reference)

동작의 전달을 위해 익명 클래스를 만들고 메서드를 구현해서 넘길 필요 없이, 준비된 함수를 메서드 참조 `::`를 이용해서 전달할 수 있다. 아래 예제를 통해 자바 8에서는 더 이상 메서드가 이급값이 아닌 일급값인 것을 확인할 수 있다.

- 익명 클래스를 이용한 파일 리스팅

```java
File[] hiddenFiles = new File(".").listFiles(new FileFilter() {
    public boolean accept(File file) {
        return file.hidden();
    }
})
```

- 메서드 참조를 통한 파일리스팅

```java
File[] hiddenFiles = new File(".").listFiles(File::isHidden);
```

<br>

#### 람다 : 익명 함수

자바8에서는 (기명 named) 메서드를 일급값으로 취급할 뿐만 아니라 **람다**(또는 익명 함수 annonymous functions)를 포함하여 함수도 값으로 취급할 수 있다. 람다 문법 형식으로 구현된 프로그램을 함수형 프로그래밍, 즉 '함수를 일급값으로 넘겨주는 프로그램을 구현힌다'라고 한다.

<br>

#### 메서드 넘겨주기

- 자바 8 이전

```java
public class Test {

    public static List<Apple> filterGreenApples(List<Apple> inventory) {
        List<Apple> result = new ArrayList<>();

        for(Apple apple : inventory) {
            if(GREEN.equals(apple.getColor())) {
                result.add(apple);
            }
        }
        return result;
    }

    public static List<Apple> filterHeavyApples(List<Apple> inventory) {
        List<Apple> result = new ArrayList<>();

        for(Apple apple : inventory) {
            if(apple.getWeight() > 150) {
                result.add(apple);
            }
        }
        return result;
    }
}

```

- 자바 8 이후

```java
public class Test {

    public static boolean isGreenApple(Apple apple) {
        return GREEN.equals(apple.getColor());
    }

    public static boolean isHeavyApple(Apple apple) {
        return apple.getWeight() > 150;
    }

    public interface Predicate<T> {
        boolean test(T t);
    }

    static List<Apple> filterApples(List<Apple> inventory, Predicate<Apple> p) {
        List<Apple> result = new ArrayList<>();
        for(Apple apple : inventory) {
            if(p.test(apple)) {
                result.add(apple);
            }
        }
        return result;
    }
}
```

다음처럼 메서드를 호출할 수 있다.

```java
filterApples(inventory, Apple::isGreenApple);
filterApples(inventory, Apple::isHeavyApple);
```

<br>

#### 메서드 전달에서 람다로

람다를 사용하면 다음과 같이 구현할 수 있다.

```java
filterApples(inventory, (Apple a) -> GREEN.equals(a.getColor()));
filterApples(inventory, (Apple a) -> a.getWeight() > 150);
filterApples(inventory, (Apple a) -> a.getWeight() < 80 || RED.equals(a.getColor()));
```

한 번만 사용할 메서드는 따로 정의를 구현할 필요가 없다. 다음처럼 메서드 filter를 이용하면 filterApples 메서드를 구현할 필요가 없다.

```java
filter(inventory, (Apple a) -> a.getWeight() > 150);
```

그러나 병렬성의 이유로 이와 같은 설계를 포기했다. 대신 filter와 비슷한 동작을 수행하는 연산집합을 포함하는 새로운 스트림 API를 제공한다.

<br>

## 스트림

스트림 API를 이용하면 컬렉션 API와는 상당히 다른 방식으로 데이터를 처리할 수 있다. 컬렉션에서는 반복 과정을 직접 처리해야 했다. 즉, for-each 루프를 이용해서 각 요소를 반복하면서 작업을 수행했다. 이런 방식의 반복을 외부 반복(external iteration)이라고 한다. 반면 스트림 API에서는 라이브러리 내부에서 모든 데이터가 처리되기에 루프를 신경 쓸 필요가 없다. 이와 같은 반복을 내부 반복(internal iteration)이라고 한다.

<br>

### 멀티스레딩

기존 버전에서 제공하는 스레드 API로 **멀티스레딩** 코드를 구현해서 병렬성을 이용하는 것은 쉽지 않다. 멀티스레딩 환경에서 각각의 스레드는 동시에 공유된 데이터에 접근하고, 데이터를 갱신할 수 있다. 자바8은 스트림 API로 '컬렉션을 처리하면서 발생하는 모호함과 반복적인 코드 문제' 그리고 '멀티코어 활용 어려움'이라는 두 가지 문제를 모두 해결했다.

스트림은 스트림 내의 요소를 쉽게 병렬로 처리할 수 있는 환경을 제공한다. 컬렉션을 필터링할 수 있는 가장 빠른 방법은 컬렉션을 스트림으로 바꾸고, 병렬로 처리한 다음에, 리스트로 다시 복원하는 것이다. 스트림과 람다 표현식을 이용하면 '병렬성을 공짜로' 얻을 수 있으며 리스트에서 무거운 사과를 순차적으로 또는 병렬로 필터링할 수 있다.

- 순차 처리 방식의 코드

```java
import static java.util.stream.Collectors.toList;

List<Apple> heavyApples =
        inventory.stream().filter((Apple a) -> a.getWeight() > 150)
                .collect(toList());
```

- 병렬 처리 방식의 코드

```java
import static java.util.stream.Collectors.toList;

List<Apple> heavyApples =
        inventory.parallelStream().filter((Apple a) -> a.getWeight() > 150)
                .collect(toList());
```

<br>

#### 자바의 병렬성과 공유되지 않은 가변 상태

- 흔히 자바의 병렬성은 어렵고 synchronized는 쉽게 에러를 일으킨다고 생각한다.
- 자바8은 이를 해결하기 위해 두 가지 방식을 사용
	- 라이브러리에서 분할 처리
		- 큰 스트림을 병렬로 처리할 수 있도록 작은 스트림으로 분할
		- filter 같은 라이브러리 메서드로 전달된 메서드가 상호작용을 하지 않는다면 가변 공유 객체를 통해 공짜로 병렬성을 누릴 수 있음.
- 함수형 프로그래밍에서 함수형의 의미
	- 함수를 일급값으로 사용한다.
	- 프로그램이 실행되는 동안 컴포넌트 간에 상호작용이 일어나지 않는다.

<br>

## 디폴트 메서드와 자바 모듈

자바 9의 모듈 시스템은 모듈을 정의하는 문법을 제공하므로 이를 이용해 패키지 모음을 포함하는 모듈을 정의할 수 있다. 모듈 덕분에 JAR 같은 컴포넌트에 구조를 적용할 수 있으며 문서화와 모듈 확인 작업이 용이해졌다. 또한 자바 8에서는 인터페이스를 쉽게 바꿀 수 있도록 디폴트 메서드를 지원한다. 디폴트 메서드는 특정 프로그램을 구현하는 데 도움을 주는 기능이 아니라 미래에 프로그램이 쉽게 변화할 수 있는 환경을 제공하는 기능이다.

```java
List<Apple> heavyApples =
        inventory.stream().filter((Apple a) -> a.getWeight() > 150)
                .collect(toList());
List<Apple> heavyApples =
        inventory.parallelStream().filter((Apple a) -> a.getWeight() > 150)
                .collect(toList());
```

자바 8 이전에는 List<T>(List가 구현하는 인터페이스인 Collection<T>도 마찬가지)가 stream이나 parallelStream 메서드를 지원하지 않는다. 따라서 위 코드는 컴파일할 수 없는 코드다. 가장 간단한 해결책은 직접 인터페이스를 만들어서 Collection 인터페이스에 stream 메서드를 추가하고 ArrayList 클래스에서 메서드를 구현하는 것이다. 그러나 이 방법은 **인터페이스에 새로운 메서드를 추가한다면 인터페이스를 구현하는 모든 클래스가 새로 추가된 메서드를 구현해야 하는 어려움**이 있다.

자바 8은 구현 클래스에서 구현하지 않아도 되는 메서드를 인터페이스에 추가할 수 있는 기능을 제공한다. 메서드 본문(bodies)은 클래스 구현이 아닌 인터페이스의 일부로 포함한다. 그래서 디폴트 메서드(defalt method)라고 부른다. **디폴트 메서드를 이용하면 기존의 코드를 건드리지 않고도 원래의 인터페이스 설계를 자유롭게 확장할 수 있다.**

<br>

## 함수형 프로그래밍에서 가져온 다른 유용한 아이디어

- 서술형의 데이터 형식을 이용해 null을 회피하는 기법
	- 자바 8에서는 NullPointer 예외를 피할 수 있도록 도와주는 Optional<T> 클래스를 제공한다.
	- (구조적 structural) 패턴 매칭 기법
		- f(0) = 1
		  f(n) = n * f(n - 1) 그렇지 않으면
		- 자바에서는 if-then-else나 switch 문을 이용하나 다른 언어에서 패턴 매칭으로 더 정확한 비교를 구현할 수 있다는 사실을 증명했다.
		- 아쉽게도 자바 8은 패턴 매칭을 완벽하게 지원하지 않는다.


