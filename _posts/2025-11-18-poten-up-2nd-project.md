---
title: 두 번째 프로젝트 회고 - LXP 프로그래밍
author: dnya0
date: 2025-11-18 09:13:00 +0900
categories: [Post, PotenUp]
tag: [Post, PotenUp, Java, Spring, QueryDsl]
math: true
mermaid: true
image:
  path: ../assets/img/posts-image/potenupLogo.png
  alt: Poten Up Image
---

## 🥑 들어가며

포텐업에 들어와 벌써 2달차가 되었다. 두 번째 프로젝트는 스프링을 사용하여 진행하게 되었다. 팀에 아예 처음이신 분이 계셔서 조금 부담스러웠다. 내가 스프링을 사용한 경험은 많지만 그에 대한 지식은 별로 없기 때문에.. 실제로 내 역량보다 너무 많은 기대를 받고 있다 생각하기도 하고...

그래서 이번 프로젝트를 진행하면서 내가 진행한 것과 처음 알게된 것에 대해 간단히 정리해보고자 한다.

<br>

## 🔖 프로젝트 정보

| Category  | Architecture                     |
| :-------: | :------------------------------- |
| Language  | Java 17                          |
|   Build   | Gradle                           |
| Framework | SpringBoot                       |
| Database  | Mysql, Spring Data JPA, QueryDsl |
|   Test    | AssertJ, Jacoco                  |
|  Logging  | Logback                          |
|    etc    | Spotless, Swagger                |

나는 카테고리와 강좌 도메인을 맡게 되었다. 어드민 권한이 없었기 때문에 따로 카테고리의 생성, 수정, 삭제는 만들지 않았다.

### 규칙 논의

코딩 컨벤션의 경우 Spotless를 적용하기로 하였고, 커밋 컨벤션의 경우 아래와 같이 적용하였다. 또한 우리는 gitFlow 전략을 사용하기로 하였다.

![](/assets/img/posts-image/2025-11-18-01.png)

코드 리뷰의 경우 2명 이상이 Approve를 해야 Merge가 가능하게 했으며, 이슈 템플릿과 PR 템플릿을 사용하기로 결정하였다.

### 폴더 구조

```
.
├── {domain name}
│   ├── controller
│   ├── dto
│   │   ├── request
│   │   └── response
│   ├── entity
│   ├── error
│   ├── repository
│   │   ├── querydsl
│   │   └── projection
│   └── service
│       └── validator
└── global
    ├── annotation
    ├── base
    ├── config
    ├── dto
    ├── error
    ├── jwt
    ├── util
    └── log
        ├── config
        └── filter
```

기본적인 폴더 구조는 위와 같다. 이 외에도 추가된 것이 있지만 따로 적진 않겠다.

### ERD

![](/assets/img/posts-image/2025-11-18-02.png)

<br>

## 📌 구현 과정

우선 위에서 잠깐 언급했듯이 내가 맡은 도메인은 카테고리와 강좌였다. 이 외에도 기본 프로젝트 세팅과 MDC 필터를 추가하였다.

### BaseEntity

equals와 hashCode를 오버라이딩하여 Hibernate 프록시 객체로 비교하더라도 잘 동작하도록 구현하였다. `Serializable`을 상속받아 직렬화시켜 혹시 캐시나 세션을 통해 전송될 경우 문제가 없도록 하였다. 지금 생각해보면 `Serializable`까지 상속받는건 너무 오버 엔지니어링이 아니었나 싶다.

```java
@Getter
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class BaseEntity implements Serializable {

    @Serial private static final long serialVersionUID = 1L;

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id", updatable = false, unique = true, nullable = false)
    private Long id;

    @CreatedDate
    @Column(name = "created_at", updatable = false)
    private LocalDateTime createdAt = null;

    @LastModifiedDate
    @Column(name = "modified_at")
    private LocalDateTime modifiedAt = null;

    protected BaseEntity() {}

    public BaseEntity(LocalDateTime createdAt, LocalDateTime modifiedAt) {
        this.createdAt = createdAt;
        this.modifiedAt = modifiedAt;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (Objects.isNull(o) || !(o instanceof BaseEntity that)) return false;
        if (this.id == null) return false;
        return Objects.equals(id, that.id);
    }

    @Override
    public int hashCode() {
        return System.identityHashCode(this);
    }
}
```

또한, 찾아보니 `System.identityHashCode()`의 경우 객체의 메모리 주소를 기반으로 해시 코드를 반환한다고 한다. 따라서 영속성(Persistence) 여부와 관계없이 모든 객체에 대해 고유한 해시 코드를 부여하게 된다.

`equals` 메서드는 ID가 같은 두 엔티티를 동일하다고 판단하지만, `hashCode`는 이 두 동일한 엔티티가 서로 다른 메모리 주소에 있다면 서로 다른 해시 코드를 반환한다.

이는 **"equals()가 true를 반환하면 hashCode()도 같아야 한다"**는 Java의 Object 계약을 위반하게 된다. 아마 코드를 수정하게 된다면 `Serializable`을 제거한 뒤, `hashCode`를 아래와 같이 수정할 것 같다.

```java
@Override
public int hashCode() {
    return id == null ? 31 : Objects.hash(id);
}
```

이 외에도 응답 DTO를 구현하였는데 따로 작성하지는 않겠다.

### Spotless 설정

우리 프로젝트는 Spotless를 사용하여 코드 컨벤션을 맞추기로 하였다. Spotless를 적용하는 것을 내가 맡기로 하였다. Spotless를 적용하는 방법은 그렇게 어렵지 않았다.

```gradle
plugins {
    ...

    id 'com.diffplug.spotless' version '8.0.0'

    ...
}

...

tasks.named('compileJava') {
    dependsOn 'spotlessApply'
}

spotless {
    java {
        googleJavaFormat().aosp()
        importOrder('java', 'javax', 'jakarta', 'org', 'com')
        removeUnusedImports()
        trimTrailingWhitespace()
        endWithNewline()
    }
}

```

`build.gradle`에 위와 같은 코드를 추가한 후

```bash
# 컨벤션을 지키고 있는지 검사
$ ./gradlew spotlessCheck

# 파일에 코드 컨밴션을 적용
$ ./gradlew spotlessApply
```

나의 경우 팀장님이 git hook에 spotlessCheck를 추가하는 법을 알려주셔서 적용하였다.

### MDC logging filter

나는 그리고 요청에 따라 로그를 분리하기 위해 MDC 필터를 추가하였다. 이것도 운영 환경에서 로깅을 위해 추가하는 것이지만, 요청별 고유 ID를 넣어서 로깅을 할 경우 요청으로 인해 생긴 로그와 기본 로그가 한눈에 분리되어 볼 수 있기 때문에 로그의 가독성이 올라간다. 또한 개발 환경에서도 여러 요청이 동시에 처리될 경우가 있을 수 있다고 판단하여, 개발 편의성을 위해 추가하게 되었다.

```java
@Slf4j
@Component
public class MDCLoggingFilter implements Filter {
    private static final String REQUEST_ID = "REQUEST_ID";

    @Override
    public void doFilter(
            ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain)
            throws IOException, ServletException {
        String requestId = "request-" + createUUID();
        long startTime = System.currentTimeMillis();

        try {
            MDC.put(REQUEST_ID, requestId);

            HttpServletRequest httpRequest = (HttpServletRequest) servletRequest;
            HttpServletResponse httpResponse = (HttpServletResponse) servletResponse;

            log.info("--> {} {}", httpRequest.getMethod(), httpRequest.getRequestURI());

            filterChain.doFilter(servletRequest, servletResponse);

            long duration = System.currentTimeMillis() - startTime;
            log.info(
                    "<-- {} {} {} ({}ms)",
                    httpResponse.getStatus(),
                    httpRequest.getMethod(),
                    httpRequest.getRequestURI(),
                    duration);
        } finally {
            MDC.remove(REQUEST_ID);
        }
    }

    private String createUUID() {
        return java.util.UUID.randomUUID().toString().replace("-", "");
    }
}

```

### 카테고리

카테고리 도메인은 2 Depth를 기준으로 하였다. 위의 ERD에서 Categories 테이블을 보면 자기 참조를 하고있다. 카테고리를 대분류, 중분류로만 나눴지만 테이블을 각각 생성하면 너무 낭비라고 생각이 들었다. 그래서 자기 참조를 통해 자식 카테고리가 부모 카테고리를 참조하도록 하였다.

어드민 권한이 없어 Read 기능만 구현하였다. JPA N+1 문제가 일어날 수 있다고 판단하여 JPQL을 사용하여 Join하여 데이터를 불러오도록 구현하였다.

```java

@Repository
public interface CategoryRepository extends JpaRepository<Category, Long> {

    @Query("SELECT c FROM Category c LEFT JOIN FETCH c.children WHERE c.parent IS NULL")
    List<Category> findAllCategoryOptimize();

    @Query("SELECT c FROM Category c LEFT JOIN FETCH c.parent WHERE c.id = :categoryId")
    Optional<Category> findByIdOptimize(Long categoryId);
}

```

### 강좌

강좌의 생성, 수정, 삭제는 그다지 특이한 점이 없었다. 문제는 조회에서 일어났다. 다중 강의 조회의 경우 페이지네이션과 함께 리뷰 별점 높은 순, 수강생 많은 순, 가격 높은 순, 가격 낮은 순, 최신 순 등 필터링을 하는 작업이 필요했다. 따라서 나는 QueryDsl을 도입하기로 결정하였다.

나의 경우 이전에 QueryDsl을 여러번 써보기도 했고 세팅을 해본 경험이 있기 때문에 빠르게 도입하여 구현할 수 있었다. 또한 다른 사람들이 코드를 읽을 때에도 쉽게 이해할 수 있다는 장점이 있다. 이전과 다른 점이라면 따로 객체를 생성해 메서드 체이닝을 이용해 개발을 진행했다는 점과 openFeign에서 만든 querydsl을 사용했다는 점이다.

#### 왜 openFeign?

openFeign에서 만든 api를 사용한 데엔 아래와 같은 이유가 있었다. 기존 querydsl은 취약점이 발견된 상태였고, 24년 1월 이후로 더이상 업데이트가 되지 않는 상태였다.

![](/assets/img/posts-image/2025-11-18-03.png)

그래서 최근 업데이트가 있는 [openFeign에서 개발중인 querydsl](https://github.com/OpenFeign/querydsl)을 사용하기로 결정하였다.

이 querydsl을 개발중인 개발자들도 querydsl이 더이상 업데이트되지 않는다는 사실에 분노하여 만든 듯 하다.

```
Querydsl is at best stale, at worse dead. By the time I made this fork, last commit was one year old and last release over 2 years old.

I reach out to the queryDSL team, but, honestly, they don't care.
```

#### querydsl 세팅

기존 querydsl과 비슷하게 `build.gradle` 파일에 정의한다.

```gradle
configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
    querydsl.extendsFrom compileClasspath
}

...

def queryDslVersion = "7.1"
def querydslSrcDir  = layout.buildDirectory.dir("generated/querydsl").get().asFile

sourceSets {
    main {
        java {
            srcDirs += querydslSrcDir
        }
    }
}

dependencies {
    ...

    // querydsl
    implementation("io.github.openfeign.querydsl:querydsl-core:${queryDslVersion}")
    implementation("io.github.openfeign.querydsl:querydsl-jpa:$queryDslVersion")
    annotationProcessor("io.github.openfeign.querydsl:querydsl-apt:$queryDslVersion:jpa")
}

...

tasks.withType(JavaCompile).configureEach {
    options.getGeneratedSourceOutputDirectory().set(file(querydslSrcDir))
}

clean {
    delete file(querydslSrcDir)
}

```

사용법은 기존 querydsl과 같다.

#### querydsl 구현

이전에 querydsl을 사용할 때는 아래와 같이 한 메서드 안에 쿼리문을 몰아넣는 방식으로 구현했다.

```java
@RequiredArgsConstructor
public class CustomCharacterRepositoryImpl implements CustomCharacterRepository {

    private final JPAQueryFactory queryFactory;

    @Override
    public Page<Characters> findBy___(CharacterListRequest request, Pageable pageable) {
        List<Characters> result = queryFactory
            .selectFrom(characters)
            .where(
                inWeaponType(request.weaponTypes()),
                ...
            )
            .offset(pageable.getOffset())
            .limit(pageable.getPageSize())
            .fetch();

        return PageableExecutionUtils.getPage(result, pageable, getCount(request)::fetchOne);
    }

    ...
}
```

그러나 이 프로젝트에선 따로 객체를 만들어 메서드 체이닝 방식을 사용하였다.

```java
public class CourseQueryRepositoryImpl implements CourseQueryRepository {

    private final EntityManager em;

    public CourseQueryRepositoryImpl(EntityManager em) {
        this.em = em;
    }

    @Override
    public Page<Course> searchAll(CourseSearchRequest request, Pageable pageable) {
        long total =
                new CourseQuery(em, course)
                        .count(
                                inInstructors(request.instructorIds()),
                                inCategories(request.categoryIds()));
        return new CourseQuery(em, course, review, enrollment, user, category)
                .fromCourse()
                .join()
                .whereCourse(
                        inInstructors(request.instructorIds()), inCategories(request.categoryIds()))
                .orderByDefault(request.sortBy())
                .fetchPage(pageable, total);
    }
    ...
}
```

사실 어느 방식이 좋은지는 모르겠다.. 처음에 내가 의도한 바는 중복 쿼리를 만들고싶지 않아서였는데 개발하다보니 오히려 더 복잡해져버린 것 같기도 하고.. 개발 기한이 얼마 남지 않은 상태에서 querydsl을 적용하였기 때문에 깊은 생각을 못했던 것 같다. 기존 방식으로 개발하는게 더 가독성 면에서도, 코드 복잡도 면에서도 좋은 방법이 아닐까.

## 💡 추가로 알게된 것

### Jacoco

Jacoco는 Java Code의 coverage를 측정하는 라이브러리라고 한다. 테스트를 실행하고, 그 결과를 html 파일이나 csv, xml 파일을 통해서 시각화해준다. 팀 내에서 나와 다른 팀원분이 TDD를 해보자 하고 Test Code를 작성하며 개발하고 있었기 때문에 강사님이 한번 써보라고 추천해주셨다.

```gradle
plugins {
    ...

    id 'jacoco'
}


tasks.named('test') {
    useJUnitPlatform()
    finalizedBy 'jacocoTestReport'
}

jacoco {
    toolVersion = "0.8.10"
}

// 리포트 생성
jacocoTestReport {
    dependsOn test
    reports {
        xml.required = true
        html.required = true
    }
}
```

위와 같이 `build.gradle`에 코드를 추가한 뒤 아래의 코드를 터미널에서 실행하면 된다.

```bash
$ ./gradlew --console verbose test jacocoTestReport jacocoTestCoverageVerification
```

이후 `build > reports > jacoco > test > html` 폴더에서 `index.html`을 실행시키면 된다.

![](/assets/img/posts-image/2025-11-18-04.png)

### 멱등성과 멱등키

이건 프로젝트와 관련된 내용은 아니라 짧게 적어보려 한다. 팀장님이 영상을 보던 중 새로운 인사이트를 공유해주셨다.

![](/assets/img/posts-image/2025-11-18-05.png)

멱등성과 멱등키에 관한 내용이였다. 중복 요청이 올 경우 어떻게 처리할 지에 대해서만 궁금해했지 막상 찾아보진 않았었다. 그런데 이 영상을 보고 새로운 정보를 얻게 된 것 같아 너무 좋았다. 지금은 간단히 이런게 있구나 정도만 알고있기도 하고 관련된 내용도 아니니 작성하지 않겠지만 나중에 이에 대해 자세히 공부하고 포스팅을 해보려 한다.

<br>

## 🚨 Issues

Spring Security와 filter를 추가하게 되면서 Controller 테스트가 전부 먹통이 되는 문제가 생겼다. 나는 Spring Security에 대해 그다지 아는 부분이 없기 때문에 많이 헤맸었다. MockUser의 권한으로 인해 문제가 생겼던 것이었다.

나는 Test용 SecurityConfig를 만들어주었다.

```java
class CourseControllerTest extends CourseControllerTestSupport {

    @TestConfiguration
    @EnableMethodSecurity
    public static class TestSecurityConfig {
        @Bean
        public MethodSecurityExpressionHandler methodSecurityExpressionHandler() {
            DefaultMethodSecurityExpressionHandler expressionHandler =
                    new DefaultMethodSecurityExpressionHandler();
            expressionHandler.setDefaultRolePrefix("");
            return expressionHandler;
        }
    }

}
```

또한, 보안 설정(JWT, SecurityConfig)을 모두 비활성화하고, Mock 객체나 테스트용 빈을 사용하여 CourseController가 의존하는 서비스나 레포지토리를 가짜(Mock)로 대체한 후 테스트를 진행하게 하였다.

```java

@WithMockUser(
        username = "1",
        authorities = {"INSTRUCTOR"})
@Import(CourseControllerTest.TestSecurityConfig.class)
@WebMvcTest(
        controllers = CourseController.class,
        excludeFilters = {
            @ComponentScan.Filter(
                    type = FilterType.ASSIGNABLE_TYPE,
                    classes = {JwtAuthenticationFilter.class, SecurityConfig.class})
        },
        excludeAutoConfiguration = {SecurityAutoConfiguration.class})
@DirtiesContext(classMode = DirtiesContext.ClassMode.AFTER_EACH_TEST_METHOD)
public abstract class CourseControllerTestSupport {

    @Autowired protected MockMvc mockMvc;

    @Autowired protected ObjectMapper objectMapper;

    @MockitoBean protected CourseService courseService;

    @MockitoBean protected JwtTokenProvider jwtTokenProvider;
}

```

<br>

## ✨ 느낀 점

Spring Security에 대해 아는 내용이 없었어서 유저 도메인을 맡은 팀원 분을 많이 도와드리지 못했던 것 같아 아쉬웠다. 그리고 TDD로 구현하려고 노력을 많이 하였는데 TDD가 아니었던 것 같다. TDD의 사이클은 Red-Green-Refactor의 사이클로 돌아가야 했는데 나의 경우엔 그냥 구현하면서 테스트 코드를 짜는 행위를 했어서.. 완벽한 TDD가 아니었다. 이 점이 많이 아쉬웠고, 팀에 애자일하게 매일 정해진 시간에 어떤 이슈가 있었고, 어떤 것을 개발하고 있는지 공유하는 시간을 갖자고 건의했어야 했는데 그러지 못했다. 애자일하게 팀을 운영하는데 기여를 못했다는 것이 많이 아쉬웠다.
