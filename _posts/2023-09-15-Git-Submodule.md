---
title: Git Submodule 
author: dnya0
date:   2023-09-15 17:23:00 +0900
categories: [TIL, Git]
tag: [Git, Github, Study, Submodule]
---

블로그를 하면서 깃허브의 다양한 기능을 알게되는 것 같다. 오늘은 서브모듈에 대해 알게되었다.

<br>

## 📝 Submodule이란?

Git의 레포지토리 하위에 다른 저장소를 관리하기 위한 도구이다. 이때 상위 레포지토리를 슈퍼 프로젝트(superproject), 하위 레포지토리를 서브 모듈(Submodule)이라고 부른다. 

나는 처음에 이 서브모듈이 그냥 다른 레포지토리를 연결한 라이브러리라는 개념으로 이해했다. 그러나 블로그를 업데이트할 때마다 이 서브모듈을 어떻게 pull해야할지 몰랐다. 서브모듈이란 이름도 몰랐기에 그냥 무작정 검색하다가 '서브모듈'이라는 단어를 발견하게 된 것이다.

![검색기록](https://github.com/dnya0/dnya0/assets/84761609/a345ccc6-f0df-4e1a-9e73-2ba0065b8e42)

<br>

## 🍥 서브모듈이 들어간 레포지토리

### 📌 clone

서브모듈이 포함된 레포지토리는 클론 방법이 기존 방식과는 다르다.

```
git clone --recurse-submodules <레포지토리의 깃 주소>
```

이 방식으로 클론하면 서브모듈 파일 안에 파일들이 보이게 된다.

- 위의 방식으로 클론했을 때

![서브모듈 클론](https://github.com/dnya0/dnya0/assets/84761609/07d7bd38-af44-4d79-979c-bb9a8741c2b1)


- 기존 방식(`git clone <레포지토리의 깃 주소>`)으로 클론했을 때

![기존 방식](https://github.com/dnya0/dnya0/assets/84761609/6651b7e3-4bf4-4fa8-b2f7-5155b8658a36)

<br>

### 📌 초기화

슈퍼 레포지토리에서 pull을 할 경우 서브 레포지토리에 새로 커밋된 내용들이 서브모듈에 pull되지 않는다. 아래는 처음 서브모듈을 사용할 때 초기화하는 과정이다. 최초 1회만하면 된다.

```
$ git submodule init
$ git submodule update
$ git submodule foreach git pull origin main
```

이 상태에서는 로컬만 불러온 상태이기 때문에 add, commit, push를 통해 원격 저장소로 보내줘야 한다.

<br>

### 📌 업데이트

여러 방법이 있는 것 같다.

1. 바로 업데이트하고 머지

```
$ git submodule update --remote --merge
```

2. 업데이트 후 따로 pull 후 add, commit, push

```
$ git submodule update

// 1.

$ git submodule foreach git pull

// 2. 사용자 저장소의 변경 사항이 git pull 을 수행했을 때 merge conflict 을 유발하는 경우가 종종 있으므로, 많은 경우에 conflict 의 가능성을 줄여주는 git pull --rebase 를 통해 사용자의 변경사항이 가장 최신 커밋 이후에 반영되도록 한다.

$ git submodule foreach git pull --rebase
```
