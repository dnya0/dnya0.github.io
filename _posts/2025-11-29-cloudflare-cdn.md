---
title: Cloudflare CDN 설정하기
author: dnya0
date: 2025-11-29 19:00:00 +0900
categories: [Infra, Cloudflare]
tag: [Post, CDN, Cloudflare]
math: true
mermaid: true
image:
  path: ../assets/img/posts-image/cloudflare.jpg
  alt: Cloudflare Image
---

## 🥑 들어가며

서비스를 잘 운영해가던 와중에 서버 로그에 불미스러운 로그가 찍혔다.

```
[2025-11-25 18:43:04.654] [request-c9be46222a2d41788f161d4fe715c7f5] [INFO ] [http-nio-8080-exec-4] day.widdle.widdle.global.log.filter.MDCLoggingFilter - --> GET /tests/vendor/phpunit/phpunit/src/Util/PHP/eval-stdin.php
[2025-11-25 18:43:04.655] [request-c9be46222a2d41788f161d4fe715c7f5] [INFO ] [http-nio-8080-exec-4] day.widdle.widdle.global.log.filter.MDCLoggingFilter - <-- 404 GET /tests/vendor/phpunit/phpunit/src/Util/PHP/eval-stdin.php (1ms)
[2025-11-25 18:43:04.845] [request-44ff4369a08245ec818fbc81990cf3ea] [INFO ] [http-nio-8080-exec-2] day.widdle.widdle.global.log.filter.MDCLoggingFilter - --> GET /test/vendor/phpunit/phpunit/src/Util/PHP/eval-stdin.php
[2025-11-25 18:43:04.846] [request-44ff4369a08245ec818fbc81990cf3ea] [INFO ] [http-nio-8080-exec-2] day.widdle.widdle.global.log.filter.MDCLoggingFilter - <-- 404 GET /test/vendor/phpunit/phpunit/src/Util/PHP/eval-stdin.php (1ms)
[2025-11-25 18:43:05.010] [request-6730b3475622444cb6f17b2af798857a] [INFO ] [http-nio-8080-exec-9] day.widdle.widdle.global.log.filter.MDCLoggingFilter - --> GET /testing/vendor/phpunit/phpunit/src/Util/PHP/eval-stdin.php
[2025-11-25 18:43:05.012] [request-6730b3475622444cb6f17b2af798857a] [INFO ] [http-nio-8080-exec-9] day.widdle.widdle.global.log.filter.MDCLoggingFilter - <-- 404 GET /testing/vendor/phpunit/phpunit/src/Util/PHP/eval-stdin.php (2ms)
[2025-11-25 18:43:05.154] [request-d8bd8f79c7294fb1a310ce4d4582725a] [INFO ] [http-nio-8080-exec-5] day.widdle.widdle.global.log.filter.MDCLoggingFilter - --> GET /api/vendor/phpunit/phpunit/src/Util/PHP/eval-stdin.php
[2025-11-25 18:43:05.155] [request-d8bd8f79c7294fb1a310ce4d4582725a] [INFO ] [http-nio-8080-exec-5] day.widdle.widdle.global.log.filter.MDCLoggingFilter - <-- 400 GET /api/vendor/phpunit/phpunit/src/Util/PHP/eval-stdin.php (1ms)
```

Widdle은 토이 프로젝트로 만들어졌고, Kordle의 내부 로직이 궁금하여 개발하게 된 서비스이다. Kordle이 하루의 한 문제이기 때문에 Kordle을 풀고 난 주변 지인들이 아쉬움에 내 서비스를 이용하곤 했다. 부모님도 치매 예방을 하겠다며 이용했기 때문에 그저 EC2와 Vercel에 배포해놓은 상태로 천천히 유지보수하다가 최근 포텐업 부트캠프 내에서 의도치않은 붐이 일어 다시 버닝하기 시작했다.

그런데 최근 이상한 로그가 찍혀 이에 대해 제미나이에게 물어보니 비정상적인 봇 접근이며, 방치할 경우 보안상 큰 위협이 될 수 있다고 하는 것이다. 나는 그렇게 Cloudflare CDN을 추가하게 되었다.

<br>

## ☁️ Cloudflare CDN

왜 Cloudflare를 선택하였을까? 우선 당장 바로 필요했기 때문에 설정이 간편해야했고, 돈 없는 취준생 입장에서 무료라는 것이 메리트가 꽤 컸다.

Cloudflare는 CDN 캐싱을 통한 웹 사이트의 로딩 혹은 다운로드 속도뿐만 아니라 HTTPS, TLS/SSL를 사용한 기본적인 보안 WAF, DDoS, Bot 관리 등의 보안 기능도 제공한다.

나는 Vercel로 프론트엔드를 배포하고 서버를 EC2로 배포한 상태였다. 내가 할 일은 클라이언트의 요청이 Vercel이 아닌 Cloudflare로 가게 하면 되는 것이었다.

### Domain 연결하기

![](/assets/img/posts-image/2025-11-29-01.png)

우선 Enter an existing domain 칸에 나의 도메인을 입력해준다. 나는 Quick scan for DNS records 상태로 continue 버튼을 눌렀다.

![](/assets/img/posts-image/2025-11-29-02.png)

그럼 위와 같은 플랜 선택 창이 뜬다. 나는 당연히 Free를 골랐다.

![](/assets/img/posts-image/2025-11-29-03.png)

혹시나 문제가 될 경우를 대비해 가려놨다.

내 [Widdle](https://github.com/dnya0/widdle)을 보면 알겠지만 `.day` 도메인을 사용하고 있다. 이 도메인이 AWS Route53에서 지원하는 도메인이 아니기 때문에 가비아에서 구매한 상태였다. 그래서 가비아 네임서버 상태가 아래와 같았다.

![](/assets/img/posts-image/2025-11-29-04.png)

이제 이 가비아 네임서버에 위에서 얻은 Cloudflare 네임서버를 연결해주면 된다.

![](/assets/img/posts-image/2025-11-29-05.png)

### WAF 설정하기

CDN만 연결해서는 문제가 끝나지 않는다. WAF를 설정해줘야 했다.

![](/assets/img/posts-image/2025-11-29-06.png)

나는 이렇게 여러개를 생성해줬다.

#### Block Bot Scans

```
(http.request.uri.path contains "phpunit") or
(http.request.uri.path contains ".php") or
(http.request.uri.path contains "wp-admin") or
(http.request.uri.path contains "wp-content") or (http.request.uri.path contains ".env") or
(http.request.uri.path contains ".git") or
(http.request.uri.path contains "phpmyadmin") or
...
```

제일 먼저 트래픽을 올리는 봇들이 해당 uri들을 계속 불러오려는 시도가 있었기 때문에 uri가 있을 시 block하도록 설정해줬다.

#### Challenge Bad Bots

```
(http.user_agent contains "scanner") or
(http.user_agent contains "bot") or
(http.user_agent eq "")
```

User Agent가 scanner, bot이거나 값이 비어있을 시 block하는 것이 아닌 `Managed Challenge`가 되도록 해주었다. `Managed Challenge`로 설정하면 해당 조건에 걸렸을 때 테스트를 실행시킨다. 점수에 따라 다른데, 점수가 높을 경우 브라우저 증명 같은 비대화형 테스트나 CAPTCHA와 같은 복잡한 테스트를 하기도 한다.

#### Geo Blocking Rule

이 규칙은 CDN을 연결한 뒤 꽤 시간이 지난 뒤에 추가한 것인데, 나의 경우 폴란드와 네덜란드에서 집중적인 봇 공격이 일어났기 때문에 해당 두 국가를 block 시키게 되었다.

```
(ip.src.country eq "NL") or
(ip.src.country eq "PL")
```

VPN을 사용해서 봇을 돌릴 것을 알지만 일단 눈 앞에 트래픽을 막아야 헸기에 이런 결정을 내렸다.

#### Access Rule

이 규칙은 IP를 block 시키기 위해 만들었다.

```
(ip.src eq 194.___.__.___) or
(ip.src eq 35.___.__.___)
```

혹시 몰라서 마스킹 해서 올린다. 해당 아이피 주소에서 계속 비정상적인 uri로 접근하려 했기에 막게 되었다.

#### Global API Rate Limit

이건 혹시 몰라서 추가해놓은 규칙이다. 10초간 30회 이상의 요청이 한 아이피에서 들어올 시 block되도록 설정하였다.

```
(http.request.uri.path contains "/")
```

### EC2 ALB 인바운드, 아웃바운드 규칙 수정

이렇게 설정을 해놓으니 Cloudflare를 우회하여 직접 봇이 요청을 보내는 상황이 벌어졌다. 이를 해결하기 위해 EC2 로드밸런서의 인바운드 규칙을 수정하게 되었다. [Cloudflare Ip 범위](https://www.cloudflare.com/ko-kr/ips/) 사이트에 들어가 EC2 인바운드, 아웃바운드 규칙에 추가해주었다.

먼저 `VPC -> 관리형 접두사 목록`에 [Cloudflare Ip 범위](https://www.cloudflare.com/ko-kr/ips/)를 추가해주었다.

![](/assets/img/posts-image/2025-11-29-07.png)

그리고 EC2 ALB의 인바운드와 아웃바운드에서 해당 목록들을 불러와 추가해주었다.

![](/assets/img/posts-image/2025-11-29-08.png)


<br>

## ✨ 느낀 점

확실히 CDN을 붙여서 관리하니 쓸모없는 로그가 생기지 않아 좋았다.

![](/assets/img/posts-image/2025-11-29-09.png)

![](/assets/img/posts-image/2025-11-29-10.png)

그리고 Cloudflare에 들어가 기록을 보니 확실히 잘 막아주고 있는 것 같아서 안심이 되었다. 또한 Cloudflare에서 시간, IP별로 로그를 볼 수 있어 운영과 유지보수에 큰 도움이 될 것 같다.
