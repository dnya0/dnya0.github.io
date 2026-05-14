---
title: AWS EC2에 접속하기
author: dnya0
date:   2024-04-01 04:17:00 +0900
categories: [TIL, AWS]
tag: [AWS, EC2]
---

## 인스턴스에 접속하기

먼저 pem 키 파일의 권한을 변경한다.

```
$ chmod 600 pem-name.pem
```

그리고 아래의 명령어를 사용하여 ec2에 접속한다. AWS 인스턴스의 종류에 따라 사용자 이름(ec2-user)의 이름이 달라지므로 인스턴스 종류를 잘 확인하고 적어넣는다.

```
$ ssh -i <pem경로> ec2-user@<ec2의 public IPv4 주소 또는 도메인>
```

아래는 성공했을 시 화면이다.

![성공했을 시 화면](https://github.com/dnya0/dnya0/assets/84761609/a8e5b8e4-9c4d-49be-bb04-971ba289ea8f)

위 화면을 나가려면 

```
$ logout
```

을 작성하면 된다.

<br>

## 설정을 통해 SSH 쉽게 접속하기

먼저 pem파일을 `~/.ssh/`로 복사한다.

```
$ cp <pem파일 경로> ~/.ssh/
```

```
$ cd ~/.ssh/
$ cd ll
```

![화면](https://github.com/dnya0/dnya0/assets/84761609/f5192295-d1a7-482f-bfe5-7ab7279d5ef8)

위 명령어를 실행했을 때 pem 키가 잘 복사 되었다면 복사된 pem키의 권한을 변경한다.

```
$ chmod 600 ~/.ssh/pem-name.pem
```

권한을 변경했다면 `~/.ssh`에 config파일을 생성하고 아래 내용을 적는다.

```
$ vi ~/.ssh/config
```

```
# public
Host <본인이 원하는 서비스명>
    HostName <ec2 instance의 public IP주소>
    User ec2-user
    IdentityFile <~/.ssh/pem-name.pem>

# private
Host <본인이 원하는 서비스명>
    HostName <ec2 instance의 private IP 주소>
    User ec2-user
    IdentityFile <~/.ssh/pem-name.pem>
    ssh <public 서버의 서비스명> -W %h:%p
```

public 서버의 Hostname은 개발 머신에서 연결하므로 퍼블릭 IP를 지정한다. 그러나 private 서버의 Hostname은 public 서버로부터 연결하므로 VPC 안에서 이용하는 프라이빗 IP를 지정한다. 생성된 config 파일은 실행 권한이 필요하기 때문에 부여해준다.

```
$ chmod 700 ~/.ssh/config
```

다음 명령어를 작성하면 접속이 성공한다.

```
$ ssh <config에 등록한 서비스명>
```