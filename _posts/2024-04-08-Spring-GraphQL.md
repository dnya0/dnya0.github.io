---
title: Spring에서 GraphQL 사용해보기
author: dnya0
date:   2024-04-08 18:18:00 +0900
categories: [Backend, Spring]
tag: [Spring, GraphQL]
---

## 🤔 GraphQL이란?

GraphQL은 페이스북에서 만든 API를 위한 쿼리 언어이다.

예를 들어 스포티파이에서 아이들을 검색하면 아티스트, 앨범, 곡 등 다양한 데이터에 대한 결과를 노출한다.

![아이들을 검색한 화면](https://github.com/dnya0/dnya0.github.io/assets/84761609/38677289-4cf1-48df-89d2-affaeb267f32)

REST API를 이용한다면 해당 화면을 구성하기 위해 여러 개의 엔드포인트에 검색 요청을 해야할 것이다.

```
GET https://www.spotify.com/artist?query=아이들
GET https://www.spotify.com/music?query=아이들
GET https://www.spotify.com/album?query=아이들
```

그러나 GraphQL을 사용하면 한 번의 요청과 쿼리로 화면 구성에 필요한 모든 데이터를 가져올 수 있다.

#### 요청 

```
POST https://www.spotify.com/graphql
```

#### 쿼리

```
{
    artist(query: "아이들") { 
    	name
        ...
    }
    music(query: "아이들") {
        name
        ...
    }
    album(query: "아이들") {
        name
        ...
    }
}
```

<br>

### 📌 SQL과의 비교

sql은 데이터베이스 시스템에 저장된 데이터를 효율적으로 가져오는 것이 목적이고, gql은 웹 클라이언트가 데이터를 서버로 부터 효율적으로 가져오는 것이 목적이다. sql의 문장(statement)은 주로 백앤드 시스템에서 작성하고 호출 하는 반면, gql의 문장은 주로 클라이언트 시스템에서 작성하고 호출한다.

```sql
SELECT plot_id, species_id, sex, weight, ROUND(weight / 1000.0, 2) FROM surveys;
```

```
//GraphQL
{
  hero {
    name
    friends {
      name
    }
  }
}
```
gql은 HTTP API처럼 특정 데이터베이스나 플렛폼에 종속적이지 않으며, 네트워크 방식에도 종속적이지 않다. 일반적으로 gql의 인터페이스간 송수신은 네트워크 레이어 L7의 HTTP POST 메서드와 웹소켓 프로토콜을 활용하는데, 필요에 따라서는 얼마든지 L4의 TCP/UDP를 활용하거나 심지어 L2 형식의 이더넷 프레임을 활용 할 수도 있다.

![image](https://github.com/dnya0/dnya0.github.io/assets/84761609/bca2d3b8-33e1-4473-9f3b-40afe0d7dcda)

<br>

### 📌 REST API와 비교

![image](https://github.com/dnya0/dnya0.github.io/assets/84761609/76e0d011-ffb2-4956-99a1-0fca15b8ecb8)

![image](https://github.com/dnya0/dnya0.github.io/assets/84761609/12efc3af-2372-4daf-bc05-8931046f5176)

<br>

## 🥝 Spring에서 GraphQL 사용하기

기존에 Java 진영에서는 아래의 3가지 대표적인 라이브러리/프레임워크 중 하나를 선택하여 Spring과 GraphQL을 함께 사용했다고 한다.

- [graphql-java/graphql-java-spring](https://github.com/graphql-java/graphql-java-spring)
- [graphql-java-kickstart/graphql-spring-boot](https://github.com/graphql-java-kickstart/graphql-spring-boot)
- [Netflix/dgs-framework](https://github.com/Netflix/dgs-framework)

위 3개는 [graphql-java/graphql-java](https://github.com/graphql-java/graphql-java)를 기반으로 개발되었다. 또한 Spring for GraphQL은 공식적으로 graphql-java 의 후속 프로젝트라 소개하고 있다.

위의 라이브러리/프레임워크를 사용하면 Resolver를 개발하고 별도의 설정 등 필요했지만
Spring for GraphQL은 graphql-java의 단순 후속 프로젝트뿐 아니라 graphql-java 개발팀이 개발을 하여서 Spring이 추구하는 방향답게 추가적인 코드 없이 기존 MVC 개발하듯 개발하면 됩니다.

Spring for GraphQL은 SpringBoot 2.7.0 버전 이상부터 지원한다. Spring for GraphQL은 JPA, WebFlux 환경에서 쉽게 사용 가능하게 지원하고 있다.

<br>

### 📌 GraphQL 종속성 추가

```
implementation 'org.springframework.boot:spring-boot-starter-graphql'
```

<br>

### 📌 yml에 GraphQL 설정 추가

```yml
spring:

  ...

  graphql:
    graphiql:
      enabled: true     # '/graphiql' 경로로 접근하면 웹 브라우저에서 GraphQL API를 테스트 할 수 있다.
    schema:
      printer:
        enabled: true   # 수행한 쿼리를 콘솔에 출력한다.
```

<br>

작성중..

