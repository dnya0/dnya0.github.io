---
title: Spring Rest Docs 시작하기
author: dnya0
date:   2024-07-14 19:58:00 +0900
categories: [Backend, Spring]
tag: [Spring, Java, RestDocs]
---

프로젝트를 진행하면서 API를 자동으로 문서화할 수 있지 않을까 고민하였다. 그동안 노션에 직접 적고 있었는데 직접 관리를 해줘야한다는 단점이 있었다. 별거 아닐 수 있지만 꽤나 큰 단점이다. 직접 시간을 들여 작성을 해야하고, 오타나 실수로 누락되는 부분이 있을 수 있기 때문이다. 나는 덜렁거리는 성격이라 깜빡하는 일이 많기에 해결 방법으로 자동으로 문서화하도록 전환하기로 하였다.

![노션에 정리된 api 문서](https://github.com/dnya0/dnya0.github.io/assets/84761609/38dbfe35-9883-4217-bd8d-5dc2015b0f7c)

API 자동 문서화를 찾아본 결과, Swagger와 Spring Rest Docs를 찾을 수 있었다.

<br>

## Spring Rest Docs와 Swagger 비교

Spring REST Docs는 테스트 코드 기반으로 RESTful 문서 생성을 도와주는 도구이다.

| | Spring Rest Docs | Swagger |
| :---: | :--- | :--- |
| 장점 | 제품코드에 영향 없다. | API 를 테스트 해 볼수 있는 화면을 제공한다. |
| | 테스트가 성공해야 문서작성된다. | 적용하기 쉽다. |
| 단점 | 적용하기 어렵다. | 제품코드에 어노테이션 추가해야한다. |
| | 제품코드와 동기화가 안될수 있다. |

> 참고: [우아한 테크 블로그](https://techblog.woowahan.com/2597/)

원래 쉽고 빠르게 Swagger를 사용하려 하였으나 제품코드와 동기화가 안될 수 있다는 단점과 Spring REST Docs는 테스트 코드를 통과한 API만 문서에 반영준다는 점으로 Spring Rest Docs를 사용하기로 결정하였다.

<br>

## Project Spec

| Stack | Info |
| --- | --- |
| SpringBoot | 3.2.1 |
| Java | 17 |
| Gradle | 8.5 |


<br>

## Spring Rest Docs 적용하기

> build.gradle은 [공식 문서](https://docs.spring.io/spring-restdocs/docs/current/reference/htmlsingle/)를 찾아 적용하였다. 

내가 적용한 build.gradle은 이렇다. 공식문서와는 조금 차이가 있을 수 있다.

```
buildscript {
    ext {
        snippetsDir = file('build/generated-snippets')
    }
}

plugins {
    id 'java'
    id 'org.springframework.boot' version '3.2.1'
    id 'io.spring.dependency-management' version '1.1.4'
    id "org.asciidoctor.jvm.convert" version "3.3.2"
}

configurations {
    asciidoctorExt
}

group = 'toyproject.genshin'
version = '0.0.1-SNAPSHOT'

java {
    sourceCompatibility = '17'
}

repositories {
    mavenCentral()
}

test {
    useJUnitPlatform()
}

ext {
    snippetsDir = file('build/generated-snippets')
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-validation'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    runtimeOnly 'com.mysql:mysql-connector-j'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'

    //rest docs
    asciidoctorExt 'org.springframework.restdocs:spring-restdocs-asciidoctor'
    testImplementation 'org.springframework.restdocs:spring-restdocs-mockmvc'

    ...

}

asciidoctor {
    inputs.dir snippetsDir
    dependsOn test // asciidoctor를 수행하기전 test를 수행하도록 선언
    configurations 'asciidoctorExt'
    baseDirFollowsSourceFile()
}

tasks.named('test') {
    useJUnitPlatform()
    outputs.dir snippetsDir
}

tasks.register('copyDocument', Copy) { // (12)
    dependsOn asciidoctor
    from file("${asciidoctor.outputDir}")
    into file("src/main/resources/static/docs")
}

build {
    dependsOn copyDocument
}

```

문서를 업데이트하려면

![image](https://github.com/user-attachments/assets/c8a41cf8-9635-4466-bcfa-d5e342716b02)

build 또는 test를 하면 된다.

<br>

## 테스트 메서드 작성

Controller Test에 `andDo()` 메서드 안에 작성하고 싶은 내용을 넣으면 된다.

```java
@Slf4j
@AutoConfigureRestDocs
@WebMvcTest(CharactersController.class)
public class CharacterControllerTest {

    private static final String STRING = "String";
    private static final String NUMBER = "Number";
    private static final String ARRAY = "Array";

    @Autowired
    protected RestDocumentationResultHandler restDocs;
    @Autowired
    protected MockMvc mockMvc;
    @Autowired
    protected ObjectMapper objectMapper;

    @MockBean
    private CharactersService charactersService;

    @Test
    @WithMockUser(username = "user", roles = {"GUEST"})
    public void getCharacterListTest() throws Exception {
        //give
        List<Stars> stars = List.of(Stars.FIVE);
        List<Country> countries = List.of(Country.INAZUMA, Country.MONDSTADT);
        List<Element> elements = List.of(Element.ANEMO, Element.ELECTRO);

        CharacterListRequest request = new CharacterListRequest(stars, countries, elements, null);

        Characters testCharacter1 = createCharacters("test", Country.INAZUMA, Element.ANEMO);
        Characters testCharacter2 = createCharacters("test2", Country.MONDSTADT, Element.ELECTRO);

        Pageable pageable = PageRequest.of(0, 20);
        List<CharacterListResponse> characterResponse = createCharacterResponse(testCharacter1, testCharacter2);

        //when
        when(
                charactersService.findAndCreateCharacterList(any(CharacterListRequest.class), eq(pageable))
        ).thenReturn(new PageImpl<>(characterResponse, pageable, characterResponse.size()));

        //then
        this.mockMvc.perform(get("/api/characters")
                        .params(RequestConverter.convertRequestToMultiValueMap(request))
                )
                .andDo(print())
                .andDo(MockMvcResultHandlers.print())
                .andDo(document("{class-name}/{method-name}",  // 문서 이름 설정
                        preprocessRequest(
                                modifyHeaders()  // 헤더 내용 수정
                                        .remove("Content-Length")
                                        .remove("Host"),
                                prettyPrint()),  // 한 줄로 출력되는 json에 pretty 포멧 적용
                        preprocessResponse(
                                modifyHeaders()
                                        .remove("Content-Length")
                                        .remove("X-Content-Type-Options")
                                        .remove("X-XSS-Protection")
                                        .remove("Cache-Control")
                                        .remove("Pragma")
                                        .remove("Expires")
                                        .remove("X-Frame-Options"),
                                prettyPrint()),
                        queryParameters(
                                parameterWithName("stars")
                                        .description("별")
                                        .attributes(new Attribute("type", ARRAY)),
                                parameterWithName("countries")
                                        .description("지역")
                                        .attributes(new Attribute("type", ARRAY))
                        ),
                        responseFields(
                                fieldWithPath("wrapper[].characterId")
                                        .type(STRING)
                                        .description("캐릭터 id")
                                        .attributes(new Attribute("constraints", "pk"))
                                        .optional()
                        )
                ))
                .andExpectAll(
                        status().isOk(),
                        content().string(containsString("test2"))
                );
    }

    private Characters createCharacters(String name, Country country, Element element) {
        return Characters.builder()
                .characterName(name)
                .element(element)
                .country(country)
                .stars(Stars.FIVE)
                .build();
    }

    private List<CharacterListResponse> createCharacterResponse(Characters... characters) {
        return Arrays.stream(characters).map(CharacterListResponse::of).toList();
    }

}

```

이제 테스트를 실행하면 `build/generated-snnipets` 아래를 보면 `adoc` 형식의 스니펫이 생성된다.

하지만 이 코드의 경우 테스트 코드가 늘어날 수록 중복 코드가 너무 많아진다. 이를 방지하기 위해 먼저 추상화를 진행하겠다.

<br>

## RestDocs 추상화

### RestConfig

테스트 전용 설정 파일을 정의해 문서 이름과 공통적으로 설정할 헤더 설정, json pretty print을 미리 해둔다. 아래의 코드는 테스트 코드를 작성한 {클래스명/테스트 메소드명} 으로 디렉토리를 지정해준 것이다.

다음과 같은 공통 부분을 해결할 수 있다.

- 요청과 응답 json에 pretty format을 적용하는 부분
- 문서 이름을 처리해 주는 부분
- header를 숨기는 부분

```java
@TestConfiguration
public class RestDocsConfig {

    @Bean
    public RestDocumentationResultHandler write() {
        return MockMvcRestDocumentation.document(
                "{class-name}/{method-name}", // identifier
                preprocessRequest(
                        modifyHeaders()
                                .remove("Content-Length")
                                .remove("Host"),
                        prettyPrint()
                ),
                preprocessResponse(
                        modifyHeaders()
                                .remove("Content-Length")
                                .remove("X-Content-Type-Options")
                                .remove("X-XSS-Protection")
                                .remove("Cache-Control")
                                .remove("Pragma")
                                .remove("Expires")
                                .remove("X-Frame-Options"),
                        prettyPrint()
                )
        );
    }

}
```

<br>

### RestDocsSupport

RestDocSupport 추상 클래스를 선언하고 API 문서 테스트 클래스가 이를 상속하게 하자.

`@Disabled`로 테스트 할 클래스에서 해당 클래스 제외해준다. `@Import`로 작성한 RestDocsConfiguration을 추가해준다. `@ExtendWith`에 `RestDocumentationExtension`을 설정해주어 context를 제공해 Spring REST Docs가 잘 작동할 수 있도록 한다.

이 클래스를 상속한 클래스가 `MockMvc`와 `ObjectMapper`를 선언할 필요가 없도록 미리 설정해준다.

`Attribute`를 간단하게 추가할 수 있도록 메서드를 추가한다.

`@BeforeEach`에서 MockMvc의 커스터마이즈를 진행한다. `RestDocsConfig`에서 설정했던 `RestDocumentationResultHandler`를 적용하고, 테스트 시 요청과 응답을 출력할 수 있도록 `print()` 메서드도 적용한다.

```java
@Disabled
@Import(RestDocsConfig.class)
@DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class)
@ExtendWith(RestDocumentationExtension.class)
public abstract class AbstractRestDocsTests {

    @Autowired
    protected RestDocumentationResultHandler restDocs;
    @Autowired
    protected MockMvc mockMvc;
    @Autowired
    protected ObjectMapper objectMapper;

    @BeforeEach
    void setUp(
            final WebApplicationContext context,
            final RestDocumentationContextProvider restDocumentation
    ) {
        this.mockMvc = MockMvcBuilders.webAppContextSetup(context)
                .apply(documentationConfiguration(restDocumentation))
                .alwaysDo(MockMvcResultHandlers.print())
                .alwaysDo(restDocs)
                .addFilters(new CharacterEncodingFilter("UTF-8", true))
                .build();
    }
}
```

> 참고: [Spring REST Docs 사용법](https://dukcode.github.io/spring/spring-rest-docs/)

<br>

### 테스트 코드 리팩토링

이렇게 해도 나는 테스트 코드가 길게 느껴졌다. 더 추상화시키기 위해 Field라는 클래스를 만들어 간단하게 작성할 수 있도록 하였다.

```java
@Getter
public class Field {
    private String path;
    private Object type;
    private String description;
    private String constraints;
    private boolean optional;

    public Field(String path, Object type, String description) {
        this.path = path;
        this.type = type;
        this.description = description;
        this.constraints = null;
        this.optional = false;
    }

    public Field(String path, Object type, String description, String constraints, boolean optional) {
        this.path = path;
        this.type = type;
        this.description = description;
        this.constraints = constraints;
        this.optional = optional;
    }

    public static List<FieldDescriptor> toFieldDescriptors(List<Field> fields) {
        return fields.stream()
                .map(field -> {
                    FieldDescriptor descriptor = fieldWithPath(field.getPath())
                            .type(field.getType())
                            .description(field.getDescription());
                    if (field.isOptional()) {
                        descriptor.optional();
                    }
                    if (field.getConstraints() != null && !field.getConstraints().isEmpty()) {
                        descriptor.attributes(constraints(field.getConstraints()));
                    }
                    return descriptor;
                })
                .collect(Collectors.toList());
    }

    public static List<ParameterDescriptor> toQueryDescriptors(List<Field> fields) {
        return fields.stream()
                .map(field -> {
                    ParameterDescriptor parameterDescriptor = parameterWithName(field.getPath())
                            .description(field.getDescription())
                            .attributes(
                                    Attributes.key("type").value(field.getType())
                            );
                    if (field.isOptional()) {
                        parameterDescriptor.optional();
                    }
                    if (field.getConstraints() != null) {
                        parameterDescriptor.attributes(constraints(field.getConstraints()));
                    }
                    return parameterDescriptor;
                })
                .collect(Collectors.toList());
    }
}
```

<br>

아래는 Field List를 스니펫으로 바꿔주는 코드이다.

```java
public class RestDocsUtil {

    public static QueryParametersSnippet generateRequestParams(List<Field> fields) {
        return queryParameters(Field.toQueryDescriptors(fields));
    }

    public static RequestFieldsSnippet generateRequestFields(List<Field> fields) {
        return requestFields(Field.toFieldDescriptors(fields));
    }

    public static ResponseFieldsSnippet generateResponseFields(List<Field> fields) {
        return responseFields(Field.toFieldDescriptors(fields));
    }

    public static Attribute constraints(String constraint) {
        return key("constraints").value(constraint);
    }
}
```

이제 테스트 코드에 저 코드들을 적용해보자.

```java
@Slf4j
@AutoConfigureRestDocs
@WebMvcTest(CharactersController.class)
public class CharacterControllerTest extends AbstractRestDocsTests {

    private static final String STRING = "String";
    private static final String NUMBER = "Number";
    private static final String ARRAY = "Array";

    @MockBean
    private CharactersService charactersService;

    @Test
    @WithMockUser(username = "user", roles = {"GUEST"})
    public void getCharacterListTest() throws Exception {
        //give
        List<Stars> stars = List.of(Stars.FIVE);
        List<Country> countries = List.of(Country.INAZUMA, Country.MONDSTADT);
        List<Element> elements = List.of(Element.ANEMO, Element.ELECTRO);

        CharacterListRequest request = new CharacterListRequest(stars, countries, elements, null);

        Characters testCharacter1 = createCharacters("test", Country.INAZUMA, Element.ANEMO);
        Characters testCharacter2 = createCharacters("test2", Country.MONDSTADT, Element.ELECTRO);

        Pageable pageable = PageRequest.of(0, 20);
        List<CharacterListResponse> characterResponse = createCharacterResponse(testCharacter1, testCharacter2);

        //when
        when(
                charactersService.findAndCreateCharacterList(any(CharacterListRequest.class), eq(pageable))
        ).thenReturn(new PageImpl<>(characterResponse, pageable, characterResponse.size()));

        //then
        this.mockMvc.perform(get("/api/characters")
                        .params(RequestConverter.convertRequestToMultiValueMap(request))
                )
                .andDo(print())
                .andDo(restDocs.document(
                        RestDocsUtil.generateRequestParams(createRequestParams()),
                        RestDocsUtil.generateResponseFields(createResponseField())
                ))
                .andExpectAll(
                        status().isOk(),
                        content().string(containsString("test2"))
                );
    }

    private Characters createCharacters(String name, Country country, Element element) {
        return Characters.builder()
                .characterName(name)
                .element(element)
                .country(country)
                .stars(Stars.FIVE)
                .build();
    }

    private List<CharacterListResponse> createCharacterResponse(Characters... characters) {
        return Arrays.stream(characters).map(CharacterListResponse::of).toList();
    }

    private List<Field> createRequestParams() {
        return Arrays.asList(
                new Field("stars", ARRAY, "5성/4성", "Enum stars", true),
                new Field("countries", ARRAY, "지역", "Enum Country", true),
                new Field("elements", ARRAY, "원소", "Enum Element", true),
                new Field("weaponTypes", ARRAY, "무기 종류", "Enum WeaponType", true)
        );
    }

    private List<Field> createResponseField() {
        return Arrays.asList(
                new Field("wrapper[].characterId", STRING, "캐릭터 id", "pk", false),
                new Field("wrapper[].characterName", STRING, "캐릭터 이름"),
                new Field("wrapper[].characterImage", STRING, "캐릭터 이미지 경로"),
                new Field("wrapper[].stars", STRING, "캐릭터의 별", "Enum Stars", false),

                new Field("page.currentPage", NUMBER, "현재 페이지"),
                new Field("page.totalPages", NUMBER, "총 페이지"),
                new Field("page.totalElements", NUMBER, "총 아이템 개수"),

                new Field("message", STRING, "메세지")
        );
    }
}
```

더 줄일 수 있을 것이라 생각하지만 우선은 여기서 멈추기로 하였다.

<br>

## 문서화

### 커스텀 스니펫 적용하기

커스텀 스니펫은 `resources/org/springframework/restdocs/templates` 아래에 작성하면 된다. 기존 스니펫에 오버라이딩해서 커스터마이징 한다고 생각하면 된다.

![커스텀 스니펫](https://github.com/user-attachments/assets/52b3b2cc-27ba-4a9a-942b-1f53a83a5cd0)

내가 작성한 스니펫들은 이렇다. 스니펫 코드가 블로그에서 제대로 뜨지 않는 관계로 사진도 추가하였다.

#### query-parameters.snippet

```
{% raw %}
|===
|Parameter|Type|Description|Optional

{{#parameters}}
|{{#tableCellContent}}`+{{name}}+`{{/tableCellContent}}
|{{#tableCellContent}}{{type}}{{/tableCellContent}}
|{{#tableCellContent}}{{description}}{{/tableCellContent}}
|{{#tableCellContent}}{{^optional}}O{{/optional}}{{#optional}}X{{/optional}}{{/tableCellContent}}

{{/parameters}}
|===
{% endraw %}
```

![query-parameters](https://github.com/user-attachments/assets/a832a6bc-add9-4bed-a99f-38f8d2620168)

#### request-fields.snippet

```
{% raw %}
|===
|필드명|타입|필수여부|제약조건|설명
{{#fields}}
|{{#tableCellContent}}`+{{path}}+`{{/tableCellContent}}
|{{#tableCellContent}}`+{{type}}+`{{/tableCellContent}}
|{{#tableCellContent}}{{^optional}}O{{/optional}}{{#optional}}X{{/optional}}{{/tableCellContent}}
|{{#tableCellContent}}{{#constraints}}{{.}}{{/constraints}}{{/tableCellContent}}
|{{#tableCellContent}}{{description}}{{/tableCellContent}}
{{/fields}}
|===
{% endraw %}
```

![request-fields](https://github.com/user-attachments/assets/25b5f127-c279-486c-8f34-53da407c4c73)

#### response-fields.snippet

```
{% raw %}
|===
|필드명|타입|제약조건|설명
{{#fields}}
|{{#tableCellContent}}`+{{path}}+`{{/tableCellContent}}
|{{#tableCellContent}}`+{{type}}+`{{/tableCellContent}}
|{{#tableCellContent}}{{#constraints}}{{.}}{{/constraints}}{{/tableCellContent}}
|{{#tableCellContent}}{{description}}{{/tableCellContent}}
{{/fields}}
|===
{% endraw %}
```

![response-fields](https://github.com/user-attachments/assets/c1fd2dd3-642f-48bc-97a1-c0628499a6b0)

<br>

### 문서 작성

`src/docs/asciidoc`에 문서 작성을 해준다. 예시는 다음과 같다.

#### index.adoc

```
= Spring REST Docs Test
:doctype: book
:source-highlighter: highlightjs
:toc: left
:toclevels: 2
:seclinks:

include::characters.adoc[]
```

#### characters.adoc

```
= Coonect API Document
:doctype: book
:icons: font
:source-highlighter: highlightjs
:toc: left
:toclevels: 2

== Characters 관련 API

=== 캐릭터 리스트 조회

==== 요청

include::{snippets}/character-controller-test/get-character-list-test/http-request.adoc[]

==== 요청 필드

include::{snippets}/character-controller-test/get-character-list-test/query-parameters.adoc[]

==== 응답

include::{snippets}/character-controller-test/get-character-list-test/http-response.adoc[]

==== 응답 필드

include::{snippets}/character-controller-test/get-character-list-test/response-fields.adoc[]
```

이제 코드를 실행하면 아래와 같이 문서가 생성된다.

![image](https://github.com/user-attachments/assets/70bb1ac2-9d1f-4490-aaf2-01650212431f)


![실행화면](https://github.com/user-attachments/assets/8a0df321-d0e1-47cd-845d-4ec39e420a3a)


최종 코드는 [이곳](https://github.com/TeybatGuide/TeybatGuide-BackEnd/tree/feat/rest-docs)에서 확인할 수 있다.