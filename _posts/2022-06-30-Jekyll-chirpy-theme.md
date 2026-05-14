---
title: Jekyll chirpy 테마 적용
author: dnya0
date:   2022-06-30 17:06:00 +0900
categories: [TIL, Jekyll]
tag: [Jekyll, Blog, Theme, Error]
---

이제껏 지킬 테마를 많이 바꿔왔지만 이렇게 최악인 테마는 처음이다.
이쁘고 깔끔하지만 os가 windows인 경우 chirpy 테마는 최악이다.
나는 fork도 시도해보고 zip파일도 다운받아서 해봤지만 어느 하나 실행되는 것이 없었다. 둘 다 빌드 오류와 에러 문구만 주구장창 봤다.

구글에 올라온 블로그란 블로그는 다 시도해봤는데 실패했다. 나는 이 테마 하나 적용하겠다고 ruby도 다운받고 linux ubuntu도 다운받았다.

결국 chirpy starter를 사용해서 적용에 성공하였다. 

```shell
bash tools/init.sh
```

해당 명령어를 사용하여 초기화를 해줘야 했는데 
```
Error: Commit unstaged files first, and then run this tool againt.
```
가 뜨면서 작동을 안하는 것이다.

나는 마지막 방법으로 chirpy starter를 사용하였고 마침내 성공했다.
다들 나처럼 쓸데없는 데에 힘빼지 말고 대충 3번 시도해서 안돼면 chirpy starter를 한번 시도해보길 바란다. 

chirpy starter를 사용하면 커스터마이징할 때 오류가 많다는 (해결할 수 있는) 글을 읽고 고민중이라면 starter를 사용하기 바란다. 나는 chirpy starter를 사용하고 왼쪽의 사이드바와 글자 색, 아바타, favicon을 손쉽게 바꿨다. 딱히 대대적인 공사를 할게 아니라 그냥 이미지나 색만 바꾸고 싶다면, 커스터마이징의 범위가 좁다면. 그냥 chirpy starter를 사용하는게 해당 테마를 이용하여 블로그를 만들 수 있는 가장 빠른 지름길이다. 커스터마이징은 대충 구글에 올라온 블로그 2~3개 참고하면 쉽게 가능하다.

<br>

<hr>

<br>

## 1. chirpy starter를 이용해 레포지토리 시작

<br>

이걸 시작하기 전 os가 windows일 경우 ruby를 설치하길 바란다.

먼저 제작자의 설명을 읽어보자.

> [설명 읽으러 가기](https://chirpy.cotes.page/posts/getting-started/)

![image](https://user-images.githubusercontent.com/84761609/176620546-1bef3d4d-a2b5-420b-bab2-bf174eb8c4ec.png)

Option 1의 Chirpy Starter를 클릭하여 레포지토리를 만들고, 로컬 저장소에 클론한다. 

최근에 파일이 바뀐건지 
```shell
bash tools/init.sh
```
를 치지 않아도 괜찮은 것 같다. 이 명령어는 넘어가주자. 폴더에 .travis.yml이 있는지 없는지를 보고 결정하면 될 것 같다. 나는 없었다.

<br>

## 2. _config.yml 수정

> [링크를 참고](https://wlqmffl0102.github.io/posts/Making-Git-blogs-for-beginners-3/)하여 수정해주기 바란다. (url을 꼭 작성해야 한다.) 개인적으로 다른 블로그보다 설명이 더 잘 되어있다고 생각돼서 가져왔다. 

<br>

## 3. Ruby

os가 windows인 사람만 ruby를 사용하고 아닌 사람은 그냥 cmd창을 사용하면 될 것이다. ruby를 작동시켜 블로그의 로컬 저장소로 이동해준다.

```shell
cd C:\Users\username\Documents\GitHub\dnya0.github.io
```

그리고 번들을 설치한다.

```shell
$ bundle
```

<br>

## 4. 깃허브에 올려준다.

git bash를 이용하여

``` shell
$ git add *
$ git commit -m "initialize"
$ git push -u origin master
```

<br>

<hr>

<br>

## ERROR

이후 나는 오류가 하나 더 났었는데

![에러사진](https://user-images.githubusercontent.com/84761609/176625973-8ba57fb2-60ee-46b1-8db6-fc7c08e13dc8.png)

```yaml
Your bundle only supports platforms ["x64-unknown"] but your local platform is
  x86_64-linux. Add the current platform to the lockfile with
  `bundle lock --add-platform x86_64-linux` and try again.
```

이 에러는 스택 오버플로우를 보고 해결하였다.

ruby로 로컬 저장소를 들어가
```shell
$ bundle lock --add-platform ruby
$ bundle lock --add-platform x86_64-linux
```
를 쳐준 뒤 다시 커밋 & 푸쉬해준다.

<br>

나는 커스터마이징은 쉽게 할 수 있었기에 따로 적지는 않겠다. 구글에 검색하여 블로그 몇 개 들어가서 따라하다보면 해결될 것이다. 나는 2개정도 참고한 것 같다.
