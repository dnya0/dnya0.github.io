---
title: Spring AnnotationException - Illegal attempt to map a non collection as a @OneToMany, @ManyToMany or @CollectionOfElements
author: dnya0
date:   2022-11-09 17:49:00 +0900
categories: [TroubleShooting, Spring]
tag: [Spring, JPA, Java, Error]
---

> [에러 메시지] AnnotationException: Illegal attempt to map a non collection as a @OneToMany, @ManyToMany or @CollectionOfElements
{: .prompt-danger }

<br>

## 해결 방법

- 에러나는 코드

```java
@OneToMany(fetch = FetchType.LAZY)
@JoinColumn(name = "userId")
private UserEntity userId;
```

<br>

- `UserEntity`를 `List`로 고치기

```java
@OneToMany(mappedBy = "music_like", fetch = FetchType.LAZY)
@JoinColumn(name = "userId")
private List<UserEntity> userId;
```