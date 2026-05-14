---
title: Spring 입문 2 - 정적 컨텐츠, MVC, API
author: dnya0
date:   2022-07-24 21:53:00 +0900
categories: [Lecture, Spring]
tag: [Spring, SpringBoot, Inflearn]
---

> 인프런 김영한 님 스프링 입문 강의를 들은 후 공부한 내용입니다.
{: .prompt-info }

- 정적 컨텐츠
    - 서버에서 뭐 하는거 없이 파일을 웹 브라우저에 넘겨줌
- MVC와 템플릿 엔진
    - 가장 많이 하는 방식
    - jsp, php가 템플릿 엔진
    - html을 그냥 넘겨주는 것이 아닌 서버에서 프로그래밍하여 html을 동적으로 바꾸어 보냄. 그것을 하기 위해 컨트롤러, 모델, 화면 이 세 가지를 모델 뷰 컨트롤러라 해서 MVC라 함.

    - 둘의 차이
        - 정적 컨텐츠는 파일을 그대로 전달.
        - mvc와 템플릿 엔진은 서버에서 변형하여 html을 좀 바꿔서 내려주는 방식
- API
    - 네이티브 앱으로 개발을 해야할 때 json이란 데이터 구조 포맷을 사용하여 전달해줌. 서버끼리 통신할 때도 사용.

# 정적 컨텐츠

- 스프링 부트 정적 컨텐츠 기능

`resource/static`에 `hello-static.html` 파일 생성

```html
<!DOCTYPE HTML>
<html>
<head>
    <title>static content</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
</head>
<body>
정적 컨텐츠입니다.
</body>
</html>
```

### 실행
- http://localhost:8080/hello-static.html

<br>

### 정적 컨텐츠 이미지

![image](https://user-images.githubusercontent.com/84761609/180646778-52df6bfb-f704-4646-9a65-fe634b1f7b8a.png)

- 컨트롤러가 우선순위가 더 높음.

<br>

# MVC와 템플릿 엔진

- MVC: Model, View, Controller

### Controller

```java
    // localhost:8080/hello-mvc/                -> 오류 name 파라미터에 값이 없음
    // localhost:8080/hello-mvc?name=spring!    -> name 파리미터에 spring! 값 넣기
    @GetMapping("hello-mvc")
    public String helloMVC(@RequestParam(name = "name") String name, Model model) {
        model.addAttribute("name", name);
        return "hello-temlplate";
    }
```

### View

`resource/template`에 `hello-template.html` 생성

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<body>
<p th:text="'hello' + ${name} ">hello! empty</p>
</body>
</html>
```

### 실행
- localhost:8080/hello-mvc?name=spring!

<br>

### MVC, 템플릿 엔진 이미지

![image](https://user-images.githubusercontent.com/84761609/180646974-5137ca14-75a9-4465-887e-d73ca9e592ce.png)

<br>

# API

### @ResponseBody 문자 반환

```java
@Controller
public class Hellocontroller {

    @GetMapping("hello-string")
    @ResponseBody       // http에서 body부에 이 데이터를 직접 넣어주겠다. -> html 태그 없이
    public String helloString(@RequestParam("name") String name) {
        return "hello " + name;
    }
}
```

- `@ResponseBody`를 사용하면 뷰 리졸버(`viewResolver`)를 사용하지 않음
- 대신에 HTTP에 문자 내용을 직접 반환

### @ResponseBody 객체 반환

```java
@Controller
public class Hellocontroller {

    @GetMapping("hello-api")        //json 구조로 return -> {"name":"spring"}
    @ResponseBody
    public Hello helloApi(@RequestParam("name") String name) {
        Hello hello = new Hello();
        hello.setName(name);
        return hello;
    }

    static class Hello {
        private String name;

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }
    }
}
```

### @ResponseBody 사용원리

![image](https://user-images.githubusercontent.com/84761609/180647222-07a0ff2c-057f-435e-83fc-229d62e08e80.png)

- `@ResponseBody`를 사용
    - HTTP의 BODY에 문자 내용을 직접 반환
    - `viewResolver` 대신에 `HttpMessageConverter`가 동작
    - 기본 문자처리: `StringHttpMessageConverter`
    - 기본 객체처리: `MappingJackson2HttpMessageConverter`
    - byte 처리 등등 기타 여러 HttpMessageConverter가 기본으로 등록되어 있음.

> 참고: 클라이언트의 HTTP Accept 헤더와 서버의 컨트롤러 반환 타입 정보 둘을 조합해서 `HttpMessageConverter`가 선택된다.