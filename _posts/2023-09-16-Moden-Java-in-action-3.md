---
title: 모던 자바 인 액션 - 03. 람다 표현식
author: dnya0
date:   2023-09-16 15:25:00 +0900
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

## 람다란 무엇인가?

**람다 표현식**은 메서드로 전달할 수 있는 익명 함수를 단순화한 것이라고 할 수 있다.

- 익명
    - 보통의 메서드와 달리 이름이 없으므로 **익명**이라 표현한다.
    - 구현해야 할 코드에 대한 걱정거리가 줄어든다.
- 함수
    - 람다는 메서드처럼 특정 클래스에 종속되지 않으므로 함수라고 부른다.
    - 하지만 메서드처럼 파라미터 리스트, 바디, 반환 형식, 가능한 예외 리스트를 포함한다.
- 전달
    - 람다 표현식을 메서드 인수로 전달하거나 변수로 저장할 수 있다.
- 간결성
    - 익명 클래스처럼 많은 자질구레한 코드를 구현할 필요가 없다.

람다 표현식을 이용하면 익명 클래스 등 판에 박힌 코드를 구현할 필요가 없다. 결과적으로 코드가 간결하고 유연해진다. 예를 들어 커스텀 Comparator 객체를 기존보다 간단하게 구현할 수 있다.

- 기존 코드

```java
Comparator<Apple> byWeight = new Comparator<Apple>() {
    public int compare(Apple a1, Apple a2) {
        return a1.getWeight().compareTo(a2.getWeight());
    }
};
```

- 람다

```java
Comparator<Apple> byWeight = 
    (Apple a1, Apple a2) -> a1.getWeight.compareTo(a2.getWeight());
```

<br>

람다 표현식은 파라미터, 화살표, 바디로 이루어진다.

![람다 표현식의 구조](https://github.com/dnya0/dnya0/assets/84761609/59ad7b51-f7ac-48ea-9920-6be251777629)

- 파라미터 리스트
    - Comparator의 compare 메서드 파리미터(사과 2개)
- 화살표
    - 화살표 `->`는 람다의 파라미터 리스트와 바디를 구분한다.
- 람다 바디
    - 두 사과의 무게를 비교한다.
    - 람다의 반환값에 해당하는 표현식이다.

<br>

다음은 자바8의 유효한 람다 표현식이다.

```java
(String s) -> s.length() 
// String 형식의 파라미터 하나를 가지며 int를 반환한다.
// 람다 표현식에는 return이 함축되어 있으므로 
// return문을 명시적으로 사용하지 않아도 된다.

(Apple a) -> a.getWeight() > 150
// Apple 형식의 파라미터 하나를 가지며 boolean을 반환한다.

(int x, int y) -> {
    System.out.print("Result: ");
    System.out.println(x + y);
}
// int 형식의 파하미터 2개를 가지며 리턴값이 없다.(void)
// 이 예제에서 볼 수 있듯이 람다 표현식은 여러 행의 문장을
// 포함할 수 있다.

() -> 42 // 파라미터가 없으며 int 42를 반환한다.

(Apple a1, Apple a2) -> 
    a1.getWeight().compareTo(a2.getWeight());
// Apple 형식의 파라미터 2개를 가지며 int를 반환한다.
```

<br>

- 람다의 기본 문법
    - 표현식 스타일 expression style
        - `(parameters) -> expression`
    - 블록 스타일 block-style
        - `(parameters) -> { statements; }`

<br>

## 어디에, 어떻게 람다를 사용할까?

### 함수형 인터페이스

2장에서 만든 `Predicate<T>` 인터페이스로 메서드를 파라미터화할 수 있었다. 바로 `Predicate<T>`가 함수형 인터페이스이다. `Predicate<T>`는 오직 하나의 추상 메서드만 지정하기 때문이다.

```java
public interface Predicate<T> {
    boolean test(T t);
}
```

**함수형 인터페이스**는 정확히 하나의 추상 메서드를 지정하는 인터페이스다. 지금까지 살펴본 자바 API의 함수형 인터페이스로 Comparator, Runnable 등이 있다.

```java
public interface Comparator<T> {
    int compare(T o1, T o2);
}

public interface Ruunable {
    void run();
}

public interface ActionListener extends EventListener {
    void actionPerformed(ActionEvent e);
}

public interface Callable<V> {
    V call() throws Exception;
}

public interface PrivilegedAction<T> {
    T run();
}
```

> 인터페이스는 **디폴트 메서드**를 포함할 수 있다. 많은 디폴트 메서드가 있더라도 **추상 메서드가 오직 하나**면 함수형 인터페이스다.<br>디폴트 메서드: 인터페이스의 메서드를 구현하지 않은 클래스를 고려해서 기본 구현을 제공하는 바디를 포함하는 메서드

람다 표현식으로 함수형 인터페이스의 추상 메서드 구현을 직접 전달할 수 있으므로 **전체 표현식을 함수형 인터페이스의 인스턴스로 취급**(기술적으로 따지면 함수형 인터페이스를 **구현한** 클래스의 인스턴스)할 수 있다. 익명 내부 클래스로도 같은 기능을 구현할 수 있다.

```java
// 람다 사용
Runnable r1 = () -> System.out.println("Hello World 1");

// 익명 클래스 사용
Runnable r2 = new Runnable() {
    public void run() {
        System.out.println("Hello World 2");
    }
};

public static void process(Runnable r) {
    r.run();
}

// Hello World 1 출력
process(r1);
// Hello World 2  출력
process(r2);
// 직접 전달된 람다 표현식으로 'Hello World 3' 출력
process(() -> System.out.println("Hello World 3")); 
```

<br>

### 함수 디스크립터

함수형 인터페이스의 추상 메서드 시그니처<sup>signature</sup>는 람다 표현식의 시그니처를 가리킨다. 람다 표현식의 시그니처를 서술하는 메서드를 **함수 디스크립터**<sup>function descriptor</sup>라고 부른다.

예를 들어 Runnable 인터페이스의 유일한 추상 메서드 run은 인수와 반환값이 없으므로 Runnable 인터페이스는 인수와 반환값이 없는 시그니처로 생각할 수 있다.

<br>

#### 람다와 메서드 호출

조금 이상해보일 수 있지만 아래는 정상적인 람다 표현식이다.

```java
process(() -> System.out.println("This is awesome"));
```

위 코드에서는 중괄호를 사용하지 않았고 `System.out.println`은 void를 반환하므로 완벽한 표현식이 아닌 것처럼 보인다. 그럼 중괄호로 감싸면 어떨까?

```java
process(() -> { System.out.println("This is awesome"); });
```

결론적으로 중괄호는 필요 없다. 자바 언어 명세에서는 void를 반환하는 메서드 호풀과 관련한 특별한 규칙을 정하고 있기 때문이다. 즉 한 개의 void 메서드 호출은 중괄호로 감쌀 필요가 없다.

<br>

#### `@FunctionalInterface`는 무엇인가

`@FunctionalInterface`는 함수형 인터페이스임을 가리키는 어노테이션이다. `@FunctionalInterface`로 인터페이스를 선언했지만 실제로 함수형 인터페이스가 아니면 컴파일러가 에러를 발생시킨다.

<br>

## 람다 활용 : 실행 어라운드 패턴

자원 처리에 사용하는 순환 패턴<sup>recurrent pattern</sup>은 자원을 열고, 처리한 다음에, 자원을 닫는 순서로 이루어진다. 설정<sup>setup</sup>과 정리<sup>cleanup</sup>과정은 대부분 비슷하다. 즉, 실제 자원을 처리하는 코드를 설정과 정리 두 과정이 둘러싸는 형태를 갖는다. 이런 형식의 코드를 **실행 어라운드 패턴**<sup>execute around pattern</sup>이라고 부른다.

```java
public String processFile() throws IOException{
    // try-with-resource 구문을 사용하면 자원을 명시적으로
    //닫을 필요가 없어 간결한 코드를 구현하는 데 도움을 준다.
    try(BufferedReader br = 
        new BufferedReader(new FileReader("data.txt"))) {
        return br.readLine(); // 실제 작업을 하는 행
    }
}
```

중복되는 준비 코드와 정리 코드가 작업 A와 작업 B를 감싸고 있다.

![코드의 구조](https://github.com/dnya0/dnya0/assets/84761609/cdd3e54b-ea6e-42a4-9d25-17771a152dab)

<br>

### 1단계 : 동작 파라미터화를 기억하라

현재 코드는 한 번에 한 줄만 읽을 수 있다. 한 번에 두 줄을 읽거나 가장 자주 사용되는 단어를 반환하려면 어떻게 해야할까? 기존의 설정, 정리 과정은 재사용하고 processFile 메서드만 다른 동작을 명령할 수 있다면 좋을것이다. processFile의 동작을 파라미터화하자. processFile 메서드가 BufferedReader를 이용해서 다른 동작을 수행할 수 있도록 processFile 메서드로 동작을 전달해야 한다.

람다를 이용해서 동작을 전달하자. processFile 메서드가 한 번에 두 행을 읽게 하려면 BufferedReader를 인수로 받아서 String을 반환하는 람다가 필요하다.

```java
String result = processFile((BufferedReader br) ->
                        br.readLine() + br.readLine());
```

<br>

### 2단계 : 함수형 인터페이스를 이용해서 동작 전달

함수형 인터페이스 자리에 람다를 사용할 수 있다. BufferedReader -> String과 IOException을 던질 수 있는 시그니처와 일치하는 함수형 인터페이스를 만들어야 한다.

```java
@Functional
public interface BufferedReaderProcessor {
    String process(BufferedReader b) throws IOException;
}
```

정의한 인터페이스를 processFile 메서드의 인수로 전달할 수 있다.

```java
public String processFile(BufferedReaderProcess p) throws IOException {
    ...
}
```

<br>

### 3단계 : 동작 실행

람다 표현식으로 함수형 인터페이스의 인스턴스의 추상 메서드 구현을 직접 전달할 수 있으며 전달된 코드는 함수형 인터페이스의 인스턴스로 전달된 코드와 같은 방식으로 처리한다. 따라서 ProcessFile 바디 내에서 BufferedReaderProcessor 객체의 process를 호출할 수 있다.

```java
public String processFile(BufferedReaderProcessor p) throws IOException {
    try (BufferedReader br = 
        new BufferedReader(new FileReader("data.txt"))) {
        return p.process(br);
    }
}
```

<br>

### 4단계 : 람다 전달

한 행을 처리하는 코드

```java
// 한 행을 처리하는 코드
String oneLine = 
    processFile((BufferedReader br) -> br.readLine());

// 두 행을 처리하는 코드
String twoLine = 
    processFile((BufferedReader br) -> 
        br.readLine() + br.readLine())
```

<br>

## 함수형 인터페이스 사용

다양한 람다 표현식을 사용하려면 공통의 함수 디스크립터를 기술하는 함수형 인터페이스 집합이 필요하다.

<br>

### Predicate

`java.util.function.Predicate<T>` 인터페이스는 test라는 추상 메서드를 정의하며 test는 제네릭 형식 T의 객체를 인수로 받아 불리언을 반환한다. T 형식의 객체를 사용하는 불리언 표현식이 필요한 상황에서 Predicate 인터페이스를 사용할 수 있다.

```java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
}

public <T> List<T> filter(List<T> list, Predicate<T> p) {
    List<T> results = new ArrayList<>();
    for(T t: list) {
        if(p.test(t)) {
            results.add(t);
        }
    }
    return results;
}

Predicate<String> nonEmptyStringPredicate = (String s) -> !s.isEmpty();
List<String> nonEmpty = filter(listOfStrings, nonEmptyStringPredicate);
```

<br>

### Consumer

`java.util.function.Consumer<T>` 인터페이스는 제네릭 형식 T 객체를 받아서 void를 반환하
는 accept라는 추상 메서드를 정의한다.

```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}

public <T> void forEach(List<T> list, Consumer<T> c) {
    for(T t: list) {
        c.accept(t);
    }
}

forEach(
    Arrays.asList(1,2,3,4,5),
    //Consumer의 accept 메서드를 구현하는 람다
    (Integer i) -> System.out.println(i)
);
```

<br>

### Function

`java.util.function.Function<T, R>` 인터페이스는 제네릭 형식 T를 인수로 받아서 제네릭 형
식 R 객체를 반환하는 추상 메서드 apply를 정의한다. 

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}

public <T, R> List<R> map(List<T> list, Function<T, R> f) {
    List<R> result = new ArrayList<>();
    for(T t: list) {
        result.add(f.apply(t));
    }
    return result;
}

// [7, 2, 6]
List<Integer> l = map(
    Arrays.asList("lambdas", "in", "action"),
    (String s) -> s.length() Function의 apply 메서드를 구현하는 람다
);
```

<br>

### 기본형 특화

자바의 모든 형식에는 참조형과 기본형이 있다. 그러나 제네릭 파라미터에는 참조형만 사용할 수 있다.

- 참조형<sup>reference type</sup>: Byte, Integer, Object, List 등
- 기본형<sup>primitive type</sup>: int, double, byte, char 등

자바에서는 참조형을 기본형으로 변환하는 기능을 제공한다. 이 기능을 **박싱**<sup>boxing</sup>이라 하고 반대 동작을 **언박싱**<sup>unboxing</sup>이라 한다. 또한, 박싱과 언박싱이 자동으로 이루어지는 **오토박싱**<sup>autoboxing</sup>이라는 기능도 제공한다.

```java
List<Integer> list = new ArrayList<>();
for (int i = 300; i < 400; i++) {
    list.add(i);
}
```

하지만 이런 변환 과정은 비용이 많이 소모된다. 박싱한 값은 기본형을 감싸는 래퍼며 힙에 저장된
다. 따라서 박싱한 값은 메모리를 더 소비하며 기본형을 가져올 때도 메모리를 탐색하는 과정
이 필요하다.

자바 8에서는 오토박싱을 피할 수 있도록 특별한 버전의 함수형 인터페이스를 제공한다.

```java
public interface IntPredicate {
    boolean test(int t);
}

IntPredicate evenNumbers = (int i) -> i % 2 == 0;
evenNumbers.test(1000); //참(박싱 없음)

Predicate<Integer> oddNumbers = (Integer i) -> i % 2 != 0;
oddNumbers.test(1000); //거짓(박싱)
```

일반적으로 특정 형식을 입력으로 받는 함수형 인터페이스의 이름 앞에는 `DoublePredicate`, 
`IntConsumer`, `LongBinaryOperator`, `IntFunction처럼` 형식명이 붙는다. Function 인터페이스
는 `ToIntFunction<T>`, `IntToDoubleFunction` 등의 다양한 출력 형식 파라미터를 제공한다.

<br>

#### 예외, 람다, 인터페이스의 관계

함수형 인터페이스는 확인된 예외를 던지는 동작을 허용하지 않기 때문에, 예외를 던지는 표현식을 만들기 위해선 직접 정의하거나 try/catch 블록으로 감싸야 한다.

```java
@FunctionalInterface
public interface BufferedReaderProcessor {
    String process(BufferedReader b) throws IOException;
}
BufferedReaderProcessor p = (BufferedReader br) -> br.readLine();
```

우리는 `Function<T, R>` 형식의 함수형 인터페이스를 기대하는 API를 사용하고 있으며 직접 함수형 인터페이스를 만들기 어려운 상황이다. 이 상황에서는 아래와 같은 방법으로 예외를 잡을 수 있다.

```java
Function<BufferedReader, String> f = (BufferedReader b) -> {
    try {
        return b.readLine();
    }
    catch(IOException e) {
        throw new RuntimeException(e);
    }
};
```

<br>

## 형식 검사, 형식 추론, 제약

### 형식 검사

람다가 사용되는 콘텍스트<sup>context</sup>를 이용해서 람다의 형식<sup>type</sup>을 추론할 수 있다. 어떤 콘텍스트에서 기대되는 람다 표현식의 형식을 **대상 형식**<sup>target type</sup>이라 한다.

```java
List<Apple> heavierThan150g =
    filter(inventory, (Apple apple) -> apple.getWeight() > 150);
```

1. filter 메서드의 선언을 확인한다.
2. filter 메서드는 두 번째 파라미터로 Predicate<Apple> 형식(대상 형식)을 기대한다.
3. Predicate<Apple>은 test라는 한 개의 추상 메서드를 정의하는 함수형 인터페이스다.
4. test 메서드는 Apple을 받아 boolean을 반환하는 함수 디스크립터를 묘사한다.
5. filter 메서드로 전달된 인수는 이와 같은 요구사항을 만족해야 한다.

<br>

### 같은 람다, 다른 함수형 인터페이스

대상 형식이라는 특징 때문에 같은 람다 표현식이더라도 호환되는 추상 메서드를 가진 다른 함수형 인터페이스로 사용될 수 있다.

- 예시 1

```java
Callable<Integer> c = () -> 42;
PrivilegedAction<Integer> p = () -> 42;
```

- 예시 2

```java
Comparator<Apple> c1 =
    (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
ToIntBiFunction<Apple, Apple> c2 =
    (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
BiFunction<Apple, Apple, Integer> c3 =
    (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
```

<br>

### 형식 추론

자바 컴파일러는 람다 표현식이 사용된 콘텍스트(대상 형식)를 이용해서 람다 표현식과 관련된 함수형 인터페이스를 추론한다. 즉, 대상형식을 이용해서 함수 디스크립터를 알 수 있으므로 컴파일러는 람다의 시그니처도 추론할 수 있다.

자바 컴파일러는 다음처럼 람다 파라미터 형식을 추론할 수 있다.

```java
//형식을 추론하지 않음
Comparator<Apple> c = 
    (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());

//형식을 추론함
Comparator<Apple> c =
    (a1, a2) -> a1.getWeight().compareTo(a2.getWeight()); 
```

<br>

### 지역 변수 사용

람다 표현식에서는 익명 함수가 하는 것처럼 **자유 변수**<sup>free variable</sup>를 활용할 수 있다. 이와 같은 동작을 **람다 캡처링**<sup>capturing lambda</sup>

- 자유 변수: 파라미터로 넘겨진 변수가 아닌 외부에서 정의된 변수

람다는 인스턴스 변수와 정적 변수를 자유롭게 캡처(자신의 바디에서 참조할 수 있도록)할 수 있다. 하지만 그러려면 지역 변수는 명시적으로 final로 선언되어 있어야 하거나 실질적으로 final로 선언된 변수와 똑같이 사용되어야 한다.

다음 코드는 컴파일할 수 없는 코드이다.

```java
int portNumber = 1337;
Runnable r = () -> System.out.println(portNumber);
portNumber = 31337;
```

<br>

#### 지역 변수의 제약

인스턴스 변수와 지역 변수는 저장되는 곳이 다르다. 인스턴스 변수는 힙에 저장되는 반면 지역 변수는 스택에 저장된다. 람다에서 지역 변수에 바로 접근할 수 있다는 가정하에 람다가 스레드에서 실행된다면 **변수를 할당한 스레드가 사라져서 변수 할당이 해제되었는데도 람다를 실행하는 스레드에서는 해당 변수에 접근하려** 할 수 있다. 따라서 자바 구현에서는 원래 변수에 접근을 허용하는 것이 아니라 자유 지역 변수의 복사본을 제공한다. 따라서 **복사본의 값이 바뀌지 않아야 하므로 지역 변수에는 한 번만 값을 할당해야 한다는 제약이 생긴 것**이다. 또한 지역 변수의 제약 때문에 외부 변수를 변화시키는 일반적인 명령형 프로그래밍 패턴(병렬화를 방해하는 요소로 나중에 설명한다)에 제동을 걸 수 있다.

<br>

#### 클로저

클로저는 함수의 비지역 변수를 자유롭게 참조할 수 있는 함수의 인스턴스를 말한다. 클로저는 클로저 외부에 정의된 변수의 값에 접근하고, 값을 바꿀 수 있다. 자바 8의 람다와 익명 클래스는 클로저와 비슷한 동작을 한다. 람다와 익명 클래스는 외부에 정의된 변수의 값에 접근할 수 있으나, 지역변수의 값은 바꿀 수 없다는 특징이 있다.

<br>

## 메서드 참조

기존의 메서드를 재활용해서 람다처럼 전달할 수 있다. 

```java
//기존 코드
inventory.sort((Apple a1, Apple a2) ->
    a1.getWeight().compareTo(a2.getWeight()));

//메서드 참조
inventory.sort(comparing(Apple::getWeight));
```

<br>

메서드 참조는 특정 메서드만을 호출하는 람다의 축약형이라고 할 수 있다. 메서드 참조를 이용하면 기존 메서드 구현으로 람다 표현식을 만들 수 있다. 이때 명시적으로 메서드명을 참조함으로써 **가독성을 높일 수 있다**. 메서드명 앞에 구분자(`::`)를 붙이는 방식으로 메서드명을 참조한다.

<br>

### 메서드 참조를 만드는 방법

1. **정적 메서드 참조**

- Integer의 parseInt 메서드는 `Integer::parseInt`로 나타낼 수 있다.

2. **다양한 형식의 인스턴스 메서드 참조**

-  String의 length 메서드는 `String::length`로 표현할 수 있다.

3. **기존 객체의 인스턴스 메서드 참조**

- Transaction 객체를 할당받은 expensiveTransaction 지역 변수가 있고, Transaction 객체에는 getValue 메서드가 있다면, 이를 `expensiveTransaction::getValue`라고 표현할 수 있다.
- 비공개 헬퍼 메서드를 정의한 상황에서 유용하게 사용할 수 있다.

<br>

## 생성자 참조

`ClassName::new`처럼 클래스명과 `new` 키워드를 이용해서 기존의 생성자 참조를 만들 수 있다.

<br>

## 람다 표현식을 조합할 수 있는 유용한 메서드

`Comparator`, `Function`, `Predicate`같은 함수형 인터페이스는 람다 표현식을 조합할 수 있도록 유틸리티 메서드를 제공한다. 간단한 여러 개의 람다 표현식을 조합해서 복잡한 람다 표현식을 만들거나 한 함수의 입력 결과가 다른 함수의 입력이 되도록 두 함수를 조합할 수 있다. 여기서 등장하는 것이 **디폴트 메서드**<sup>default method</sup>이다.

<br>

### Comparator 조합

```java
Comparator<Apple> c = Comparator.comparing(Apple::getWeight);
```

1. 역정렬

```java
inventory.sort(comparing(Apple::getWeight).reversed());
```

2. Comperator 연결

```java
inventory.sort(comparing(Apple::getWeight)
    .reversed() //무게를 내림차순으로 정렬
    .thenComparing(Apple::getCountry)); //두 사과의 무게가 같으면 국가별로 정렬
```

<br>

### Predicate 조합

Predicate 인터페이스는 복잡한 프레디케이트를 만들 수 있도록 negate, and, or 세 가지 메서드를 제공한다.

```java
//기존 프레디케이트 객체 redApple의 결과를 반전시킨 객체를 만든다
Predicate<Apple> notRedApple = redApple.negate();

//두 프레디케이트를 연결해서 새로운 프레디케이트 객체를 만든다.
Predicate<Apple> redAndHeavyApple = 
    redApple.and(apple -> apple.getWeight() > 150);

//프레디케이트 메서드를 연결해서 더 복잡한 프레디케이트 객체를 만든다.
Predicate<Apple> redAndHeavyAppleOrGreen =
    redApple.and(apple -> apple.getWeight() > 150)
            .or(apple -> GREEN.equals(a.getColor()));
```

<br>

### Function 조합

Function 인터페이스는 Function 인스턴스를 반환하는 andThen, compose 두 가지 디폴트 메서드를 제공한다.

- andThen: 주어진 함수를 먼저 적용한 결과를 다른 함수의 입력으로 전달하는 함수를 반환
- compose: 인수로 주어진 함수를 먼저 실행한 다음에 그 결과를 외부 함수의 인수로 제공

```java
Function<Integer, Integer> f = x -> x + 1;
Function<Integer, Integer> g = x -> x * 2;

//수학으로는 write g(f(x)) 또는 (g o f)(x)라고 표현
Function<Integer, Integer> h = f.andThen(g);
int result = h.apply(1); //4를 반환

//수학으로는 f(g(x)) 또는 (f o g)(x)라고 표현
Function<Integer, Integer> h = f.compose(g);
int result = h.apply(1); //3을 반환
```