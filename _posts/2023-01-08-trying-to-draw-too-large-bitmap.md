---
title: Android Studio - trying to draw too large bitmap
author: dnya0
date:   2023-01-08 21:32:00 +0900
categories: [TIL, Android]
tag: [Android, AndroidStudio, Study, Java, ERROR]
---

> [에러 메시지] trying to draw too large bitmap
{: .prompt-danger }

그냥 카카오 로그인 사진 하나 추가했을 뿐인데 해당 오류가 나타났다.

<br>

## 해결방법

### 하드웨어의 가속 해제

![image](https://user-images.githubusercontent.com/84761609/211195937-72c342b6-cc9d-4f03-bf79-20bc61ab38bf.png)


`AndroidManifest.xml` 파일에 application 내에 해당 코드를 추가한다.

```yml
android:hardwareAccelerated="false"
```

<br>

### 파일 분류

하드웨어 가속을 사용하여야 하는 경우 이미지 파일을 옮기는 것으로 해당 오류를 해결할 수 있다. 해당 오류는 이미지 파일이 너무 크기 때문에 발생하는 오류이기에 높은 해상도 파일로 옮겨준다.

`drawable` 우클릭하여 `Open In`을 클릭 `Explorer`로 들어간다.

![image](https://user-images.githubusercontent.com/84761609/211196134-65000e7e-fb2e-4aba-91cf-e71270988b83.png)


`drawable-xxxhdpi`를 생성하여 문제가 되는 이미지 파일을 옮겨준다.
