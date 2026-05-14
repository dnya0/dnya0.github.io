---
title: 온플리 개발 후기
author: dnya0
date:   2022-12-28 20:26:00 +0900
categories: [Project, Web]
tag: [Project, Web, OwnPli, DB, Database]
math: true
mermaid: true
image:
  path: https://user-images.githubusercontent.com/84761609/207571757-c7360e46-fce4-48e9-a725-972f8859a2c8.png
---

> [공주대학교 데이터베이스 프로그래밍 프로젝트 후기](https://github.com/youbbin/db-project-ownpli)
{: .prompt-info }



## Info

| Classification | Date       |
|----------------|------------|
| Start Date     | 2022-09-01 |
| End Date       | 2022-12-12 |

| Stack             | Info                                                                      |
|-------------------|---------------------------------------------------------------------------|
| Backend Language  | JAVA                                                                      |
| Idea              | IntelliJ                                                                  |
| Build             | Gradle                                                                    |
| Framework         | Spring, JPA , QueryDSL                                                              |

처음으로 웹 서버 개발을 해본 것이라 코드가 얼레벌레 개발하였다.. 강의만 들었을 땐 쉬울줄 알았는데 생각보다 신경쓸 것이 많았다. 아쉬운 점은 시간이 조금 부족했다는 것.. 

<br>

<hr>

# 아이디어 선정

우리 팀은 모두 음원 스트리밍 사이트에서 음악을 듣고 있었고, 기존 사이트에서 불편한 점을 느끼고 있었다. 

- 다양한 취향 반영한 플레이리스트 생성 불가
- 인기차트 재생 시 싫어하는 가수나 장르의 곡이 포함되어 재생 
- 다른 사용자들이 플레이리스트에 많이 담은 노래 파악 불가

해당 문제점을 바탕으로 우리는 My Own Playlist 줄여서 OwnPli 온플리라는 아이디어를 선정하였다. 온플리는 사용자의 정보에 따라 음악/플레이리스트를 추천, 재생하는 서비스이다.

<br>

<hr>


# 본격적인 개발

## 요구사항 분석

데이터베이스 강의에서 배운대로 먼저 요구사항 분석을 시작했다.

1 온플리(OwnPli)에 회원으로 가입하려면 회원아이디, 비밀번호, 이름, 나이, 성별을 입력해야 한다.

2 회원은 회원아이디로 식별한다.

3 음악에 대한 음악아이디, 제목, 장르, 분위기, 앨범 정보, 가수, mp3 파일 주소, 가사 파일 주소, 이미지 파일 주소, 나라를 유지해야 한다.

4 음악은여러분위기를가질수있고,하나의분위기는여러노래의속성이될수있다.

5 음악은 하나의 장르만을 가질 수 있다.

6 음악은 음악아이디로 식별한다.

7 회원은여러개의플레이리스트를생성할수있고플레이리스트하나는한명의회원만생성할수있다.

8 플레이리스트에 대한 플레이리스트 번호, 플레이리스트 이름, 회원아이디, 음악 유지해야한다.

9 플레이리스트는 플레이리스트 번호로 식별한다.

10 플레이리스트에는 여러 음악을 담을 수 있고, 하나의 음악을 여러 플레이리스트에 담을 수 있다.

<br>

## 개념적 설계

![erd](https://user-images.githubusercontent.com/84761609/220889236-7bae0ad8-785e-403e-bd8a-999a330d51a0.png)

<br>

## 논리적 설계

![db 스키마](https://user-images.githubusercontent.com/84761609/220887016-46fab15d-71a7-49ce-88cd-6f02d4982290.png)

<br>

## 시스템 구성도 및 흐름도

### 시스템 구성도

![구성도](https://user-images.githubusercontent.com/84761609/220889633-12991c8d-7495-44d9-96c2-531b6b384171.png)

### 시스템 흐름도

![흐름도](https://user-images.githubusercontent.com/84761609/220889954-993bd0f9-656e-4b89-bf9a-db3cc812d9fb.png)

<br>

### CRUD

![CRUD](https://user-images.githubusercontent.com/84761609/220890147-8b585791-beeb-4531-8401-2b2d4c36eabd.png)

<br>

## DB 구축

우리는 367개의 곡 정보로 db를 구축하였다. 노래들을 들어보며 노션에 곡 정보와 분위기 등을 직접 입력하였다.

![db](https://user-images.githubusercontent.com/84761609/220890641-0900f8ee-7642-49ab-a59f-8df6f42599b0.png)


## 웹페이지 개발

우리 팀은 전체 프로젝트 진행을 노션에서 진행하였다. 

![노션](https://user-images.githubusercontent.com/84761609/220886369-9887e939-7f20-4db2-837e-36ae23a5d232.png)

하지만 난 노션을 잘 쓸 줄 모르기에.. 깃허브 프로젝트 기능을 주로 사용하였다.

![프로젝트 기능](https://user-images.githubusercontent.com/84761609/220892330-44a53cc3-0461-4d61-9de2-fdade76cd2b6.png)

노션은 아직까지도 익숙해지지 않았다..ㅎㅎ

먼저 나는 웹페이지의 서버를 주로 개발하였고, 다른 팀원들은 리액트를 이용하여 클라이언트단을 맡아주었다. 처음 개발하는 서버라서 많이 헤매고 부족했지만 대부분의 기능은 구현하였다. 아쉽게도 사진 보내고, 음악 재생시키는게 생각보다 너무 어렵더라... 언젠가 관련 기능들만 따로 공부해봐야겠다. 

서버를 개발하면서 가장 고민이 많았던 부분은 음악 필터링에 관한 부분이었다.

![필터링 화면](https://user-images.githubusercontent.com/84761609/207040610-58f164af-e0da-4544-b5e8-5ddcd259044c.png)

정적 쿼리만으론 해당 기능이 구현되지 않아 동적 쿼리에 대해 많이 찾아봤다.. Criteria, Specification 등 무엇을 사용해야할 지 많이 고민해봤지만 시간이 촉박했고, 상대적으로 쉬워보였던 QueryDSL로 결정했다. 결국 해당 기능을 잘 구현해낼 수 있었다.


## 결과물

> [결과물 보러가기](https://github.com/youbbin/db-project-ownpli)

<table style="width:100%">
  <tr>
    <th>Home</th>
    <th>LogIn</th>
  </tr>
  <tr>
    <td><img alt="home" src="https://user-images.githubusercontent.com/84761609/207040582-42769f8c-f7a3-4618-8a73-650f1072fa97.png"/></td>
    <td><img alt="log in" src="https://user-images.githubusercontent.com/84761609/207040598-6bbfdda5-bfd1-411f-abc5-3f26abb9cdcf.png"/></td>
  </tr>
  <tr>
    <th>LogOut</th>
    <th>MusicFiltering 1 - Select Singers</th>
  </tr>
  <tr>
    <td><img alt="log out" src="https://user-images.githubusercontent.com/84761609/207040602-ec1cf1f5-eec5-4c77-aed9-a18fbb32824e.png"/></td>
    <td><img alt="MusicFiltering 1" src="https://user-images.githubusercontent.com/84761609/207040610-58f164af-e0da-4544-b5e8-5ddcd259044c.png"/></td>
  </tr>
  <tr>
    <th>MusicFiltering 2 - Selects</th>
    <th>MyPlaylists</th>
  </tr>
  <tr>
    <td><img alt="MusicFiltering 2" src="https://user-images.githubusercontent.com/84761609/207040608-501029ad-8a28-43a9-bb0f-b045b921537b.png"/></td>
    <td><img alt="MyPlaylists" src="https://user-images.githubusercontent.com/84761609/207040605-5529d8ea-2a48-431c-ae00-e6eeb5b7349d.png"/></td>
  </tr>
  <tr>
    <th>MusicLists in Playlist</th>
    <th>Modifying my Playlist Title</th>
  </tr>
  <tr>
    <td><img alt="플레이리스트 안에 음악들" src="https://user-images.githubusercontent.com/84761609/207040574-3cf140c5-edf3-4109-bf29-18bfd7ff43fb.png"/></td>
    <td><img alt="플레이리스트 제목 정하는 화면" src="https://user-images.githubusercontent.com/84761609/207040579-9553a636-6bd1-4ab1-8887-7bd978e624c5.png"/></td>
  </tr>
  </table>