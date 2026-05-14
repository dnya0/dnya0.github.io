---
title: Querydsl과 JPA 같이 쓰기, 페이징과 Map
author: dnya0
date:   2024-03-20 11:21:00 +0900
categories: [Backend, Spring]
tag: [Spring, Querydsl, JPA, Paging]
math: true
mermaid: true
image:
  path: https://github.com/dnya0/dnya0.github.io/assets/84761609/9569bfb4-2c75-448c-acb1-24a1f3ca9a6d
  alt: querydsl logo
---

## Querydsl

Spring으로 개발을 하다 보면 JPA만으론 안될 때가 많다. 나 같은 경우에는 requestBody에 있는 값이 null인지 아닌지 여부에 따라 데이터를 조회하는 경우 어떻게 처리할까 고민하다가 querydsl을 사용하게 되었다. querydsl을 사용해서 동적 쿼리를 생성하면 비즈니스 로직에서 null을 검사할 필요 없이 직관적이고 간편하게 데이터를 조회할 수 있다. 아래는 동적쿼리가 필요할 때의 예시이다.

```json
{
    "stars": null,
    "countries":["MONDSTADT", "SUMERU", "LIYUE"],
    "elements": null,
    "weaponTypes": ["CATALYSTS", "POLEARMS"]
}
```

<br>

### 설정

querydsl을 적용하면서 꽤나 까다로운 구간이다. 아래의 build.gradle은 SpringBoot 3.2.1, querydsl 5.0.0 기준으로 작성된 것이다.

```
buildscript {
    ext {
        queryDslVersion = "5.0.0"
    }
}

plugins {
    id 'java'
    id 'org.springframework.boot' version '3.2.1'
    id 'io.spring.dependency-management' version '1.1.4'
    id "com.ewerk.gradle.plugins.querydsl" version "1.0.10"
}

group = 'toyproject.test'
version = '0.0.1-SNAPSHOT'

java {
    sourceCompatibility = '17'
}

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-validation'

    ...

    //querydsl
    implementation 'com.querydsl:querydsl-jpa:5.0.0:jakarta'
    annotationProcessor "com.querydsl:querydsl-apt:${dependencyManagement.importedProperties['querydsl.version']}:jakarta"
    annotationProcessor "jakarta.annotation:jakarta.annotation-api"
    annotationProcessor "jakarta.persistence:jakarta.persistence-api"

}

def querydslDir = "$buildDir/generated/querydsl"

querydsl {
    jpa = true
    querydslSourcesDir = querydslDir
}

sourceSets {
    main.java.srcDir querydslDir
}

configurations {
    querydsl.extendsFrom compileClasspath
}

compileQuerydsl {
    options.annotationProcessorPath = configurations.querydsl
}

tasks.named('test') {
    useJUnitPlatform()
}

```

<br>

기본적으로 querydsl은 프로젝트 내의 `@Entity` 어노테이션을 선언한 클래스를 탐색하고, `JPAAnnotationProcessor`를 사용해 Q 클래스를 생성한다. `querydsl-apt`가 `@Entity` 및 `@Id` 등의 어노테이션을 알 수 있도록, `javax.persistence`와 `javax.annotation`을 `annotationProcessor`에 함께 추가해야 한다.

![gradle image](https://github.com/dnya0/dnya0.github.io/assets/84761609/1dab9001-4a6a-4d57-8cfa-d71702643199)

Q 클래스를 사용하기 위해서 `compileQuerydsl`을 빌드해주면 된다.

> 나중에 서버를 실행할 때 cannot find symbol Q class 에러가 뜬다면 build clean을 한 후 서버를 실행하면 된다. 이미 Q클래스가 generated 되어있는데 서버를 실행하면서 다시 generate 하려해서 생기는 오류다.

<br>

### +) Spring Boot 3.0.0 이상 querydsl 설정

Spring Boot 3.x.x 대로 넘어오면서 querydsl 플러그인을 사용하면 여러 문제점이 생길 수 있다는 것을 알게되었다. 그래서 3버전 이상부터는 아래와 같이 설정을 해야한다. dependency는 유지하고 Querydsl 관련 플러그인 제거 및 clean 동작에 QType 제거 동작을 추가해주면 gradle task - build시 새 QType이 생성된다.

```
buildscript {
    ext {
        queryDslVersion = "5.0.0"
    }
}

plugins {
    id 'java'
    id 'org.springframework.boot' version '3.2.1'
    id 'io.spring.dependency-management' version '1.1.4'
    // querydsl plugin 제거
//    id "com.ewerk.gradle.plugins.querydsl" version "1.0.10"
}

def generatedDir = 'src/main/generated'

group = 'toyproject.genshin'
version = '0.0.1-SNAPSHOT'

java {
    sourceCompatibility = '17'
}

//configurations {
//    compileOnly {
//        extendsFrom annotationProcessor
//    }
//}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-validation'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    compileOnly 'org.projectlombok:lombok'
    runtimeOnly 'com.mysql:mysql-connector-j'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'

    implementation 'org.springframework.boot:spring-boot-starter-security'
    testImplementation 'org.springframework.security:spring-security-test'

    implementation 'com.fasterxml.jackson.datatype:jackson-datatype-jsr310'

    implementation 'io.jsonwebtoken:jjwt-api:0.11.5'
    runtimeOnly 'io.jsonwebtoken:jjwt-impl:0.11.5'
    runtimeOnly 'io.jsonwebtoken:jjwt-jackson:0.11.5'

    //querydsl
    implementation 'com.querydsl:querydsl-jpa:5.0.0:jakarta'
    annotationProcessor "com.querydsl:querydsl-apt:${dependencyManagement.importedProperties['querydsl.version']}:jakarta"
    annotationProcessor "jakarta.annotation:jakarta.annotation-api"
    annotationProcessor "jakarta.persistence:jakarta.persistence-api"

}

//def querydslDir = "$buildDir/generated/querydsl"

//querydsl {
//    jpa = true
//    querydslSourcesDir = querydslDir
//}

//sourceSets {
//    main.java.srcDir querydslDir
//}

//configurations {
//    querydsl.extendsFrom compileClasspath
//}

clean {
    delete file (generatedDir)
}

//compileQuerydsl {
//    options.annotationProcessorPath = configurations.querydsl
//}

tasks.named('test') {
    useJUnitPlatform()
}

```

<br>

## Querydsl과 JPA 같이 쓰기

querydsl과 JPA를 같이 쓰는 방법은 간단하다. JPA 레포지토리에 querydsl 코드를 구현한 인터페이스를 상속시켜주면 된다.

기존에 JpaRepository를 상속받는TestRepository가 있다고 가정하자.

```java

@Repository
public interface TestRepository extends JpaRepository<Test, String> {

}
```


지금 나의 목적은 TestRepository 하나로 querydsl까지 함께 사용하고자 하는 것이다. 먼저 interface를 생성해준다.


```java
public interface CustomTestRepository {

    List<Test> search(TestRequest testRequest);

}
```


그리고 interface를 구현해준다.


```java
@RequiredArgsConstructor
public class CustomTestRepositoryImpl implements CustomTestRepository {

    private final JPAQueryFactory jpaQueryFactory;

    @Override
    public List<Test> search(TestRequest testRequest) {
        return jpaQueryFactory
                .selectFrom(test)
                .where(
                        betweenDate(testRequest.localDateTime),
                        eqBannerType(testRequest.bannerType)
                )
                .fetch();
    }

    private BooleanExpression betweenDate(LocalDateTime localDateTime) {
        ...
    }

    private BooleanExpression eqBannerType(BannerType bannerType) {
        ...
    }

}

```


그 후 TestRepository에 interface를 상속받도록 하면 된다.


```java

@Repository
public interface TestRepository extends JpaRepository<Test, String>, CustomTestRepository {

}

```

<br>

### Paging

querydsl에서 페이징을 하려면 어떻게 해야할까. 아래는 페이징을 구현한 코드이다.

```java
@RequiredArgsConstructor
public class CustomTestRepositoryImpl implements CustomTestRepository {

    private final JPAQueryFactory queryFactory;

    @Override
    public Page<Test> findByTestRequest(TestRequest request, Pageable pageable) {
        List<Characters> result = queryFactory
                .selectFrom(test)
                .where(
                        inName(request.name()),
                        inCountry(request.countries())
                )
                .offset(pageable.getOffset())
                .limit(pageable.getPageSize())
                .fetch();

        return PageableExecutionUtils.getPage(result, pageable, getCount(request)::fetchOne);
    }

    private JPAQuery<Long> getCount(TestRequest request) {
        return queryFactory
                .select(test.count())
                .from(test)
                .where(
                        inName(request.name()),
                        inCountry(request.countries())
                );
    }
}

```

<br>

## Map

Map으로 리턴받기 위해서는 transform을 이용하면 된다.

```java
@RequiredArgsConstructor
public class CustomTestRepositoryImpl implements CustomTestRepository {

    private final JPAQueryFactory jpaQueryFactory;

    @Override
    public Map<Country, List<test>> findByDateTimeBetweenGroupBy(LocalDateTime localDateTime) {
        return jpaQueryFactory
                .from(test)
                .where(betweenDate(localDateTime))
                .transform(GroupBy
                        .groupBy(test.country)
                        .as(list(test))
                );
    }
```