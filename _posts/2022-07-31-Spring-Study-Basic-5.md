---
title: Spring 입문 5 - 회원 관리 예제(웹 MVC 개발)
author: dnya0
date:   2022-07-31 14:13:00 +0900
categories: [Lecture, Spring]
tag: [Spring, SpringBoot, Inflearn]
---

> 인프런 김영한 님 스프링 입문 강의를 들은 후 공부한 내용입니다.
{: .prompt-info }

# 홈 화면 추가

### 홈 컨트롤러 추가

```java
package hello.hellospring.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class HomeController {

    @GetMapping("/")
    public String home() {
        return "home";
    }
}
```

### 회원 관리용 홈

`resources/templates/home.html`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <div class="container">
        <h1>Hello Spring</h1>
        <p>회원 기능</p>
        <p>
            <a href="/members/new">회원가입</a>
            <a href="/members">회원목록</a>
        </p>
    </div>
</body>
</html>
```

> 참고: 컨트롤러가 정적 파일보다 우선순위가 높다.

![image](https://user-images.githubusercontent.com/84761609/182006044-f56f6187-37ed-4c32-90af-8040527affaa.png)

<br>

# 회원 웹 기능 - 등록

## 회원 등록 폼 개발

### 회원 등록 폼 컨트롤러

`hellospring/controller/MemberForm.java`

```java
package hello.hellospring.controller;

public class MemberForm {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

`hellospring/controller/MemberController.java`

```java
package hello.hellospring.controller;

import hello.hellospring.domain.Member;
import hello.hellospring.service.MemberService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;

import java.util.List;

@Controller
public class MemberController {

    private final MemberService memberService;

    @Autowired
    public MemberController(MemberService memberService) {
        this.memberService = memberService;
    }

    @GetMapping("/members/new")
    public String createForm() {
        return "members/createMemberForm";
    }

    @PostMapping("/members/new")
    public String cteate(MemberForm form) {
        Member member = new Member();
        member.setName(form.getName());

        memberService.join(member);

        return "redirect:/";
    }
}
```

### 회원 등록 폼 HTML

`resources/templates/members/createMemberForm.html`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <div class="container">
        <form action="/members/new" method="post">
            <div class="form-group">
                <label for="name">이름</label>
                <input type="text" id="name" name="name" placeholder="이름을 입력하세요">
            </div>
            <button type="submit">등록</button>
        </form>
    </div>
</body>
</html>
```

![image](https://user-images.githubusercontent.com/84761609/182006214-0d5ba51f-fb68-47b9-8325-24bd3321fb86.png)

<br>

# 회원 웹 기능 - 조회

### 회원 컨트롤러에서 조회 기능

`hellospring/controller/MemberController.java`

```java
@GetMapping("/members")
public String list(Model model) {
    List<Member> members = memberService.findMembers();
    model.addAttribute("members", members);
    return "members/memberList";
}
```

### 회원 리스트 HTML

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <div class="container">
        <div>
            <table>
                <thead>
                <tr>
                    <th>#</th>
                    <th>이름</th>
                </tr>
                </thead>
                <tbody>
                <tr th:each="member : ${members}">
                    <td th:text="${member.id}"></td>
                    <td th:text="${member.name}"></td>
                </tr>
                </tbody>
            </table>
        </div>
    </div>
</body>
</html>
```

![image](https://user-images.githubusercontent.com/84761609/182011025-2d647ec6-d20a-49c3-9954-57d791a460b1.png)
