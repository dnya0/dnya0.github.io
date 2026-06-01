---
title: DevOps Challenge
author: dnya0
date: 2026-05-30 17:00:00 +0900
categories: [Infra, DevOps]
tag: [Study, DevOps, Linux, KodeKloud]
math: true
mermaid: true
---

## 🥑 들어가며

DevOps로 나아가기 위해 챌린지를 시작했다. [이 곳](https://engineer.kodekloud.com/)에서 진행중이다. 100일 챌린지인데 매일 이 글에 기록하려 한다.


<br>

## Day1 - Linux User Setup with Non-Interactive Shell

### Overview

| Item | Description |
| :---: | :--- |
| Date | 2026-05-30 Sat. |
| Category | Linux / User Management |
| Difficulty | Easy |
| Environment | KodeKloud / Nautilus |
| Keywords | useradd, nologin, non-interactive shell |

<br>

### Problem

> To accommodate the backup agent tool's specifications, the system admin team at `xFusionCorp Industries` requires the creation of a user with a non-interactive shell. Here's your task:
>
> Create a user named `mariyam` with a non-interactive shell on `App Server` 1.

<br>

### Explanation

문제의 요구사항은 

> App Server 1에 mariyam 이라는 사용자를 생성하는데, 로그인 가능한 일반 쉘이 아니라 `비대화형(non-interactive) shell`로 설정하라

는 뜻이다.

보통 리눅스에서 **로그인 못 하는 계정**을 만들 때 사용한다. 예를 들어, 백업 에이전트, 시스템 서비스, 데몬 계정과 같은 것들은 파일 권한만 필요하지 사람이 SSH 로그인할 필요는 없기 때문이다. 일반적으로 아래와 쉘을 사용한다.

- `/sbin/nologin`
- `/usr/sbin/nologin`
- `/bin/false`

일반적으로 `nologin`을 많이 사용한다고 한다.

추가로 `non-interactive shell`은 단순히 “로그인을 막는 것” 이상의 의미를 가진다.

Linux에서는 서비스 실행용 계정이나 시스템 계정을 생성할 때 보안을 위해 로그인 기능을 제한하는 경우가 많다. 만약 일반 shell(`/bin/bash` 등)을 부여하면 해당 계정으로 SSH 로그인이나 shell 접근이 가능해질 수 있기 때문이다.

반면 `nologin`이나 `false`를 사용하면 계정 자체는 존재하지만 interactive shell 세션을 시작할 수 없게 된다. 즉, 프로세스 실행이나 파일 권한 관리 용도로는 사용할 수 있지만 사람이 직접 로그인할 수는 없다.

특히 `/sbin/nologin`은 로그인 시도 시 안내 메시지를 출력한 뒤 종료하며, `/bin/false`는 아무 메시지 없이 즉시 종료된다는 차이가 있다.

실제 운영 환경에서도 다음과 같은 계정들은 대부분 non-interactive shell을 사용한다.

* nginx
* mysql
* postgres
* redis
* ftp 서비스 계정
* backup agent 계정

따라서 이번 문제는 단순 사용자 생성보다는 “서비스 계정을 안전하게 생성하는 방법”을 이해하는 것이 핵심이라고 볼 수 있다.

<br>

### Answer

```shell
$ ssh tony@stapp01
$ sudo useradd -s /sbin/nologin mariyam
$ grep mariyam /etc/passwd
mariyam:x:1001:1001::/home/mariyam:/sbin/nologin
```

<br>

## Day2 - Temporary User Setup with Expiry

### Problem

> As part of the temporary assignment to the Nautilus project, a developer named `ravi` requires access for a limited duration. To ensure smooth access management, a temporary user account with an expiry date is needed. Here's what you need to do:
>
> Create a user named ravi on App Server 3 in Stratos Datacenter. Set the expiry date to `2027-04-15`, ensuring the user is created in lowercase as per standard protocol.
>
> Note: You can find the infrastructure details by clicking on the Details of all Users and Servers button on the top-right section of the page.

<br>

### Explanation

이 문제는

> 특정 날짜까지만 사용할 수 있는 임시 Linux 계정을 생성하라

라는 뜻이다. 지정된 날짜 이후에는 계정이 자동 만료되도록 설정해야 한다.

Linux에서 *외주 인력, 단기 프로젝트 인원, 임시 접근 계정* 같은 상황에서 자주 사용한다.

<br>

### Answer

`-e`를 사용하면 만료 날짜를 지정할 수 있다.

- `-e`: expiry date 지정 옵션

```shell
$ ssh banner@stapp03
$ sudo useradd -e 2027-04-15 ravi

$ sudo chage -l ravi
Last password change                                    : May 31, 2026
Password expires                                        : never
Password inactive                                       : never
Account expires                                         : Apr 15, 2027
Minimum number of days between password change          : 0
Maximum number of days between password change          : 99999
Number of days of warning before password expires       : 7
```

<br>

## Day3 - Secure Root SSH Access

### Problem

> Following security audits, the `xFusionCorp Industries` security team has rolled out new protocols, including the restriction of direct root SSH login.
> 
> Your task is to disable direct SSH root login on all app servers within the `Stratos Datacenter`.

<br>

### Explanation

이 문제는

> 모든 App Server에서 root 계정으로 직접 SSH 로그인하는 걸 막아라

라는 뜻이다. 보안상 `ssh root@stapp01`로 접속하는 것을 금지하려는 것이다. 보통은 일반 사용자로 로그인 또는 sudo 사용 방식을 강제한다.

<br>

### Answer

각 서버에 접속하여 설정을 수정하면 된다.

```shell
$ ssh tony@stapp01
$ sudo vi /etc/ssh/sshd_config
# PermitRootLogin yes에서 no로 변경
$ sudo systemctl restart sshd
```