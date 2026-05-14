---
title: Reflection이란?
author: dnya0
date: 2025-09-21 19:36:00 +0900
categories: [Backend, Spring]
tag: [Study, Spring, Java, Reflection]
math: true
mermaid: true
image:
  path: https://user-images.githubusercontent.com/84761609/220085479-19ee260a-d709-4a47-b788-416275d8a2d8.png
  alt: Spring Image
---

## 🥑 들어가며

스프링을 사용하면서 스프링 공부의 필요성을 느꼈다.. 미뤄뒀던 토비의 스프링을 꺼내 다시 공부하면서 이번엔 대충 공부하는 것이 아닌 더 꼼꼼히 보려 한다. 그렇게 읽던 중 책의 1장에서부터 **리플렉션**에 대해 언급하는 것이었다. 그동안 리플렉션이 무엇인지 감으로 알고있었는데 이번에 제대로 알아보려 한다.

<br>

토비의 스프링에 나온 설명에 따르면, 자바 빈(JavaBean)은 아래의 두 가지 관례를 따라 만들어진 오브젝트라고 한다.

1. 디폴트 생성자(Default Constructor)

   - 툴이나 프레임 워크에서 리플렉션을 이용해 오브젝트를 생성하기 때문에 필요하다.

2. 프로퍼티(Property)

   - 자바 빈이 호출하는 이름을 가진 속성이다.
   - setter와 getter를 이용해 수정 or 조회할 수 있다.

여기에서 리플렉션이란 무엇이고, 어떻게 이 것을 이용해 오브젝트를 생성한다는 걸까? 이에 대해 알려면 클래스(Class)라는 객체에 대해 알아야 한다.

<br>

## 📌 클래스(Class)란?

코드를 작성하다보면 어플리케이션 실행 중 클래스의 이름을 동적으로 불러와 다뤄야할 일이 있다. 이 **클래스(Class) 객체**가 **코드 상에서 호출 로직을 통해 클래스 정보를 얻어와 다룸**으로써 런타임 단에서 다이나믹하게 클래스를 핸들링하게 도와준다.

클래스는 `java.lang.Class` 패키지에 별도로 존재하는 독립형 클래스로서, 자신이 속한 클래스의 모든 멤버 정보를 담고 있기 때문에 런타임 환경에서 동적으로 저장된 클래스나 인터페이스 정보를 가져오는데 사용된다.

- 클래스 객체

```java
public final class Class<T> implements java.io.Serializable,
                              GenericDeclaration,
                              Type,
                              AnnotatedElement,
                              TypeDescriptor.OfField<Class<?>>,
                              Constable {
    private static final int ANNOTATION = 0x00002000;
    private static final int ENUM       = 0x00004000;
    private static final int SYNTHETIC  = 0x00001000;

    private static native void registerNatives();
    static {
        registerNatives();
    }
    ...
}
```

자바의 모든 클래스와 인터페이스는 컴파일 후 `.java` -> `.class` 파일로 변환된다. `.class` 파일은 컴파일된 바이트코드를 담고 있는 파일인데, 이 파일은 클래스나 인터페이스의 모든 정보를 포함하며, **JVM(자바 가상 머신)**이 실행될 때 **클래스 로더(ClassLoader)**에 의해 메모리에 로드된다.

`java.lang.Class` 클래스는 JVM이 클래스를 메모리에 로드할 때, 해당 클래스의 **메타데이터(metadata)**를 담는 객체이다. 즉, JVM이 `.class `파일을 읽어서 메모리(힙 영역)에 로드할 때, 이 클래스의 정보를 담기 위해 자동으로 클래스 객체를 생성한다. 이 객체를 통해 런타임에 클래스의 정보를 얻거나, 새로운 인스턴스를 동적으로 생성할 수 있다.

### 왜 new 없이 생성 가능할까?

클래스 객체는 **JVM이 클래스 로딩 과정에서 자동으로 생성하여 메모리에 단 하나만 존재**한다. 따라서 개발자가 명시적으로 new 키워드를 사용해 클래스 객체를 만들 필요가 없다. **이미 로딩된 클래스에 대한 클래스 객체는 항상 존재**하므로, 아래의 세 가지 방법 중 하나로 해당 객체에 접근하기만 하면 된다. 이처럼 런타임에 클래스 정보를 다루는 기술을 **리플렉션(Reflection)**이라고 한다.

> JVM의 클래스 로더(class loader)는 실행 시에 필요한 클래스를 동적으로 메모리에 로드하는 역할을 한다. 먼저 기존에 생성된 클래스 객체가 메모리에 존재하는지 확인하고 있으면 객체의 참조를 반환한다. 없을 경우 classpath에 지정된 경로를 따라서 클래스 파일을 찾아 해당 클래스 파일을 읽어서 Class 객체로 변환한다. 만일 못 찾으면 `ClassNotFoundException` 예외를 띄우게 된다.
> {: .prompt-danger }

### 클래스 객체를 얻는 방법

#### `getClass()` 메서드 사용

- 모든 클래스의 최상위 클래스인 Object 클래스에서 제공하는 `getClass()` 메서드를 통해 가져온다.
- 단, 해당 클래스가 인스턴스화 된 상태 이어야 한다.

```java
String s = new String();
Class c = s.getClass();
```

#### `.class` 리터럴 사용

```java
Class c = String.class;
```

#### `Class.forName()` 메서드 사용(동적 로딩)

- 클래스의 도메인을 상세히 적어주어야 한다.
  - 클래스 파일 경로에 오타가 있으면 에러가 발생할 수 있다.
- 만일 Class 객체를 찾지 못한다면 ClassNotFoundException를 발생 시키기 때문에 예외처리가 강제된다.
  - 보통 다른 클래스 파일을 불러올때는 컴파일 시 JVM의 Method Area에 클래스 파일이 같이 바인딩(binding)된다.
  - forName()으로 `.class` 파일을 불러올 때는 컴파일에 바인딩이 되지않고 런타임 때 불러오게 되기 때문에 동적 로딩이라고 부른다.
  - 컴파일 타임에 체크 할 수 없기 때문에 예외 처리를 해주어야 한다.
- 다른 두가지 방법보다 **메모리를 절약**할 수 있으며, 동적 로딩 할 수 있기 때문에 가장 성능이 좋다.

```java
try {
    Class c = Class.forName("java.lang.String");
} catch (ClassNotFoundException e) {
    e.printStackTrace();
}
```

<br>

## 📌 리플렉션(Reflection)이란?

**리플렉션(Reflection)**이란 위에서 설명한 것과 같이 **런타임에 클래스 정보를 다루는 기술**을 말한다.

- **자바에서 클래스나 멤버에 대한 정보를 런타임에 조사하고, 검사하거나 조작할 수 있는 기능**
- 코드의 유연성과 확장성을 높일 수 있음
- ex) 클래스의 이름, 메서드, 필드, 생성자 등에 대한 정보를 프로그램 실행 중에 알아내, 이를 통해 객체를 생성 or 메서드를 호출

**예시**

```java
Class<?> clazz = Class.forName("java.lang.String");
Method[] methods = clazz.getDeclaredMethods();

for (Method method : methods) {
    System.out.println(method.getName());
}
```

### 리플렉션의 중요성

리플렉션은 애플리케이션 개발에서보다는 프레임워크, 라이브러리에서 많이 사용된다. 프레임워크, 라이브러리는 사용하는 사람이 어떤 클래스명과 멤버들을 구성할지 모르는데, 이러한 **사용자 클래스들을 기존의 기능과 동적으로 연결 시키기 위하여** 리플렉션을 사용한다

대표적인 예로는 스프링의 DI(dpendency injection), Proxy, ModelMapper 등이 있다. 이미 Spring, Hibernate, Lombok 등 많은 프레임워크에서 Reflection 기능을 사용하고 있다.

### 스프링 프레임워크에서 리플렉션

- 스프링의 DI는 스프링은 리플렉션을 통해서 클래스의 메타데이터를 분석하고, @Autowired 어노테이션이 붙은 필드를 찾아서 자동으로 의존성을 주입해 준다.

- 이 과정에서 스프링 컨테이너는 리플렉션을 사용해서 해당 필드의 타입에 맞는 빈(bean)을 찾고, 그 빈을 해당 필드에 할당한다. 이를 통해 개발자는 객체의 생성과 의존성 관리를 수동으로 하지 않아도 된다. 이런 방식으로 스프링은 개발자가 더 쉽고 유연하게 어플리케이션을 구성하고 확장할 수 있게 도와준다.

- 스프링 프레임워크에서는 내부적으로 리플렉션을 사용하여 많은 작업을 처리한다. 예를 들어, 의존성 주입, AOP(Aspect-Oriented Programming), 이벤트 처리 등 많은 기능들이 리플렉션을 기반으로 동작한다. 이 과정에서 invoke() 메서드가 자주 사용된다.

### 리플렉션의 흔적: invoke

`invoke()` 메서드는 리플렉션을 사용하여 메서드를 동적으로 호출하는 데 사용된다. 즉, 클래스의 이름이나 메서드의 이름이 컴파일 시점이 아닌, 프로그램 실행 중에 결정될 때 유용하다. 

주로 프레임워크, 라이브러리, 테스트 도구 등에서 유연하고 확장 가능한 코드를 작성할 때 사용되며, 스프링 프레임워크의 경우 **컨트롤러의 메서드를 동적으로 호출**하는 데 리플렉션을 활용합니다.

#### invoke()의 역할

`invoke()` 메서드는 `java.lang.reflect.Method` 클래스에 속해 있으며, 다음과 같은 기능을 수행합니다.

- **동적 메서드 호출**: 컴파일 시점에는 알 수 없는 클래스와 메서드를 런타임에 찾아 호출
- **인스턴스 전달**: 호출하려는 메서드가 속한 객체 인스턴스를 첫 번째 인자로 전달
- **매개변수 전달**: 호출하려는 메서드의 매개변수들을 두 번째 인자(가변 인자 또는 배열)로 전달


#### invoke()의 특징과 주의사항

- **성능 저하**
    - 정적으로 호출하는 방식(`instance.printMessage("...")`)에 비해 성능이 느림
    - JVM이 메서드를 최적화하고 직접 호출하는 대신, 런타임에 메서드를 찾고 호출하는 추가적인 작업이 필요하기 때문

- **예외 처리**
    - 메서드 호출 실패, 접근 권한 문제 등 여러 예외(`InvocationTargetException`, `IllegalAccessException`)가 발생할 수 있음


<br>

## 출처

- [[Java] 자바 리플렉션(reflection)이란?](https://curiousjinan.tistory.com/entry/java-reflection-explain)
- [☕ 누구나 쉽게 배우는 Reflection API 사용법](https://inpa.tistory.com/entry/JAVA-☕-누구나-쉽게-배우는-Reflection-API-사용법)
