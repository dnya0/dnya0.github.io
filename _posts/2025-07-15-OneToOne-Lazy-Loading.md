---
title: OneToOne에서 Fetch 전략을 Lazy로 설정했을 때
author: dnya0
date: 2025-07-15 17:49:00 +0900
categories: [Backend, Spring]
tag: [Study, Spring, JPA, Kotlin, OneToOne]
---

JPA에서 `@OneToOne` 매핑시 Fetch 전략을 Lazy로 설정해도 EAGER로 동작하는 경우가 있다. 어떤 경우에 이러한 문제점이 발생하는지, 그리고 나는 어떻게 해결했는지 글을 써보도록 하겠다.

## 기존 문제점

나는 Users 엔티티와 Marketing 엔티티가 1:1 양방향 매핑이 되어있었다. 그런데 user를 조회하면 뒤늦게 marketing이 조회되는 문제가 있었다.

### `@OneToOne` 매핑

```kotlin

@Entity
class Users(
    @OneToOne(mappedBy = "user", cascade = [CascadeType.ALL], fetch = FetchType.LAZY)
    @JsonIgnore
    var marketing: Marketing? = null
)

@Entity
class Marketing(
    @OneToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id")
    val user: Users
)
```

### 쿼리문

```
Hibernate:
    select
        a1_0.id,
        a1_0.auth_id,
        a1_0.auth_provider,
        a1_0.created_at,
        a1_0.email,
        a1_0.last_password_modified_at,
        a1_0.password,
        u1_0.id,
        u1_0.created_at,
        u1_0.deleted_at,
        u1_0.is_approved,
        u1_0.is_deleted,
        u1_0.nickname,
        u1_0.profile,
        u1_0.role,
        u1_0.updated_at
    from
        user_auth a1_0
    left join
        users u1_0
            on u1_0.id=a1_0.user_id
    where
        a1_0.email=?
Hibernate:
    select
        m1_0.id,
        m1_0.app_push_consent_date,
        m1_0.app_push_consent_yn,
        m1_0.email_consent_date,
        m1_0.email_consent_yn,
        m1_0.marketing_consent_date,
        m1_0.marketing_consent_yn,
        m1_0.user_id
    from
        marketing m1_0
    where
        m1_0.user_id=?

```

그래서 처음엔 그저 marketing도 같이 fetch join해서 가져오도록 하였으나 쓸모없는 join을 하는 것이 불편하였다. 그래서 배포 전 이유가 무엇인지 찾아보다가 한 [블로그 글](https://1-7171771.tistory.com/143)을 찾게 되었다.

<br>

## @OneToOne에서 fetch 전략을 lazy로 설정해도 eager로 조회한다?

연관관계의 주인이 호출할 때는 지연 로딩이 정상적으로 동작하지만, 연관관계의 주인이 아닌 곳에서 호출한다면 지연 로딩이 아닌 즉시 로딩으로 동작한다는 내용이었다. 내가 작성한 엔티티를 보면 marketing이 주인이기 때문에 계속 쿼리문이 나가고 있었던 것이다.

왜 이런 문제가 발생하는지 알아보기 전에, 지연 로딩이 동작하는 매커니즘을 이해해야 한다.

- 지연 로딩은 로딩되는 시점에 Fetch 전략이 Lazy로 설정되어있는 엔티티를 프록시 객체로 가져온다. 해당 예제에서는 User를 조회할때 cart를 프록시 객체로 가져오게 된다.
- 이후 실제로 Cart 객체를 사용하는 시점에 초기화 되면서 쿼리가 실행된다.
- 예를들어, getCart() 처럼 cart 객체가 사용되었을때 쿼리가 실행되는 것이다.

이렇게 지연 로딩으로 설정이 되어있는 엔티티를 조회할 때는 프록시로 감싸서 동작하게 되는데, 프록시는 null을 감쌀 수 없기 때문에 이와 같은 문제점이 발생하게 된다. 즉, 프록시의 한계로 인해 발생하는 문제이다.

그렇다면 해결방법은 무엇일까?

<br>

## 해결방법

### 1. 연관관계의 주인을 users로 설정한다.

```kotlin
@Entity
class Users(
    @OneToOne(fetch = FetchType.LAZY, cascade = [CascadeType.ALL])
    @JoinColumn(name = "marketing_id")
    val marketing: Marketing? = null
)

@Entity
class Marketing(
    @OneToOne(mappedBy = "marketing")
    val user: Users
)
```

### 2. @OneToOne으로 매핑하나 단방향 매핑으로 변경

```kotlin
@Entity
class Marketing(
    @OneToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", unique = true, nullable = false)
    val user: Users,
)
```


### 3. @OneToOne 대신 @ManyToOne + unique 제약조건 사용

```kotlin
@Entity
class Users(
    ...
) : BaseDateEntity(Domain.USER) {
    @OneToMany(mappedBy = "user")
    var marketings: MutableSet<Marketing> = mutableSetOf()
}

@Entity
class Marketing(
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", unique = true)
    val user: Users
)
```

- 의미상으로는 1:1 관계와 같지만, JPA에서는 지연로딩이 정확히 동작한다.
- Users에는 mappedBy 필요 없이 marketing 컬럼을 제거해준다.
- 데이터베이스에서 user_id에 UNIQUE 제약조건을 걸어주면 동일한 효과.

### 4. @EntityGraph 등으로 명시적 로딩 제어 (기존에 사용)

```kotlin
@EntityGraph(attributePaths = ["marketing"])
fun findById(id: String): Users
```

EntityGraph로 가져오거나 fetch join을 하는 등의 방법으로 컨트롤할 수 있다.


<br>

나는 이 중에서 4번을 사용하기로 하였다. user에서 marketing을 조회할 일이 없고 기존 db도 marketing이 user의 id를 불러왔기 때문에 db를 수정할 일이 없기 때문이다.

### 수정 이후 쿼리문

```
Hibernate:
    select
        a1_0.id,
        a1_0.auth_id,
        a1_0.auth_provider,
        a1_0.created_at,
        a1_0.email,
        a1_0.last_password_modified_at,
        a1_0.password,
        u1_0.id,
        u1_0.created_at,
        u1_0.deleted_at,
        u1_0.is_approved,
        u1_0.is_deleted,
        u1_0.nickname,
        u1_0.profile,
        u1_0.role,
        u1_0.updated_at
    from
        user_auth a1_0
    left join
        users u1_0
            on u1_0.id=a1_0.user_id
    where
        a1_0.email=?
```
