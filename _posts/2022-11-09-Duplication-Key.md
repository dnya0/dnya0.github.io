---
title: Spring 중복키 설정
author: dnya0
date:   2022-11-09 16:07:00 +0900
categories: [TroubleShooting, Spring]
tag: [Spring, JPA, Java, Error]
---

JPA 엔티티에 중복키를 선언하려하니 duplicate 오류가 나서 방법을 찾아보았다.

오류가 나는 코드 - `entity/music/MusicMoodEntity.java`

```java
@Data
@Builder
@AllArgsConstructor
@Entity
@Table(name = "Music-Mood")
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class MusicMoodEntity {

    @Id
    private String musicId;

    @Id
    private int moodNum;

}

```

아이디 두 개를 저 방식으로 설정하면 애노테이션이 겹쳐 duplication 에러가 나는 모양이다.

<br>

이 코드를 아래와 같이 두 개의 파일로 나눠야한다.

<br>

아이디 선언 `entity/id/MusicMoodId.jaca`

```java
@Data
@Embeddable
public class MusicMoodId implements Serializable {

    @Column(name = "musicId")
    private String musicId;

    @Column(name="moodId")
    private int moodNum;
}
```

<br>

중복키 사용 - `entity/music/MusicMoodEntity.java`

```java
@Data
@Builder
@AllArgsConstructor
@Entity
@Table(name = "Music-Mood")
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class MusicMoodEntity {

    @EmbeddedId
    private MusicMoodId musicMoodId;

}
```