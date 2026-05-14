---
title: Spring 입문 1 - 프로젝트 생성, 라이브러리, 빌드
author: dnya0
date:   2022-07-22 19:46:00 +0900
categories: [Lecture, Spring]
tag: [Spring, SpringBoot, Inflearn]
---

> 인프런 김영한 님 스프링 입문 강의를 들은 후 공부한 내용입니다.
{: .prompt-info }

전에 스프링부트 프로젝트를 생성한 경험이 있어서 편하게 할 수 있었다. 인프런 김영한 님 강의를 보고 스프링 공부를 시작하게 되었다! 감기 걸려서 하루에 1강씩 들을 예정..

<br>

# Create a Project

## Settings

| Settings | Info |
|---|---|
| Name | hello-spring|
| lang | Java |
| Type | Gradle | 
| Group | hello |
| Artifact | hello-spring |
| JDK | 11 |
| Java | 11 |
| Packaging | Jar |

![image](https://user-images.githubusercontent.com/84761609/180404027-6ba0ca42-cb41-4c63-893c-01d777f8f5b5.png)

요즘엔 maven 보단 gradle로 하는 추세라고 한다.

artifact는 maven project에서 사용되는 개념으로만 한정하고 있다. maven project의 빌드 결과물로 나오는 개체를 artifact라고 한다. 빌드로 나오는 JAR 파일을 artifact라고 보면 된다. 

Group엔 보통 기업명이나 기업 도메인명을 넣는다고 한다.

![image](https://user-images.githubusercontent.com/84761609/180405632-a859d2c9-67dd-44c6-8c4f-5c1c451b8a92.png)

스프링부트 버전은 가장 최신 버전으로 한다. 사진에선 2.7.2가 가장 최신 버전이다. 스냅샷 등 괄호가 붙지 않은 것을 선택하면 된다. 스냅샷 등이 붙은 것은 베타버전.

- dependencies
    - spring web
    - thymeleaf

## 폴더

![image](https://user-images.githubusercontent.com/84761609/180406379-d1a020ca-ca12-4c41-80ab-73957e36c2f5.png)


요즘엔 메인과 테스트 폴더가 기본적으로 따로 나눠져있으며 해당 방식으로 표준화되어있다. main의 java 밑에 실제 패키지가 있으며 테스트밑엔 테스트 코드들이 들어가있다. 테스트 코드는 매우 중요하다.

자바 제외한 html 파일 등은 다 resources 폴더에 넣으면 된다.

![image](https://user-images.githubusercontent.com/84761609/179361717-1875298c-a854-4379-8561-bf377b83ba78.png)

실행 시켰을 때 해당 화면이 뜨면 성공!

<br>

### build.gradle


```yml
repositories {
    mavenCentral()  // mavenCentral에서 디펜던시스를 다운받아라
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

<br>

## Build 설정

![image](https://user-images.githubusercontent.com/84761609/180408574-d9bc154d-3445-4be0-82fa-a1ee1d1606ca.png)

gradle을 통해서 빌드하면 느릴 때가 있다. gradle이 아닌 InteliJ IDEA로 바로 빌드할 수 있도록 설정을 바꿔주자.


# Library 살펴보기

![image](https://user-images.githubusercontent.com/84761609/180409079-64a1823b-d44d-410b-9f65-77e163369a19.png)

요즘 툴들은 의존 관계를 관리해준다.

![image](https://user-images.githubusercontent.com/84761609/180409624-938a13ea-4669-49f6-8f58-09690d238c1d.png)

현업에선 System.out.println(이하 syso)보단 log를 많이 사용

로그로 남겨야 심각한 에러만 따로 모아보거나 로그파일들이 관리되기 때문에 syso를 거의 쓰지 않는다고 한다. 취준생이나 아직 서버 등 관련 경험 못한 사람들은 의문이 있을 수 있다. 실무에선 log를 쓰지만 강의에선 syso를 많이 쓸 것같다고 하였다.

> 사람들이 저 두가지를 많이 써서 아예 포함됐다고 한다. 로그에 관해 궁금한게 있으면 slf4j랑 logback에 대해 검색해보기.

<br>

## 정리

> Gradle은 의존관계가 있는 라이브러리를 함께 다운로드 한다.

### 스프링 부트 라이브러리

- spring-boot-starter-web
    - spring-boot-starter-tomcat: 톰캣 (웹서버)
    - spring-webmvc: 스프링 웹 MVC
- spring-boot-starter-thymeleaf: 타임리프 템플릿 엔진(View)
- spring-boot-starter(공통): 스프링 부트 + 스프링 코어 + 로깅
    - spring-boot
        - spring-core
    - spring-boot-starter-logging
        - logback, slf4j

### 테스트 라이브러리

- spring-boot-starter-test
    - junit: 테스트 프레임워크
    - mockito: 목 라이브러리
    - assertj: 테스트 코드를 좀 더 편하게 작성하게 도와주는 라이브러리
    - spring-test: 스프링 통합 테스트 지원

요즘엔 junit보단 junit5를 많이 쓴다고 한다.

<br>

# View 환경설정

## Welcom Page 만들기

resources/static에 index.html을 만든다.

```html
<!DOCTYPE HTML>
<html>
<head>
    <title>Hello</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
</head>
<body>
Hello
<a href="/hello">hello</a>
</body>
</html>
```

### springboot가 지원하는 Welcome Page 기능

- `static/index.html`을 올려두면 Welcome Page 기능을 한다.
- String.io에서 설명하는 Welcome Page
    - 1.1.5. Welcome Page
        - Spring Boot supports both static and templated welcome pages. It first looks for an index.html file in the configured static content locations. If one is not found, it then looks for an index template. If either is found, it is automatically used as the welcome page of the application.

<br>

## Controller

웹 어플리케이션에서 첫번째 진입점이 컨트롤러이다.

![image](https://user-images.githubusercontent.com/84761609/180412490-81c057cd-9162-4044-84f1-4b0ccdd7eff4.png)

hello.hellospring을 마우스 우클릭하여 controller 패키지를 만든다. controller 패키지에 hellocontroller.java 파일을 만든다.

코드 입력

```java
package hello.hellospring.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class Hellocontroller {

    @GetMapping("hello")
    public String hello(Model model) {
        model.addAttribute("data", "hello!!");
        return "hello";
    }
}
```

웹 어플리케이션에서 /hello라고 들어오면 hello 함수를 호출해줄 것이다.

![image](https://user-images.githubusercontent.com/84761609/180413194-c91bc37b-37ff-428b-b4c6-c84c28d8c777.png)

`resources/tempates`에 `hello.html`을 만들어준다.

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Hello</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
</head>
<body>
<p th:text="'안녕하세요.' + ${data} ">안녕하세요. 손님</p>
<!--
    ${data}는
    model.addAttribute("data", "hello!!");의 data를 가리킴
    따라서 hello!!로 치환됨
-->
</body>
</html>
```

![image](https://user-images.githubusercontent.com/84761609/180413527-957ac13d-ac4e-46c8-aeab-ef6318b1a9c3.png)

이렇게 뜨면 성공이다.

<br>

## 동작 환경

![springweb (3)](https://user-images.githubusercontent.com/84761609/180420349-e9543324-5a69-451d-92f8-b4d6b8c567eb.jpg)


- 컨트롤러에서 리턴 값으로 문자를 반환하면 viewResolver가 화면을 찾아서 처리한다.
    - 스프링 부트 템플릿 엔진 기본 viewName 매핑
    - `resource:templetes/` + {ViewName} + `.html`

return hello를 하면 리소스에 있는 템플릿에 hello를 찾아서 렌더링을 해라라는 뜻이다.

<br>

> 참고: `spring-boot-devtools` 라이브러리를 추가하면, `html` 파일을 컴파일만 해주면 서버 재시작 없이 View 파일 변경이 가능하다. <br>InteliJ 컴파일 방법: 메뉴 build -> Recompile

<br>

# 빌드하고 실행하기

> cmd로 빌드

강의에선 따로 cmd 창을 열어서 했는데 나는 그냥 인텔리제이 터미널에서 실행했다.

![image](https://user-images.githubusercontent.com/84761609/180421176-43b34154-880a-4b3d-964f-e3a54fef2c85.png)

맥의 경우엔 `gradlew`를, 윈도우의 경우엔 `gradlew.bat`을 빌드해주면 된다. 나는 윈도우이기에 gradle.bat 빌드방법을 적기로 하였다.

![image](https://user-images.githubusercontent.com/84761609/180421511-5e5d0e1a-594d-42a7-bb25-31eec9f2e259.png)

```shell
./gradlew.bat build
cd build
cd libs
```

해당 코드 이후에 `dir`을 다시 쳐주면 

![image](https://user-images.githubusercontent.com/84761609/180421898-4fba761c-b1e7-4679-9b0d-06dbb80f8a1e.png)

를 볼 수 있다. 여기서 `hello-spring-0.0.1-SNAPSHOT.jar`를 실행해주면 된다.

```shell
java -jar hello-spring-0.0.1-SNAPSHOT.jar
```

![image](https://user-images.githubusercontent.com/84761609/180422454-279d8063-e9ef-4b68-a990-d241080b75c5.png)

`^C`로 종료.

서버 배포할 땐 저 파일만 넣어서 배포해주고 java -jar 해서 실행시키면 된다. 그러면 서버에서 스프링이 실행된다.

<br>

# Error

`./gradlew.bat build`를 실행시켜서

```shell
> Task :compileJava FAILED

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':compileJava'.
> invalid source release: 11

* Try:
> Run with --stacktrace option to get the stack trace.
> Run with --info or --debug option to get more log output.
> Run with --scan to get full insights.

* Get more help at https://help.gradle.org

BUILD FAILED in 8s
1 actionable task: 1 executed
```

해당 오류가 뜬 경우 환경변수가 jdk11로 되어있는지 확인해야한다.