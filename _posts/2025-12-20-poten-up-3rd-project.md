---
title: 세 번째 프로젝트 회고 - LXP 프로그래밍
author: dnya0
date: 2025-12-20 16:08:00 +0900
categories: [Post, PotenUp]
tag: [Post, PotenUp, Java, Spring, QueryDsl]
math: true
mermaid: true
image:
  path: ../assets/img/posts-image/potenupLogo.png
  alt: Poten Up Image
---

## 🥑 들어가며

드디어 세 번째 LXP 프로젝트에 들어가게 되었다. 이번 프로젝트부터는 팀 전체가 바뀌는 대신 기존 팀원 중 1~3명만 교체되는 방식으로 운영되는데, 이는 실무에서 문서화와 온보딩 프로세스가 얼마나 중요한지 체감시키려는 의도로 보인다. 이번 프로젝트는 헥사고날로 구현하게 되었다. 또한 이번 프로젝트부터 프론트엔드와 협업을 하게 되었다.

<br>

## 🔖 프로젝트 정보

|   Category   | Architecture                                         |
| :----------: | :--------------------------------------------------- |
|   Language   | Java 17                                              |
|    Build     | Gradle                                               |
| Architecture | Hexagonal, Multi-Module, Monolithic, Spring Modulith |
|  Framework   | SpringBoot 4.0.0                                     |
|     Auth     | Spring Security, JJWT                                |
|   Database   | Mysql, Spring Data JPA, Redis                        |
|     API      | Swagger(OAS)                                         |

이번 프로젝트에서 유저와 인증(Auth) 도메인을 맡게 되며, Spring Security와 JWT를 활용한 보안 체계 구축을 담당하게 되었다. 우리 팀은 설계 단계에서 **헥사고날 아키텍처(Hexagonal Architecture)**와 멀티모듈 기반의 모놀리식 구조를 채택했다.

팀원 모두 개발 경험이 있었기에 새로운 아키텍처 패턴에 도전해보고자 하는 의지가 컸고, 무엇보다 향후 서비스 규모 확장에 따른 MSA(Microservices Architecture) 전환 과정을 직접 경험해보고 싶었기 때문이다. 멀티모듈을 통해 도메인 간 결합도를 물리적으로 격리하는 것이 그 시작이 될 것이라 믿었다. 그러나 이 결정은 미래에 뼈저리게 후회하게 된다..

컨벤션의 경우 따로 헤더를 나누지 않았다. 저번 프로젝트와 비슷하게 브랜치 룰을 정하고 이슈, pr 템플릿을 만들고,.. 한 가지 다른 점이 있다면, Lint에 대한 것이다. 저번 프로젝트에서 `Spotless`를 사용한 결과, 포멧팅은 잘 이루어졌지만, 오히려 코드가 더 읽기 불편해지는 결과를 초래했다. Spotless를 제거하자고 말을 꺼낼지 꽤나 고민했던 기억이 있다. 이번 프로젝트는 Spotless를 적용하지 않기로 하였다. 또한

멀티 모듈 프로젝트의 핵심인 의존성 단방향 원칙을 준수하기 위해 계층별 모듈화를 진행했다. 모든 모듈이 공통적으로 사용하는 common 모듈과 진입점이 되는 api 모듈을 정의하고, 각 도메인 모듈들이 정해진 의존 관계에 따라 이를 참조하도록 설계하여 모듈 간의 결합도를 최소화했다.

### 폴더 구조

```bash
├── application
│   └── local
│       ├── port
│       │   ├── provided
│       │   │   ├── command
│       │   │   ├── policy
│       │   │   └── usecase
│       │   └── required
│       │       └── dto
│       └── service
├── domain
│   ├── common
│   │   ├── exception
│   │   ├── model
│   │   │   └── vo
│   │   └── policy
│   ├── local
│   │   ├── event
│   │   ├── model
│   │   │   ├── entity
│   │   │   └── vo
│   │   ├── policy
│   │   └── repository
│   └── service
├── infrastructure
│   ├── external
│   ├── mapper
│   ├── messaging
│   ├── persistence
│   │   └── local
│   │       ├── adapter
│   │       ├── entity
│   │       └── repository
│   └── security
│       ├── adapter
│       ├── config
│       ├── filter
│       ├── handler
│       ├── jwt
│       │   └── config
│       ├── model
│       └── resolver
└── presentation
    └── rest
        ├── common
        └── local
            └── dto
                └── reqeust
```

Auth 도메인은 확장성이 중요하므로, 향후 소셜 로그인 도입을 고려하여 local 패키지를 별도로 분리해 구조화했다. 또한 헥사고날 아키텍처의 포트(Port) 개념을 명확히 하기 위해 in/out이라는 방향 중심의 명칭 대신, 인터페이스의 역할 관계를 나타내는 provided와 required라는 명칭을 채택했다.

### ERD

![](/assets/img/posts-image/2025-12-20-01.png)

멀티 모듈 구조에서 각 Bounded Context(BC)를 독립적인 모듈로 분리하고, 모듈 간의 직접적인 의존성을 제거하여 결합도를 낮췄다.

## 📌 구현 과정

### JWT 기반 인증 체계와 쿠키를 이용한 토큰 관리 전략

이번 프로젝트에서는 효율적인 보안 관리를 위해 JWT 기반의 인증 체계를 구축하였다.

먼저, 초기 단계의 복잡도를 낮추고 빠른 MVP 구현을 위해 Refresh Token은 별도로 도입하지 않기로 결정하였다. 대신 보안성 강화를 위해 Access Token을 HttpOnly 옵션이 적용된 쿠키에 저장하여 전달하는 방식을 채택했다. 이를 통해 자바스크립트를 이용한 XSS(Cross-Site Scripting) 공격으로부터 토큰 유출을 방지하고자 하였다.

단일 토큰(Access Token) 체계의 특성상 토큰 유출 시 보안에 취명적일 수 있다는 한계가 있다. 이를 보완하기 위해 Redis를 활용한 블랙리스트(Blacklist) 관리 메커니즘을 도입했다. 로그아웃 요청이나 이상 징후가 발견된 토큰을 Redis에 저장하고 요청마다 검증함으로써, 유효 기간이 남은 토큰이라도 즉시 무효화할 수 있도록 설계했다.

### 어노테이션으로 userId 가져오기

`common` 모듈에 커스텀 어노테이션 `@CurrentUserID`를 정의했고, 그 구현을 Auth 모듈에서 해놓는 식으로 하였다.

`common`에 아래와 같이 어노테이션을 구현하였다.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.PARAMETER)
public @interface CurrentUserId {
}
```

그리고 `auth` 모듈에 아래와 같이 `resolver`를 구현해준다.

```java
@Slf4j
@Component
public class CurrentUserArgumentResolver implements HandlerMethodArgumentResolver {

    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        return parameter.hasParameterAnnotation(CurrentUserId.class) &&
            parameter.getParameterType().equals(String.class);
    }

    @Override
    public @Nullable String resolveArgument(MethodParameter parameter,
                                            @Nullable ModelAndViewContainer mavContainer,
                                            NativeWebRequest webRequest,
                                            @Nullable WebDataBinderFactory binderFactory) throws Exception {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();

        if (Objects.isNull(authentication) || !authentication.isAuthenticated()) {
            log.debug("Argument resolution skipped: Authentication object is null or user is not authenticated.");
            return null;
        }

        Object principal = authentication.getPrincipal();

        if (principal instanceof String longPrincipal) {
            return longPrincipal;
        } else if (principal instanceof CustomUserDetails c) {
            return c.getUserId();
        }

        log.info("Argument resolution failed: Principal type is not supported. Type: {}", principal.getClass().getName());
        return null;
    }
}
```

그리고 이 resolver를 `Configuration`으로 등록해주면 끝이 난다.

```java
@Configuration
@RequiredArgsConstructor
public class AuthWebConfig implements WebMvcConfigurer {

    private final CurrentUserArgumentResolver currentUserArgumentResolver;

    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
        resolvers.add(currentUserArgumentResolver);
    }

    ...
}

```

### CORS 설정

```java
@Configuration
@RequiredArgsConstructor
public class AuthWebConfig implements WebMvcConfigurer {

    ...

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
            .allowedOriginPatterns("http://localhost:*")
            .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
            .allowCredentials(true);
    }
}

```

현재 `addCorsMappings` 설정은 개발 환경에서 CORS 에러를 방지하기 위해 임시로 구성한 것이다. 실제 배포까지 할 경우 prod 프로필을 분리하여 운영할 계획이다. 예를 들어, 현재 운영 중인 다른 프로젝트처럼 별도의 `CorsProperties` 클래스를 만들어 yml 설정값에 따라 프로필별(dev/prod)로 허용 도메인을 관리하거나, 환경 변수에 따라 동적으로 설정을 불러오는 방식을 적용할 예정이다. 아래는 다른 프로젝트의 예시이다.

```Kotlin
@ConfigurationProperties(prefix = "cors")
data class CorsProperties(
    val allowedOrigins: List<String>
) {
    val originsArray: Array<String>
        get() = allowedOrigins.toTypedArray()
}

@Configuration
class WebConfig(
    private val corsProperties: CorsProperties
) : WebMvcConfigurer {

    ...

    override fun addCorsMappings(registry: CorsRegistry) {
        registry.addMapping("/api/**")
            .allowedOrigins(*corsProperties.originsArray)
            .allowedMethods("GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS")
            .allowCredentials(true)
            .maxAge(3600)
    }
}
```

```properties
# application-dev.properties
cors.allowed-origins=http://localhost:3000

# application-prod.properties
cors.allowed-origins=https://test-domain.com
```

## 🚨 Issues

### OAS

이번에 처음으로 OAS(OpenAPI Specification)를 도입해 보았다. 하지만 처음이라 익숙하지 않았던 탓인지, 오히려 문서 업데이트가 늦어지는 원인이 되기도 했다. '살아있는 문서'를 지향했지만, 첫 헥사고날 아키텍처 적용과 구현에 급급해 yml 명세까지 세심히 챙기지 못한 점이 아쉽다. 설계 공부에 집중하느라 프론트엔드 작업은 다소 뒷전이 되기도 했고, 지금까지 진행한 프로젝트 중 가장 고비가 많고 힘들었다. 하지만 결국 API 구현을 모두 마쳤고, 그 과정에서 깊이 있는 고민을 할 수 있었던 시간이었다.

### Spring Modulith 환경에서의 단방향 의존성 관리

#### 단방향 의존성과 모듈 분리

멀티 모듈 아키텍처의 핵심은 의존성이 단방향으로 흐르게 하여 결합도를 낮추는 것이다. 하지만 프로젝트 진행 중 모듈 간 참조가 복잡해지면서, 기존에 없던 api 모듈을 뒤늦게 추가하게 되었다. 모듈간의 통신을 구현하는 과정에서 **순환 의존성(Circular Dependency)**과 `Spring Modulith`의 의존성 규칙 위반(Violations) 오류를 마주하게 되었다.

#### Spring Modulith 의존성 위반 분석

```
org.springframework.modulith.core.Violations: - Module 'auth' depends on non-exposed type com.lxp.user.application.port.provided.external.ExternalUserAuthPort within module 'user'!
UserInfoSearchPortHandler declares constructor UserInfoSearchPortHandler(ExternalUserAuthPort) in (UserInfoSearchPortHandler.java:0)
- Module 'auth' depends on non-exposed type com.lxp.user.application.port.provided.dto.UserAuthResponse within module 'user'!
Method <com.lxp.auth.application.local.handler.UserInfoSearchPortHandler.getUserInfoByEmail(java.lang.String)> calls method <com.lxp.user.application.port.provided.dto.UserAuthResponse.userId()> in (UserInfoSearchPortHandler.java:27)
- Module 'auth' depends on non-exposed type com.lxp.user.application.port.provided.dto.UserAuthResponse within module 'user'!
Method <com.lxp.auth.application.local.handler.UserInfoSearchPortHandler.getUserInfoByEmail(java.lang.String)> calls method <com.lxp.user.application.port.provided.dto.UserAuthResponse.email()> in (UserInfoSearchPortHandler.java:27)
...
- Module 'auth' depends on non-exposed type com.lxp.user.application.port.provided.external.ExternalUserSavePort within module 'user'!
Constructor <com.lxp.auth.application.local.handler.UserSavePortHandler.<init>(com.lxp.user.application.port.provided.external.ExternalUserSavePort)> has parameter of type <com.lxp.user.application.port.provided.external.ExternalUserSavePort> in (UserSavePortHandler.java:0)
	at app//org.springframework.modulith.core.Violations.and(Violations.java:141)
	at app//org.springframework.modulith.core.ApplicationModules.detectViolations(ApplicationModules.java:508)
	at app//org.springframework.modulith.core.ApplicationModules.verify(ApplicationModules.java:462)
	at app//org.springframework.modulith.core.ApplicationModules.verify(ApplicationModules.java:446)
	at app//com.lxp.ApplicationModuleTests.verifyModularStructure(ApplicationModuleTests.java:17)
```

발생한 오류는 `auth` 모듈이 `user` 모듈의 내부에 숨겨져 있어야 할 타입(`ExternalUserAuthPort`, `UserAuthResponse`)에 의존하고 있다는 경고였다. `Spring Modulith`는 기본적으로 모듈의 최상위 패키지 외의 하위 패키지들을 '내부 구현'으로 간주하여 외부 노출을 차단한다. 즉, `user` 모듈이 명시적으로 허용한 API 외에는 다른 모듈(`auth` 등)에서 참조할 수 없도록 엄격히 제한하는 것이다.

아래는 당시 `user` 모듈의 `application`의 패키지 구조이다.

```
├── command
│   └── UserSaveCommand.java
├── event
├── port
│   ├── provided
│   │   ├── dto
│   │   │   ├── UserAuthResponse.java
│   │   │   └── UserInfoResponse.java
│   │   └── external
│   │       ├── ExternalUserAuthPort.java
│   │       ├── ExternalUserInfoPort.java
│   │       ├── ExternalUserSavePort.java
│   │       └── ExternalUserStatusPort.java
│   └── required
│       ├── command
│       │   ├── ExecuteSearchUserCommand.java
│       │   ├── ExecuteUpdateUserCommand.java
│       │   └── ExecuteWithdrawUserCommand.java
│       ├── dto
│       │   └── UserInfoDto.java
│       └── usecase
│           ├── SearchUserProfileUseCase.java
│           ├── UpdateUserProfileUseCase.java
│           └── WithdrawUserUseCase.java
└── service
    ├── ExternalUserAuthService.java
    ├── ExternalUserSaveService.java
    ├── SearchUserProfileService.java
    ├── UpdateUserService.java
    └── WithdrawUserService.java
```

당시 `user` 모듈의 구조를 보면, `provided.external` 패키지에 위치한 포트들이 외부로 노출되지 않은 상태에서 `auth` 모듈에 의해 참조되고 있었다. 이는 `Modulith`가 지향하는 **"캡슐화된 모듈 구조"**를 위반한 것이다.

#### 해결을 위한 두 가지 선택지

1. API 패키지 규칙 준수 (권장)

`Spring Modulith`의 기본 컨벤션에 따라, 외부 모듈에 노출할 클래스들을 `api`라는 이름이 포함된 패키지로 이동시키는 방법이다.

예를 들어 `ExternalUserAuthPort`의 경우 기존의 `application.port.provided.external`가 아닌 `application.api.port.provided.external`와 같이 패키지 경로를 설정하는 것이다.

`Modulith`에서 애플리케이션이 여러 모듈(예: user, auth)로 나뉘어져 있을 때, 모듈 간의 통신은 **공식적인 계약(Contract)** 을 통해서만 이루어져야 한다. `Modulith`는 모듈의 `api` 패키지에 있는 타입만 다른 모듈이 안전하게 의존할 수 있는 공식적인 계약으로 간주한다.

- `user` 모듈 (제공자): 자신의 내부 구현(비즈니스 로직, 데이터베이스 접근 등)을 숨기고, 외부 모듈이 사용할 수 있는 기능 목록만 정의해야 한다.
- `api` 패키지의 역할: 이 기능 목록, 즉 계약을 담는 곳이 바로 `api` 패키지다.

따라서 다른 모듈로 노출시키고 싶을 경우 `api`와 같이 미리 정해진 노출 패키지에 두지 않으면 "이것은 외부에 노출되어서는 안 되는 내부 타입인데, `auth` 모듈이 의존하고 있다!"라고 판단하여 위반 오류를 발생시키는 것이다.

2. 사용자 정의 노출 규칙 적용

만약 패키지 구조를 변경하기 어렵다면, `user` 모듈의 `package-info.java` 파일을 사용하여 명시적으로 해당 패키지를 외부에 노출하도록 설정할 수 있다. `@ApplicationModule(type = EXPOSED)` 또는 `@NamedInterface` 등을 활용하여 특정 패키지를 외부에 공개하도록 설정할 수 있다.

그러나 이 방법은 전체 패키지를 노출하므로, 노출 범위가 너무 넓어지는 것을 원치 않는다면 **1번 방법(API 패키지 사용)**을 권장한다.

#### 최종 해결책: 공통 api 모듈 도입

1번 방안의 아이디어를 확장하여, 헥사고날 아키텍처에서의 모듈 간 통신 문제를 해결하기 위해 별도의 전역 `api` 모듈을 구현했다.

- 각각의 도메인 모듈은 `common` 모듈과 `api` 모듈을 의존하며, 도메인 모듈 간의 직접적인 의존은 제거했다.
- `api` 모듈에 `External Port`들을 정의하고, 각 모듈이 이를 구현(Implement)하거나 주입받아 사용하도록 설계했다.

데드라인이 임박한 상황에서 아키텍처를 뒤엎고 `api` 모듈을 분리하는 과정이 매우 힘들었다. 하지만 이 과정을 통해 **"진정한 의미의 모듈화와 캡슐화"**가 무엇인지 깊이 있게 체득할 수 있었다.

<br>

## ✨ 느낀 점 - 헥사고날 아키텍처와 팀 협업: 시행착오를 통한 성장

### 1. Port의 개념 재정립: In과 Out의 경계

프로젝트 초기에는 Port의 In/Out 개념을 명확히 정의하지 못한 채 시작했다. 단순히 모듈(BC)의 경계를 기준으로 들어오면 In, 나가면 Out이라고 생각했으나, 구현 과정에서 헥사고날의 진정한 의미를 깨달았다.

- Inbound Port: 외부(Presentation/Application 계층)에서 도메인 내부로 진입하는 통로
- Outbound Port: 도메인이 외부 데이터(Infrastructure/Persistence 계층)에 접근하기 위해 나가는 통로

도메인을 중심에 두고 의존성 방향을 제어하는 헥사고날의 핵심을 뒤늦게나마 이해하고 구현에 반영할 수 있어 다행이었다.

### 2. 설계와 현실 사이의 간극

팀 회고를 통해 헥사고날 아키텍처 도입에 따른 여러 명암을 확인할 수 있었다.

- 복잡도 증가: 도메인의 순수성을 지키려는 목표 때문에 패키지 구조가 파편화되고 코드량이 급격히 늘어났습니다.
- 의존성 관리의 어려움: 아키텍처가 익숙하지 않아 Application 계층에서 의도치 않게 외부 모듈을 의존하는 실수가 발생하기도 했다.
- 오버 엔지니어링에 대한 고민: "아직 확장성이 불확실한 초기 단계에서 너무 과한 설계를 택한 것이 아닌가"라는 우려도 있었다.

### 3. 기술적 부채와 구현의 아쉬움

모듈 간 통신은 동기 방식(Port)과 비동기 방식(Event)을 혼합하기로 계획했다. 특히 `Spring Modulith`를 활용해 이벤트 처리의 안정성을 높이고자 했으나, 실제 구현 과정에서는 한계가 있었다.

나는 유저(User)와 인증/인가(Auth), 쿠키 관리 로직을 담당했는데, 두 도메인이 분리되어 있다 보니 물리적인 작업량이 예상보다 두 배 이상 많아졌다. 결국 마감 기한 내에 안정적인 구현을 위해 비동기 이벤트 처리를 포기하고 전량 동기식으로 구현하게 된 점이 큰 아쉬움으로 남는다. 이 부채는 추후 MSA 분리 단계에서 리팩토링할 예정이다.

바쁜 일정 탓에 코드 리뷰와 Issue/PR 문서화에 소홀했던 점 역시 반성하고 있다. 이제 안정화 단계에 접어든 만큼, 다음 달에는 질 높은 문서화와 리뷰 문화를 정착시키는 데 집중할 계획이다.

### 4. 협업 문화에 대한 성찰

이번 프로젝트에서 가장 뼈아픈 실수는 '공부'에 치중하느라 '협업'을 뒷전으로 미룬 것이다.

- 소통의 부재: 매일 아침 데일리 스크럼을 통해 작업 현황과 이슈를 공유했다면 프론트엔드와의 협업이 훨씬 매끄러웠을 것이다.
- 도구 활용의 미흡: GitHub Organization을 분리하기보다 하나로 통합하고, GitHub Projects를 적극적으로 활용했다면 서로의 진행 상황을 실시간으로 파악하기 쉬웠을 것이다.

### 맺으며

헥사고날 아키텍처라는 새로운 기술에 매몰되어, 팀 프로젝트의 본질인 '팀 문화'와 '원활한 소통'에 소홀했던 점이 개인적으로 큰 아쉬움으로 남는다. 하지만 이번 시행착오를 통해 기술은 결국 팀과 서비스의 문제를 해결하기 위한 수단임을 다시 한번 절감했다. 다음 프로젝트에서는 기술적 완성도만큼이나 팀의 호흡을 중요하게 생각하려 한다.
