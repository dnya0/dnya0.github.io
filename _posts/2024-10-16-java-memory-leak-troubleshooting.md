---
title: 자바 메모리 누수 해결해보기
author: dnya0
date:   2024-10-16 10:05:00 +0900
categories: [TroubleShooting, Java]
tag: [Study, Linux, TroubleShooting, Java, MemoryLeak]
---

## 🥑 들어가며

먼저 나는 현재 회사에서 순수 자바코드로 크롤러를 만들고 있다. 크롤러는 멀티 스레드, 멀티 프로세스로 동작하는데 난 셀레니움 파트를 맡아서 개발중이다. 사실 크롤러에서 가장 중요한 부분이기에 인턴인 내가 이걸 맡아도 되나 싶기도 하다. 크롤러 매니저에서 크롤러를 실행·종료시카는데, 크롤러를 강제 중지할 일이 많다. 그래서 크롬과 크롬드라이버 관리를 위해 레디스를 도입하였다. 크롤러에서 크롬과 크롬 드라이버의 포트를 레디스에 저장하여 이후 크롤러를 강제 중지시킬 때 매니저에서 레디스에 있는 값을 찾아 제거시키기로 하였다. 그런데 레디스를 추가하고 코드를 고치는 과정에서 계속 메모리 누수가 일어나는 것 같았다.

<br>

## 🚨 문제 상황

![image](https://github.com/user-attachments/assets/d6751a83-09ab-4de5-92bb-90797c85de7d)

![image](https://github.com/user-attachments/assets/2af5e648-45eb-47cf-aa5e-edbd0e7f3ad0)



10분도 안되는 시간 동안 java의 메모리 점유율이 30%가 넘고, 메모리가 야금야금 올라가는 상황. GC가 실행되지 않고 계속 쌓이는건가 싶어 `System.gc()`를 스레드가 끝날 때 마다 호출하였다. 그러나 이건 급한 불을 끄는 것일 뿐 좋은 해결방안이 아니다. 위의 사진들이 GC를 호출하게 하여 하루동안 크롤러를 돌린 상태이다. 그래서 검색을 해보았고 IntelliJ의 VisualVM이라는 플러그인을 발견하였으나, `cannot open requested application`이라는 오류로 인해 모니터링을 하지 못하는 상황이다. 아직 GC가 꽉 찼다는 메세지를 받은 것도 아니고 OutOfMemory가 뜬 것도 아니지만 메모리 점유율이 높은건 좋은 상황이 아니기에 빠르게 해결해야 한다. eclipse의 Memory Analyzer(MAT)를 사용하기로 결정하였다.

<br>

## 🍥 VisualVM으로 확인하기

IntelliJ에 플러그인을 설치한 후 VisualVM 프로그램을 다운받아주면 준비가 끝난다.

![image](https://github.com/user-attachments/assets/175955d0-13e7-422a-b15b-17ecd6cd99e3)

로컬에서 VisualVM으로 확인해본 결과, 멀쩡하게 동작하지만 어딘가 파란색 그래프가 점점 높아지고 있다는 느낌이 들었다. 확실히 하기 위해 dump 파일을 확인하기로 했다.


<br>

## 🎈 dump.hprof 생성하기

정확도를 올리기 위해 로컬과, 리눅스 원격 서버에서 dump.hprof 파일을 각각 생성하기로 하였다. 로컬은 간단하다. visualVM의 `Heap Dump`버튼을 누르면 파일이 생성된다.

자바 애플리케이션이 실행 중에 메모리 누수 등의 문제가 발생했을 때 관련된 문제를 정리하여 덤프파일을 생성할 수 있다. 상황이 발생했을 때 힙덤프를 생성하는 선택사항(`-XX:+HeapDumpOnOutOfMemoryError`)과 위치를 지정하는 선택사항(`-XX:HeapDumpPath=/var/log`)를 추가한다. 이렇게 설정해놓으면 OutOfMemoryError가 발생했을 때 java_pid{pid}.hprof파일이 지정된 위치에 생성된다.

우선 `System.gc()`를 주석처리한 뒤 `-XX:+HeapDumpOnOutOfMemoryError`와 `-XX:HeapDumpPath=/var/log`를 추가하였다.

```java
...
//메모리 덤프 생성 옵션 설정
processStartCmd.append("-XX:+HeapDumpOnOutOfMemoryError").append(blank);
processStartCmd.append("-XX:HeapDumpPath=/var/log").append(blank);
...
```

```gradle
application {
    applicationDefaultJvmArgs = ['-Dgreeting.language=ko',
                                 '-Xms1024m',
                                 '-Xmx8096m'
                                 , '-XX:+HeapDumpOnOutOfMemoryError', '-XX:HeapDumpPath=/webfilter/logs'
    ]
    mainClass = '<메인클래스 경로>'
}
```

이제 OutOfMemory가 발생할 경우 dump 파일을 확인하면 된다. 나의 경우 주기적으로 JVM을 내렸다가 다시 올려 OutOfMemory가 나려면 시간이 꽤 걸릴 것 같았다 그래서 dump 파일을 생성하기로 결정하였다.

```shell
# pid 확인
$ jps

# dump.hprof 생성
$ jmap -dump:format=b,file=heap_dump.hprof <PID>
Dumping heap to /webfilter/logs/heap_dump.hprof ...
Heap dump file created [2350510466 bytes in 60.630 secs]

$ ll
-rw-------  1 webfilter webfilter 2350510466 10월 16 18:12 heap_dump.hprof
```

이제 이 파일을 이용해 분석을 하면 된다.

<br>

## ✨ Eclipse MAT(Memory Analyzer Tool) 사용해보기

MAT는 자바 힙 분석을 위한 풍부한 기능을 제공하여 메모리 누수 및 메모리 소비 감축요소를 찾을 수 있도록 돕는다.

우선 [Eclipse MAT 사이트](https://eclipse.dev/mat/)에 접속해 본인 운영체제에 맞는 MAT를 다운받는다.

처음 실행을 시키면 아래와 같은 오류가 뜬다.

![image](https://github.com/user-attachments/assets/140d31c4-a051-4426-93ee-39b9e17db0ea)

해결방법은 간단하다. 콘솔로 MAT 경로를 들어가 `MemoryAnalyzer.ini` 파일을 수정하면 된다.

```ini
# ini 파일에 jvm 경로 추가
# -vm 경로는 -vmargs 위에 명시되어야 함
-vm
C:\Program Files\Java\jdk-17\bin
```

만약 `A JRE or JDK must be available in order to run MemoryAnalyzer. No JVM was found after searching the following locations`와 같은 오류가 난다면 JDK를 다시 설치해보도록 하자.

![image](https://github.com/user-attachments/assets/e2e2db96-360d-4acf-ab66-b39be4368843)

설치가 완료되어 MAT 파일을 실행할 시 위와같은 창이 뜬다. 이제 `Open a Heap Dump` 버튼을 클릭하여 동작시키면 된다.

![image](https://github.com/user-attachments/assets/be1faaeb-dd11-4345-93c4-d9eb5a74fc23)

`Leak Suspects Report` 버튼을 클릭한 뒤 `Finish`를 누르면 된다.

![image](https://github.com/user-attachments/assets/b8e31f4b-bf25-4b42-9c55-c094b097df3a)

로컬의 경우 크롤러를 실행시키는 매니저가 실행되지 않아서 그런지 Problem Suspects가 1개로 나왔다. Problem Suspects를 다른 곳에 저장해준 뒤 원격 서버에서 실행시킨 것도 분석을 해보자. 원격 서버에서는 매니저와 크롤러 둘 다 동작하고 있으니 dump.hprof 파일을 각각 생성하기로 결정하였다. manager의 보고서를 생성하고 원격 서버의 크롤러 dump.hprof 파일을 생성을 시도하자 오류가 발생하였다.

<br>

## 🚨 hprof 파일 업로드 에러

### 파일 손상

![image](https://github.com/user-attachments/assets/ce6b8440-91a2-4f0c-a9f3-1be6d1363740)

파일이 생성도중 손상된 것인지 해당 오류가 발생하였다. ini 파일에 옵션을 추가하기 전에 hprof 파일을 다시 생성해서 도전해보기로 하였다. 다시 확인해보니 다운이 다 되지 않았는데 파일 열기를 시도해서 오류가 난 것이었다. 다시 다운받고 업로드를 하려하자 이번엔 메모리에서 오류가 생겼다.

### 메모리 부족

![image](https://github.com/user-attachments/assets/078a395b-44ec-45b7-ade0-b315c5ae2b8e)

이 오류는  Java 힙 메모리가 부족해서 발생한 것이다. 큰 힙 덤프 파일을 분석하려면 MAT의 메모리 할당을 늘려야 한다. 아래는 최종 ini파일이다.

```ini
-startup
plugins/org.eclipse.equinox.launcher_1.6.600.v20231106-1826.jar
--launcher.library
plugins/org.eclipse.equinox.launcher.win32.win32.x86_64_1.2.800.v20231003-1442
-vm
C:\Program Files\Java\jdk-17\bin
-vmargs
--add-exports=java.base/jdk.internal.org.objectweb.asm=ALL-UNNAMED
-Xmx4096m
```

<br>

## 🍡 메모리 누수 추적하기

MAT에서 `Path to GC Roots` 기능을 사용해  GC 루트에서 특정 객체까지의 참조 경로를 추적하여 메모리 누수를 진단할 수 있다. 왼쪽 상단의 `Dominator Tree`를 클릭하여 큰 메모리를 차지하는 객체를 우선적으로 확인한다.

이후 메모리를 많이 차지하고 있는 객체를 우클릭하고 `List Objects > with incoming references`를 클릭한다. 해당 객체가 어디서 참조되고 있는지 확인할 수 있다. 혹은, 해당 객체를 우클릭한 후, `Path to GC Roots > Exclude Weak References`를 선택할 시 GC 루트로부터 해당 객체로의 모든 강한 참조 경로를 추적한다.

분석 결과, main에서 참조하고 있는 queue에 너무 많은 데이터가 들어갔기 때문이었다. 한번 더 dump를 만들어 확인해봐야겠지만 우선 limit를 걸어 일정 개수 이하가 되면 데이터를 100개씩 가져오도록 수정하였다.

<br>

<br>

비록 완벽히 트러블슈팅을 해냈는지는 시간을 두고 더 지켜봐야 하지만 경험했다는 것에 의의를 두고 글을 마치려한다.