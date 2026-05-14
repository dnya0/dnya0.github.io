---
title: AWS ec2에 Spring Boot 서버 배포하기
author: dnya0
date:   2024-04-06 11:57:00 +0900
categories: [TIL, AWS]
tag: [AWS, ec2, RDS, mysql]
---

## JAR 파일 복사해서 서버 띄우기

### 1. 서버 빌드

```
$ ./gradlew clean build
```

### 2. JAR 파일 원격지에 복사하기

나는 홈에 ec2서버를 설정하였기에 홈으로 하였다.

```
# scp <jar 파일 경로> <host 이름>:<파일이 저장될 ec2 실행경로>
$ scp build/libs/TeybatGuide-0.0.1-SNAPSHOT.jar teybat-web01:~
```

<br>

### 3. ec2 실행 및 jdk 다운

```
$ ssh <host 이름>        # ec2 실행
$ sudo yum update       # yum 업데이트

# jdk17 다운
$ sudo yum install java-17-amazon-corretto
```

<br>

### 4. 인바운드 추가

Spring Boot는 서버 포트가 8080이기 때문에 8080 포트로 들어오는 요청들을 허용해야하므로 이를 추가해준다. 그리고 SSH 와 http, https 도 함께 추가한 뒤 instance에 추가해준다.

![인바운드 규칙](https://github.com/dnya0/dnya0.github.io/assets/84761609/7c4091c5-cdb1-42b3-b9fd-c39efa5e5713)

<br>

### 5. JAR 파일 실행

```
# java -jar <jar 파일 이름>
$ java -jar TeybatGuide-0.0.1-SNAPSHOT.jar
```

<br>

### 오류

> [에러 메시지] com.mysql.cj.jdbc.exception.CommunicationsException: Communications link failure
{: .prompt-danger }

![오류](https://github.com/dnya0/dnya0.github.io/assets/84761609/11075116-2288-4911-b480-e9614e55eebc)

서버가 mysql, jdbc를 사용하기 때문에 RDS 에러를 낸다.

<br>

## ec2에 rds 연결

### mysql 설치

ec2를 실행해준 뒤 mysql을 설치해준다.

```
$ ssh <host 이름> 
$ sudo yum -y install mysql
```

<br>

#### 에러

> [에러 메시지] <br>No match for argument: mysql<br>Error: Unable to find a match: mysql
{: .prompt-danger }

해당 에러 메시지가 떠서 좀 해맸다; 

```
$ yum install http://dev.mysql.com/get/mysql80-community-release-el7-11.noarch.rpm
```

<br>

> [에러 메시지] Error: This command has to be run with superuser privileges (under the root user on most systems).
{: .prompt-danger }

인터넷에 검색한 결과 os가 달라서 오류가 나는 것 같다. linux2023에서 linux2로 변경하여 재생성해주었다.

<br>

> [에러 메시지] Cannot find a valid baseurl for repo: amzn2-core/2/x86_64
{: .prompt-danger }

DNS도 설정해주었는데 계속 해당 오류가 발생하였다. 인스턴스 설정을 다시 보니 private 서브넷에 연결해야 했는데 public 서브넷에 연결하는 실수를 범했다. 아마 이 이유로 계속 DNS 오류가 떴던 것 같다. 재생성해주고 `sudo yum -y install mysql`을 하니 잘 작동한다.

<br>

### mysql 설치가 되지 않을 때

`sudo yum -y install mysql`가 되지 않을 경우 아래와 같이 실행하면 된다.

```
$ sudo yum install https://dev.mysql.com/get/mysql80-community-release-el7-11.noarch.rpm

# 위의 명령어가 듣지 않을 경우 아래 실행
$ wget http://dev.mysql.com/get/mysql80-community-release-el9-5.noarch.rpm
$ sudo rpm -ivh mysql80-community-release-el7-11.noarch.rpm
```

<br>

### rds 연결

이제 미리 rds 대시보드에서 생성한 데이터베이스를 연결해준다. 데이터베이스 메뉴의 상세정보 탭을 들어간다.

![상세정보](https://github.com/dnya0/dnya0.github.io/assets/84761609/ff35c2ad-fd64-4ab8-869d-c21e60c788ab)

인스턴스 정보가 표시되면 **연결 & 보안** 탭을 눌러 정보를 확인한다. **엔드포인트 및 포트**가 연결 대상지 정보이다.

![연결 & 보안](https://github.com/dnya0/dnya0.github.io/assets/84761609/5f1cd88d-e626-4eef-b91b-4e1284786b9f)

엔드포인트에 표시된 텍스트를 복사하여 아래 명령어에 붙여넣는다.

```
$ mysqladmin ping -u admin -p -h <엔드포인트>
```

<br>