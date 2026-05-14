---
title: springboot로 Rest api 만들기 HelloWorld
author: dnya0
date:   2022-07-17 00:51:00 +0900
categories: [Lecture, Spring]
tag: [Spring, SpringBoot, Java, API]
---

## 사용환경

> 공부한 내용을 되새겨볼겸 적어보았다.

| stack | Info |
|---|---|
| tool | IntelliJ Community |
| lang | JAVA |
| JDK | 11 |

<br>

# Create a Project

내가 참고한 블로그에선 [spring initializer](https://start.spring.io/) 사이트를 이용하였는데 나는 그냥 인텔리제이를 사용하는데 익숙해져야 하기도 하고 그래서 인텔리제이 내에서 생성하였다. 

### 생성방법

![image](https://user-images.githubusercontent.com/84761609/179360001-cecd57b9-8dc1-4595-9a0e-b6f65945c62c.png)

인텔리제이 창에서 [New Project]를 선택하면 해당 창이 뜬다. 사진에서 설정한대로 따라하면 된다.

| Settings | Info |
|---|---|
| lang | Java |
| Type | Gradle |
| JDK | 11 |
| Java | 11 |
| Packaging | Jar |

프로젝트 이름은 맘에드는데로 아무거나 하면 된다.

[Next]를 누르면

![image](https://user-images.githubusercontent.com/84761609/179360263-8fac1bce-29ec-4e7c-a616-069ae16dc3de.png)


해당 창이 뜨는데 여기서

- Spring Web
- Spring Security
- Spring Boot Actuator
- Spring Data JPA
- Spring Data Redis (Access + Driver)
- Lombok
- MySQL Driver
- H2 Database

를 추가해주면 된다.

다 완료됐으면 [Create] 버튼을 클릭하여 프로젝트를 생성한다.

프로젝트 생성이 완료되면 [run]을 클릭하여 실행한다. 성공적으로 실행되면

http://localhost:8080 링크로 접속한다.

![image](https://user-images.githubusercontent.com/84761609/179360464-024f610d-6552-47d4-a116-b0de8e3267f7.png)

위 화면이 뜨면 성공한 것이다.

<br>

# Print HelloWorld

## Package

자바에서 패키지는 클래스와 인터페이스의 집합을 의미한다. 협업 시 서로 작업한 클래스 사이에서 발생할 수 있는 이름 충돌 문제도 패키지를 이용하면 피할 수 있다.

자바에서 패키지는 물리적으로 하나의 디렉터리를 의미한다. 따라서 하나의 패키지에 속한 클래스나 인터페이스 파일은 모두 해당 패키지 이름의 디렉터리에 포함되어 있다. 이러한 패키지는 다른 패키지를 포함할 수 있으며, 이때 디렉터리의 계층 구조는 점(.)으로 구분된다.

예시로

```
package com.example.pepega.helloworld.java;
package com.example2.pepega.helloworld.java;
```

​가 있다.

클래스 이름(helloworld)은 같지만 package 이름이 다르므로 다른 class로 구분된다.

<br>

## @SpringBootApplication

SpringBoot에서는 @SpringbootApplication (annotation이라고 한다.) 선언만으로 대부분 설정이 자동으로 이루어진다. 하단은 boot실행을 위한 Application 설정 파일이다.

```java
package com.example.pepega;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class PepegaApplication {

	public static void main(String[] args) {
		SpringApplication.run(PepegaApplication.class, args);
	}

}
```
​
## port 지정

![image](https://user-images.githubusercontent.com/84761609/179360912-3dc8fe2c-2943-47ed-8d5b-983d645e0e11.png)

[resources] 우클릭 -> [New] -> [File] 클릭

```
application.yml
```

이라는 이름의 파일을 생성한 후 port를 지정한다.
 
```
server:
  port: 8080
```

1024번 포트 이하는 특권(privileged) 포트로 지정되어 있어 root 권한으로만 사용할 수 있는 port이다.

<br>

## controller 패키지 생성

controller 패키지를 생성한다. 

![image](https://user-images.githubusercontent.com/84761609/179361159-d9305958-9504-4c4b-9e64-db53f7ee07b9.png)

[com.example.프로젝트_이름] 우클릭 -> [new] -> [Package]를 클릭한다. 나는 이미 생성한 뒤라서 사진에 controller가 뜨는 것을 볼 수 있다.

controller 패키지에 ApiPracticeController.java를 생성한다. 저 이름이 맘에들지 않는다면 다른 원하는 이름으로 지어도 상관없다.

혹시 자바파일을 생성하고 난 후

```
package com.example.프로젝트_이름.controller;
```

패키지 이름이 위와 같지 않다면 수정해준다. 수정은 해당 파일 우클릭 -> [Refactor] -> [Rename]이다.

![image](https://user-images.githubusercontent.com/84761609/179361304-a3c125fa-b4bc-44d7-9a1d-6d0805ca3097.png)

대충 이런 식으로 되어있어야 한다.

<br>

## controller 작성

```java
package com.example.api_practice.controller;

import lombok.Getter;
import lombok.Setter;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ResponseBody;

/*
@Controller
Spring에 해당 Class가 Controller 임을 알려주기 위해 class명 상단에 @Controller를 붙여준다.
 */

@Controller
public class ApiPracticeController {

    /*
    @GetMapping("RequestURI")
    GetMapping을 붙이면 해당 주소의 Resource를 이용하기 위해서

    Get method(get, post) 로 호출해야 한다는 뜻이다.
     */

    /*
    @ResponseBody
    결과를 응답에 그대로 출력한다는 의미다.

    @ResponseBody를 지정하지 않으면 return에 지정된 "helloworld" 이름으로 된 파일을 프로젝트 경로에서 찾아 화면에 출력한다.
     */

    //화면에 helloworld 출력
    @GetMapping(value = "/helloworld/string")
    @ResponseBody
    public String helloworldString() {
        return "helloworld";
    }

    //화면에 {message:"helloworld"} 출력
    @GetMapping(value = "/helloworld/json")
    @ResponseBody
    public Hello helloworldJson() {
        Hello hello = new Hello();
        hello.message = "helloworld";
        return hello;
    }

    //화면에 helloworld.ftl의 내용이 출력
    @GetMapping(value = "/helloworld/page")
    public String helloworld() {
        return "helloworld";
    }

    /*
    @Getter
    @Setter
    선언되어있는 변수를 알아서 Getter, Setter(lombok)가 되도록 지정해준다.
     */

    @Setter
    @Getter
    public static class Hello {
        private String message;
    }
}
```

- @Controller

    - Spring에 해당 Class가 Controller 임을 알려주기 위해 class명 상단에 @Controller를 붙여준다.

- @GetMapping("RequestURI")

    - GetMapping을 붙이면 해당 주소의 Resource를 이용하기 위해서 Get method(get, post) 로 호출해야 한다는 뜻이다.
        
            http://localhost:8080/helloworld/json

    - 웹 브라우저에 위 주소창을 실행하면 Get방식으로 호출된다.
    
    - /helloworld/ json을 Mapping하고 있는 메서드( public Hello helloworldJson() )가 실행된다.

- @ResponseBody
    - 결과를 응답에 그대로 출력한다는 의미다.

    - @ResponseBody를 지정하지 않으면 return에 지정된 "helloworld" 이름으로 된 파일을 프로젝트 경로에서 찾아 화면에 출력한다.

- @Getter <br> @Setter
    
    - 선언되어있는 변수를 알아서 Getter, Setter(lombok)가 되도록 지정해준다.

<br>

## Create ftl File

![image](https://user-images.githubusercontent.com/84761609/179361887-a860375d-edaa-49ba-b390-9d953a55d9f7.png)

resource 파일 아래 templates 우클릭 -> [New] -> [File]을 클릭하여

```
helloworld.ftl
```

을 생성한다.

ftl 파일 내에 

```
HelloWorld
```

작성

application.yml 에 하단 내용을 추가한다.

```yml
spring:
  freemarker:
    template-loader-path: classpath:/templates/
    suffix: .ftl
```

build.gradle 에 하단 내용을 추가한다.

```
implementation 'org.springframework.boot:spring-boot-starter-freemarker'
```
​
<br>

## 실행결과

![image](https://user-images.githubusercontent.com/84761609/179360464-024f610d-6552-47d4-a116-b0de8e3267f7.png)

- User name: user
- Passworld: 밑의 콘솔 확인

![image](https://user-images.githubusercontent.com/84761609/179361653-b65eeb51-4103-47ca-8586-b5f84455ad23.png)

아래는 로그인 후 뜨는 화면이다.

![image](https://user-images.githubusercontent.com/84761609/179361717-1875298c-a854-4379-8561-bf377b83ba78.png)

로그인이 됐으면 각각 주소를 실행한다.

```
http://localhost:8080/helloworld/string
http://localhost:8080/helloworld/json
http://localhost:8080/helloworld/page
```

<br>

## 실행화면

![image](https://user-images.githubusercontent.com/84761609/179361751-19dd77e5-7284-45f2-bde1-862309c3b9c1.png)

![image](https://user-images.githubusercontent.com/84761609/179361776-a45a5072-d72b-4ee8-a2b9-4bf66ef363eb.png)

![image](https://user-images.githubusercontent.com/84761609/179361792-0bc15de8-4d84-47f7-aa08-f11af8ac643f.png)
