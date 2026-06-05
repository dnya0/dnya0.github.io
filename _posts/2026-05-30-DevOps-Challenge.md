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

<br>

## Day4 - Script Execution Permissions

### Problem

> In a bid to automate backup processes, the `xFusionCorp Industries` sysadmin team has developed a new bash script named `xfusioncorp.sh`. While the script has been distributed to all necessary servers, it lacks executable permissions on `App Server 3` within the Stratos Datacenter.
>
> Your task is to grant executable permissions to the `/tmp/xfusioncorp.sh` script on `App Server 3`. Additionally, ensure that all users have the capability to execute it.

<br>

### Explanation

이 문제는

> `/tmp/xfusioncorp.sh` 파일에 실행 권한(executable permission)을 추가하라

라는 뜻이다.

<br>

### Answer

```shell
$ ssh banner@stapp03
$ sudo chmod 755 /tmp/xfusioncorp.sh

$ ls -l /tmp/xfusioncorp.sh
-rwxr-xr-x 1 root root 40 Jun 3 14:39 /tmp/xfusioncorp.sh
```

처음에 `+x`로 했는데 틀렸다. 파일을 확인해보니 권한이 `---x--x--x`인 비정상적인 형태였다.

<br>

## Day5 - SElinux Installation and Configuration

### Problem

> Following a security audit, the xFusionCorp Industries security team has opted to enhance application and server security with SELinux. To initiate testing, the following requirements have been established for `App server 1` in the `Stratos Datacenter:`

>   1. Install the required `SELinux` packages
>   2. Permanently disable SELinux for the time being; it will be re-enabled after necessary configuration changes.
>   3. No need to reboot the server, as a scheduled maintenance reboot is already planned for tonight.
>   4. Disregard the current status of SELinux via the command line; the final status after the reboot should be `disabled`.

<br>

### Explanation

> SELinux 관련 패키지를 설치하고, 재부팅 후에는 SELinux가 비활성화(disabled) 상태가 되도록 설정하라

SELinux는 Linux 보안 시스템 중 하나로 정식 명칭은 `Security-Enhanced Linux`이며, 원래는 `National Security Agency(NSA)`에서 개발했다.

보통 Linux 권한은 사용자(user), 그룹(group), 파일 권한(rwx)으로 이루어져 있어 `-rwxr-xr-x`와 같이 누가 파일을 읽을 수 있고 실행시킬 수 있는지 정도로만 관리한다. 그러나 이 방식만으로는 부족할 때가 있어 나온 것이 SELinux이다. 예를 들어, nginx 프로세스가 해킹됨, root 권한 획득, 시스템 파일 접근과 같은 문제가 발생할 경우 기존 권한 체계만으로는 부족하다.

SELinux는 **프로세스가 무엇을 할 수 있는지 강제로 제한**한다. 즉, root라도 모든 것을 다할 수 있는 것이 아니라는 뜻이다.

예를 들어, 일반 Linux에서는 nginx가 root면 뭐든 실행 가능했지만 SELinux는 nginx는 웹 파일만 읽도록, DB 접근 금지, 특정 포트만 사용 가능하게 제한을 걸어놓을 수 있는 것이다.

즉, **사용자 권한 + 프로세스 정책**을 동시에 검사한다. 

그런데 SELinux는 강력하지만 설정 복잡하고 원인 분석과 **접근이 안되는 이유를 찾기 어렵다**는 단점이 있다. 그래서 개발 환경에서는 종종 `disabled`로 꺼두기도 한다.

SELinux 상태는 아래와 같이 있다.

| 상태 | 의미 |
| :---: | :--- |
| Enforcing | 정책 적용 + 차단 |
| Permissive | 차단 안 함, 로그만 기록 |
| Disabled | SELinux 비활성화 |

확인 방법은 `getenforce`를 입력하면 된다.

<br>

### Answer

```shell
$ ssh tony@stapp01
$ sudo yum install -y selinux-policy selinux-policy-targeted
$ sudo vi /etc/selinux/config
# SELINUX=disabled로 고칠 것

#확인
$ getenforce
Disabled
```

<br>

## Day6 - Create a Cron Job

### Problem

> The `Nautilus` system admins team has prepared scripts to automate several day-to-day tasks. They want them to be deployed on all app servers in `Stratos DC` on a set schedule. Before that they need to test similar functionality with a sample cron job. Therefore, perform the steps below:
>
>   a. Install `cronie` package on all `Nautilus` app servers and start `crond` service.
>   b. Add a cron `*/5 * * * * echo hello > /tmp/cron_text` for `root` user.

<br>

### Explanation

모든 App Server에 cron 서비스를 설치하고 실행한 뒤, root 계정에 5분마다 실행되는 cron job을 등록하면 된다.

<br>

### Answer

```shell
# 아래 과정 반복
$ ssh tony@stapp01
$ sudo yum install -y cronie

$ sudo systemctl enable crond
$ sudo systemctl start crond

$ sudo crontab -e
# */5 * * * * echo hello > /tmp/cron_text

$ sudo crontab -l
*/5 * * * * echo hello > /tmp/cron_text
```

<br>

