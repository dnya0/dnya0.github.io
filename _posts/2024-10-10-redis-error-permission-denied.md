---
title: Redis Error - MISCONF Redis is configured to save RDB snapshots...
author: dnya0
date:   2024-10-10 11:16:00 +0900
categories: [TroubleShooting, Redis]
tag: [Redis, Error, Permission]
math: true
mermaid: true
image:
  path: https://blog.kakaocdn.net/dn/RAW9i/btsEizJaYLD/BHuas9r4K6OZD96rI4x94K/img.png
  alt: redis image
---

> [에러 메시지] MISCONF Redis is configured to save RDB snapshots, but it is currently not able to persist on disk. Commands that may modify the data set are disabled, because this instance is configured to report errors during writes if RDB snapshotting fails (stop-writes-on-bgsave-error option). Please check the Redis logs for details about the RDB error.
{: .prompt-danger }

원격 서버에 배포해 테스트하던 중 오류가 발생하였다. 이 오류는 Redis가 RDB 스냅샷을 디스크에 저장할 수 없기 때문에 데이터를 수정하는 명령어를 실행하지 못하는 상황이다. Redis는 `stop-writes-on-bgsave-error` 옵션에 따라, RDB 스냅샷 저장에 실패했을 때 데이터를 수정하는 작업을 중단하도록 설정되어 있다. 이 문제는 주로 디스크 권한 문제, 디스크 공간 부족, 또는 파일 시스템 오류 때문에 발생한다. 따라서 Redis의 로그를 확인해보자.

```shell
# redis.conf에서 redis.log의 파일 위치 확인
$ sudo vi /etc/redis.conf
logfile /var/log/redis/redis.log # '/logfile'을 입력해 경로를 확인한다.

# logfile 확인
$ cat /var/log/redis/redis.log
...
3335648:M 08 Oct 2024 17:50:58.191 # Background saving error
3335648:M 08 Oct 2024 17:51:04.011 * 1 changes in 900 seconds. Saving...
3335648:M 08 Oct 2024 17:51:04.011 * Background saving started by pid 3490665
3490665:C 08 Oct 2024 17:51:04.012 # Failed opening the RDB file dump.rdb (in server root dir /redis/rdb) for saving: Permission denied
3335648:M 08 Oct 2024 17:51:04.112 # Background saving error
3335648:M 08 Oct 2024 17:51:10.031 * 1 changes in 900 seconds. Saving...
3335648:M 08 Oct 2024 17:51:10.031 * Background saving started by pid 3490666
3490666:C 08 Oct 2024 17:51:10.032 # Failed opening the RDB file dump.rdb (in server root dir /redis/rdb) for saving: Permission denied
3335648:M 08 Oct 2024 17:51:10.132 # Background saving error
3335648:M 08 Oct 2024 17:51:16.053 * 1 changes in 900 seconds. Saving...
3335648:M 08 Oct 2024 17:51:16.054 * Background saving started by pid 3490670
3490670:C 08 Oct 2024 17:51:16.054 # Failed opening the RDB file dump.rdb (in server root dir /redis/rdb) for saving: Permission denied
3335648:M 08 Oct 2024 17:51:16.154 # Background saving error
...
```

현재 로그에서 문제가 발생한 경로는 `/redis/rdb`이다. Redis가 해당 경로에 RDB 파일을 저장하려고 하지만, 권한이 없어 저장할 수 없다고 나와 있다. 권한을 수정하고 Redis를 재시작하면 해결된다.

```shell
# 해당 디렉토리를 Redis가 쓰기 권한을 가지도록 설정
# 권한을 확인한 후 필요하면 수정한다.
$ sudo chown redis:redis /redis/rdb
$ sudo chmod 770 /redis/rdb

# Redis 재시작
$ sudo systemctl restart redis
```


