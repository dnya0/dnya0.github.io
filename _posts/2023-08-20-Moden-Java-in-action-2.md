---
title: 모던 자바 인 액션 - 02. 동작 파라미터화 코드 전달하기
author: dnya0
date:   2023-08-20 02:04:00 +0900
categories: [Book, 모던 자바 인 액션]
tag: [Book, 모던 자바 인 액션]
math: true
mermaid: true
image:
  path: https://github.com/dnya0/dnya0.github.io/assets/84761609/be791143-bd96-46a4-b9b4-dd6ed60b3ccf
  alt: book image
---

> [모던 자바 인 액션](https://www.yes24.com/Product/Goods/77125987)을 읽고 정리한 글입니다.
{: .prompt-info }

<br>

## 변화하는 요구사항에 대응하기

### 첫 번째 시도 : 녹색 사과 필터링

기존의 농장 재고목록 애플리케이션에 리스트에서 녹색 사과만 필터링하는 기능을 추가한다고 가정하자. 1장과 마찬가지로 사과 색을 정의하는 다음과 같은 `Color num`이 존재한다고 가정하자.

```java
enum Color { RED, GREEN }
```

다음은 첫 번째 시도 결과 코드이다.

```java
public static List<Apple> filterGreenApples(List<Apple> inventory) {
    // 사과 누적 리스트
    List<Apple> result = new ArrayList<>();
    
    //녹색 사과만 선택
    for(Apple apple : inventory) {
        if(GREEN.equals(apple.getColor())) {
            result.add(apple);
        }
    }

    return result;
}
```

그런데 갑자기 빨간 사과도 필터링하고 싶어졌다면 어떻게 고쳐야할까. `filterGreenApples`를 복사해서 if문의 조건을 빨간 사과로 바꾸는 방법을 선택할 수 있다. 그러나 이 방법으로 빨간 사과를 필터링할 순 있지만 이후에 다양한 색으로 필터링하는 등의 변화에는 적절하게 대응하지 못한다. 이 상황에서 다음과 같은 규칙이 있다.

> 거의 비슷한 코드가 반복 존재한다면 그 코드를 추상화한다.

<br>

### 두 번째 시도 : 색을 파라미터화

색을 파라미터화할 수 있도록 메서드에 파라미터를 추가하면 변화하는 요구사항에 좀 더 유연하게 대응하는 코드를 만들 수 있다.

```java
public static List<Apple> filterApplesByColor(List<Apple> inventory, Color color) {
    List<Apple> result = new ArrayList<>();
    
    for(Apple apple : inventory) {
        if(apple.getColor().equals(color)) {
            result.add(apple);
        }
    }

    return result;
}
```

다음 처럼 구현한 메서드를 호출할 수 있다.

```java
List<Apple> greenApples = filterApplesByColor(inventory, GREEN);
List<Apple> redApples = filterApplesByColor(inventory, RED);
```

그러나 갑자기 '색 이외에도 가벼운 사과와 무거운 사과로 구분할 수 있다면 정말 좋겠네요. 보통 무게가 150그램 이상인 사과가 무거운 사과입니다'라고 요구한다. 다음 코드는 다양한 무게에 대응할 수 있도록 무게 정보 파라미터도 추가한다.

```java
public static List<Apple> filterApplesByWeight(List<Apple> inventory, int weight) {
    List<Apple> result = new ArrayList<>();
    
    for(Apple apple : inventory) {
        if(apple.getWeight() > weight) {
            result.add(apple);
        }
    }

    return result;
}
```

위 코드도 좋은 해결책이라 할 수 있지만 코드가 대부분 중복된다. 이는 소프트웨어 공학의 DRY(Don't repeat yourself, 같은 것을 반복하지 말 것) 원칙을 어기는 것이다. 탐색 과정을 고쳐서 성능을 개선하기 위해 메서드 전체 구현을 바꾸는 일은 엔지니어링적으로 비싼 대가를 치러야 한다.

색과 무게를 filter 메서드로 합치는 방법도 있으나 이 방법은 어떤 기준으로 필터링하는지 구분이 필요하다. 이 방법을 위해 flag를 추가할 수 있다. (그러나 실전에서는 절대 이 방법을 사용하지 말아야 한다.)

<br>

### 세 번째 시도 : 가능한 모든 속성으로 필터링

```java
public static List<Apple> filterApples(List<Apple> inventory, 
                            Color color, int weight, boolean flag) {
    List<Apple> result = new ArrayList<>();
    
    for(Apple apple : inventory) {
        if((flag && apple.getColor().equals(color)) 
                || (!flag && apple.getWeight() > weight)) {
            result.add(apple);
        }
    }

    return result;
}
```

다음처럼 메서드를 사용할 수 있다.

```java
List<Apple> greenApples = filterApples(inventory, GREEN, 0, true);
List<Apple> heavyApples = filterApples(inventory, null, 150, false);
```

이 방법은 `true`와 `false`의 의미를 구분하기 힘들다. 또한 앞으로 요구사항이 바뀌었을 때 유연하게 대응할 수도 없다. (사과의 크기, 모양, 출하지 등으로 필터링하고 싶다면? 녹색 사과 중 무거운 사과를 필터링하고 싶다면?) 결국 여러 중복된 필터 메서드를 만들거나 모든 것을 처리하는 거대한 하나의 필터 메서드를 구현해야 한다.

<br>

## 동작 파라미터화

참 또는 거짓을 반환하는 함수를 **프레디케이트**라고 한다. **선택 조건을 결정하는 인터페이스**를 정의하자.

```java
public interface ApplePredicate {
    boolean test(Apple apple);
}
```

다음 예제처럼 다양한 선택 조건을 대표하는 여러 버전의 ApplePredicate를 정의할 수 있다.

```java
public class AppleHeavyWeightPredicate implements ApplePredicate {
    public boolean test(Apple apple) {
        return apple.getWeight() > 150;
    }
}

public class AppleGreenColorPredicate implements ApplePredicate {
    public boolean test(Apple apple) {
        return GREEN.equals(apple.getColor());
    }
}
```

위 조건에 따라 filter 메서드가 다르게 동작할 것이라고 예상할 수 있다. 이를 **전량 디자인 패턴(strategy design pattern)**이라고 부른다. 전략 디자인 패턴은 각 알고리즘을 캡슐화하는 알고리즘 패밀리를 정의해둔 다음에 런타임에 알고리즘을 선택하는 기법이다. `ApplePredicate`가 알고리즘 패밀리고 `AppleHeavyWeightPredicate`, `AppleGreenColorPredicate`가 전략이다. 이렇게 **동작 파리미터화**. 즉 메서드가 다양한 동작(또는 전략)을 **받아서** 내부적으로 다양한 동작을 **수행**할 수 있다.

이제 `filterApples`에서 `ApplePredicate` 객체를 받아 사과의 조건을 검사하도록 고쳐야 한다.

<br>

### 네 번째 시도 : 추상적 조건으로 필터링

```java
public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
    List<Apple> result = new ArrayList<>();
    for(Apple apple : inventory) {
        if(p.test(apple)) {
            result.add(apple)
        }
    }
    return result;
}
```

<br>

#### 코드/동작 전달하기

만약 농부가 150그램이 넘는 빨간 사과를 검색해달라고 부탁하면 ApplePredicate를 적절하게 구현하는 클래스만 만들면 된다.

```java
public class AppleRedAndHeavyPredicate implements ApplePredicate {
    public boolean test(Apple apple) {
        return RED.equals(apple.getColor()) 
                    && apple.getWeight() > 150;
    }
}

List<Apple> redAndHeavyApples = 
                filterApples(inventory, new AppleRedAndHeavyPredicate());
```

<br>

#### 한 개의 파라미터, 다양한 동작

동작 파라미터화의 강점은 컬렉션 탐색 로직과 각 항목에 적용할 동작을 분리할 수 있다는 것이다. 한 메서드가 다른 동작을 수행할 수 있도록 재활용할 수 있다. 

<br>

## 복잡한 과정 간소화

`ApplePredicate` 인터페이스를 구현하는 여러 클래스를 정의한 다음에 인스턴스화하는 작업은 상당히 번거로우며 시간낭비다.

```java
public class AppleHeavyWeightPredicate implements ApplePredicate {
    public boolean test(Apple apple) {
        return apple.getWeight() > 150;
    }
}

public class AppleGreenColorPredicate implements ApplePredicate {
    public boolean test(Apple apple) {
        return GREEN.equals(apple.getColor());
    }
}

public class FilteringApples {
    public static void main(String...args) {
        List<Apple> inventory = Arrays.asList(
            new Apple(80, GREEN),
            new Apple(155, GREEN),
            new Apple(120, RED)
        );

        List<Apple> heavyApples = 
            filterApples(inventory, new AppleHeavyWeightPredicate());
        List<Apple> greenApples = 
            filterApples(inventory, new AppleGreenColorPredicate());   
    }

    public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
        List<Apple> result = new ArrayList<>();
        for(Apple apple : inventory) {
            if(p.test(apple)) {
                result.add(apple)
            }
        }
        return result;
    }
}
```

로직과 관련 없는 코드가 많이 추가되었다. 자바는 클래스의 선언과 인스턴스화를 동시에 수행할 수 있도록 **익명 클래스(annonymous class)라는 기법을 제공한다.

<br>

### 익명 클래스

익명 클래스는 자바의 지역 클래스(local class)와 비슷한 개념이다. 익명 클래스를 이용하면 클래스 선언과 인스턴스화를 동시에 할 수 있다.

<br>

### 다섯 번째 시도 : 익명 클래스 사용

```java
List<Apple> redApples = filterApples(inventory, new ApplePredicate() {
    public boolean test(Apple apple) {
        return RED.equals(apple.getColor());
    }
});
```

GUI 애플리케이션에서 이벤트 핸들러 객체를 구현할 때 익명 클래스를 종종 사용한다.

```java
button.setOnAction(new EventHandler<ActionEvent>() {
    public void handle(ActionEvent event) {
        System.out.println("Whoooo a click!");
    }
});
```

익명 클래스로도 부족한 점이 있다.

- 익명 클래스는 여전히 많은 공간을 차지한다.
- 많은 프로그래머가 익명 클래스의 사용에 익숙하지 않다.

장황한 코드는 나쁜 특성이다. 구현하고 유지보수하는 데 시간이 오래 걸릴 뿐 아니라 읽는 즐거움을 빼앗는 요소이다.

<br>

### 여섯 번째 시도 : 람다 표현식 사용

자바 8의 람다 표현식을 이용해서 간단하게 재구현할 수 있다.

```java
List<Apple> result = 
        filterApples(inventory, (Apple, apple) -> RED.equals(apple.getColor()));
```

![image](https://github.com/dnya0/dnya0/assets/84761609/ae2b1688-6bea-492f-97d9-e45b3fddcbbd)

<br>

### 일곱 번째 시도 : 리스트 형식으로 추상화

```java
public interface Predicate<T> {
    boolean test(T t);
}

public static <T> List<T> filter(List<T> list, Predicate<T> p) {
    List<T> result = new ArrayList<>();
    for(T e : list) {
        if(p.test(e)) {
            result.add(e)
        }
    }
    return result;
}
```

이제 바나나, 오렌지, 정수, 문자열 등의 리스트에 필터 메서드를 사용할 수 있다.

```java
List<Apple> redApples = 
    filter(inventory, (Apple apple) -> Red.equals(apple.getColor()));

List<Integer> evenNumbers = 
    filter(numbers, (Integer i) -> i % 2 == 0);
```

<br>

## 실전 예제

### Comparator로 정렬하기

```java
//java.util.Comparator
public interface Comparator<T> {
    int compare(T o1, T o2);
}
```

- 익명 클래스 사용

```java
inventory.sort(new Comparator<Apple>() {
    public int compare(Apple a1, Apple a2) {
        return a1.getWeight().compareTo(a2.getWeight());
    }
});
```

- 람다 표현식 사용

```java
inventory.sort((Apple a1, Apple a2) 
        -> a1.getWeight().compareTo(a2.getWeight));
```

<br>

### Runnable로 코드 블록 실행하기

```java
//java.lang.Runnable
public interface Runnable {
    void run();
}
```

- 익명 클래스

```java
Thread t = new Thread(new Runnable() {
    public void run() {
        System.out.println("Hello world");
    }
});
```

- 람다 표현식

```java
Thread t = new Thread(() -> System.out.println("Hello world"));
```

<br>

### Callable을 결과로 반환하기

`ExecutorService` 인터페이스는 태스크 제출과 실행 과정의 연관성을 끊어준다. `ExecutorService`를 이용하면 태스크를 스레드 풀로 보내고 결과를 `Future`로 저장할 수 있다는 점이 스레드와 Runnable을 이용하는 방식과는 다르다.

```java
//java.util.concurrent.Callable
public interface Callable<T> {
    V call();
}
```

- 익명 클래스

```java
ExecutorService executorService = Executors.newCachedThreadPool();
Future<String> threadName = executorService.submit(new Callable<String>() {
    @Override
    public String call() throws Exception {
        return Thread.currentThread().getName();
    }
})
```

- 람다 표현식

```java
Future<String> threadName = 
    executorService.submit(() -> Thread.currentThread().getName());
```

<br>

### GUI 이벤트 처리하기

- 익명 클래스

```java
Button button = new Button("Send");
button.setOnAction(new EventHandler<ActionEvent>() {
    public void handle(ActionEvent event) {
        label.setText("Sent!");
    }
});
```

- 람다 표현식

```java
button.setOnAction((ActionEvent event) -> label.setText("Sent!"));
```

