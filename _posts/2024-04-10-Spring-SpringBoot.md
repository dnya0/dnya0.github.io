---
title: Spring Framework와 Spring Boot의 차이
author: dnya0
date:   2024-04-10 17:30:00 +0900
categories: [Backend, Spring]
tag: [Backend, Spring]
math: true
mermaid: true
image:
  path: https://user-images.githubusercontent.com/84761609/220085479-19ee260a-d709-4a47-b788-416275d8a2d8.png
  alt: Spring Image
---

Spring Framework와 SpringBoot의 차이점이 무엇일까? SpringBoot는 Spring Framework를 편리하게 사용하게 도와주는 도구인 것은 알았지만 구체적으로 차이점이 무엇인지는 궁금해하지 않았다.

<br>

## Spring Framework

> Whatever happened next, the framework needed a name. In the book it was referred to as the “Interface21 framework” (at that point it used com.interface21 package names), but that was not a name to inspire a community. Fortunately Yann stepped up with a suggestion: “Spring”. His reasoning was association with nature (having noticed that I'd trekked to Everest Base Camp in 2000); and the **fact that Spring represented a fresh start after the “winter” of traditional J2EE**. We recognized the simplicity and elegance of this name, and quickly agreed on it.

위는 [Spring 공식 문서](https://spring.io/blog/2006/11/09/spring-framework-the-origins-of-a-project-and-a-name)의 일부분으로 개발자에게 "겨울"은 끝나고 "봄"이 왔다는 뜻으로 Spring이라는 이름을 붙여줬다는 것을 알 수 있다.

스프링 프레임워크<sub>Spring Framework</sub>는 Java 기반 애플리케이션 개발을 지원하는 오픈소스 애플리케이션 프레임워크로 간단히 스프링<sub>Spring Framework</sub>이라고도 불린다.

<br>

### 특징

#### DI(Dependency Injection)

DI란 개발자가 Spring 프레임워크에 의존성을 주입하면서 객체 간 결합을 느슨하게 하는 것이다. 객체 간 결합이 느슨하면 코드의 재사용성이 증가하고, 단위 테스트가 용이해진다.

<br>

#### IoC(Invesion of Control)

IoC는 컨트롤의 제어권이 개발자에게 있는 것이 아닌 프레임워크가 대신해서 해주는 것을 말한다. `Servlet`이나 `Bean` 같은 코드를 개발자가 직접 작성하지 않고, 프레임워크가 대신 수행한다.

제어의 역전이라는 말이 어려울 수 있는데, 쉽게 말하자면 기존에는 자바 코드를 작성할 때 객체의 생성, 의존관계 설정 등을 개발자가 해줘야 했지만, 프레임워크가 대신해준다는 의미이다.


<br>

#### AOP(Aspect Oriented Programming)

AOP는 핵심기능을 제외한 부수적인 기능을 프레임워크가 제공하는 특징이다. 예를 들어 Spring 프로젝트에 security를 적용하거나, logging 등을 추가하고 싶을 때 기존 비즈니스 로직을 건들지 않고 AOP로 추가할 수 있다.

<br>

#### 중복 코드 제거

예를 들어 JDBC 같은 템플릿을 사용할 때 중복되는 코드도 많고 복잡하다. 이를 모두 제거할 수 있다.

<br>

#### 다른 프레임워크와의 통합
JUnit, Mockito와 같은 유닛 테스트 프레임워크와 통합이 간단하다. 이를 통해 개발하는 프로그램의 품질이 향상된다.

<br>

## Spring Boot

> Spring Boot makes it easy to create stand-alone, production-grade Spring based Applications that you can "just run".<br><br>We take an opinionated view of the Spring platform and third-party libraries so you can get started with minimum fuss. Most Spring Boot applications need minimal Spring configuration.

위는 [Spring Boot 공식 문서](https://spring.io/blog/2006/11/09/spring-framework-the-origins-of-a-project-and-a-name)의 일부분으로 스프링 부트는 단독적이고, 상용화 수준의, 스프링 기반 애플리케이션을 단지 실행할 수 있을 정도로 쉽게 만들 수 있다는 내용이다.

Spring Boot는 Spring Framework를 기반으로 좀더 확장된 모듈로써, Spring Framework의 기능을 포함하고 있고, Spring Framework의 단점이라고도 할수 있는 복잡한 환경설정을 자동으로 해주기때문에 개발자가 좀더 개발에만 집중할수 있다는 장점이 있다.

<br>

### 차이점

기본적으로 Spring은 경량 애플리케이션 프레임워크이며 Struts, JSP, Hibernate 등과 같은 다양한 프레임워크에 대한 지원을 제공하고 애플리케이션을 빌드하는 데 사용되는 반면 **Spring Boot는 스프링 - REST API 개발에 주로 사용**되는 기반 프레임워크이다.

Spring의 주요 차이점 또는 주요 기능은 종속성 주입이며 Spring Boot의 경우 자동 구성이며 Spring Boot Framework 개발자의 도움으로 개발 시간, 개발자 노력을 줄이고 생산성을 높일 수 있다.

| | spring | spring boot | 
| --- | --- | --- |
| 사용 | 자바 기반 어플리케이션을 만드는 데 사용됨 | 주로 REST API 개발을 위해 사용됨 |
| 주요기능 | 의존성 주입 | AutoConfiguration |
| 개발 유형 | 느슨하게 결합된 어플리케이션을 만드는데 도움이 된다. | 독립 실행형 어플리케이션을 만드는데 도움이 된다. |
| 서버 종속성 | 서버를 명시적으로 설정해야 함 | Tomcat이나 Jetty와 같은 임베디드 서버를 제공 |
| 구성 | 수동으로 구성을 빌드해야 함 | 부트스트랩 가능한 기본 구성이 있음 |

