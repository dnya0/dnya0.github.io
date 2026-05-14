---
title: Linux Swap 파일 삭제 및 재생성 방법
author: dnya0
date:   2024-10-14 18:05:00 +0900
categories: [TIL, Linux]
tag: [Linux, Swap]
---

## 🥑 들어가며

메모리를 확인하니 스왑 파일을 삭제하고 다시 만들어야할 것 같다. 

```shell
$ free -h
              total        used        free      shared  buff/cache   available
Mem:          9.5Gi       3.8Gi       4.3Gi       128Mi       1.4Gi       5.3Gi
Swap:         4.0Gi       2.1Gi       1.9Gi
```

스왑 파일을 삭제할 일이 종종 있는데 계속 찾기가 귀찮아졌다. 많이 쓰면 외워지겠지만 스왑 파일 초기화를 외울 일이 많으면 안되지 않을까..

<br>

## Swap File이란?

리눅스와 같은 운영체제에서 **RAM(주기억장치)**이 부족할 때 임시로 사용하는 가상 메모리 영역이다. 쉽게 말해, 물리 메모리(RAM)가 다 차면 스왑 파일에 데이터를 저장해서 시스템의 안정성과 성능을 유지하는 역할을 한다.

<br>

### 스왑의 동작 원리

1. 메모리 부족 상황 발생 시, 사용 빈도가 낮은 데이터나 프로그램을 RAM에서 스왑 파일로 옮김.
2. 이로 인해 RAM의 여유 공간을 확보해 현재 실행 중인 중요한 작업이 중단되지 않도록 보장.
3. 필요해지면 스왑 파일에 있는 데이터를 다시 RAM으로 불러옴.

### 스왑 공간의 종류

- 스왑 파티션: 디스크의 특정 파티션을 스왑 용도로 사용.
- 스왑 파일: 기존 파일 시스템 내에서 파일을 만들어 스왑 용도로 사용. (더 유연하게 크기를 조정할 수 있다.)

### 스왑 파일의 필요성

- 메모리 부족: RAM이 부족한 경우, 시스템이 멈추지 않고 작업을 지속하게 해줌.
- 멀티태스킹: 여러 프로세스가 동시에 실행될 때 사용 빈도가 낮은 프로세스를 스왑 파일로 옮겨 메모리를 확보.
- 서버 운영: 서버에서는 메모리 부족으로 인한 서비스 중단을 방지하기 위해 스왑을 사용하는 경우가 많음.

### 스왑 파일의 장점과 단점

- 장점:
    - RAM이 부족해도 시스템이 안정적으로 유지됨.
    - 스왑 파일 크기 조정이 쉽고 유연함.
    - 메모리가 부족한 순간에도 프로그램이 종료되지 않음.
- 단점:
    - 디스크 속도가 느리기 때문에 스왑 파일을 사용하는 순간 성능 저하가 발생.
    - 너무 의존할 경우 시스템이 느려지고, 과도한 스왑 사용으로 인해 "스왑 스로싱(Swap Thrashing)" 현상이 나타날 수 있음 (스왑과 RAM 간의 빈번한 데이터 이동으로 성능이 크게 저하됨).

### 언제 스왑 파일을 사용해야 할까?

- RAM이 충분하지 않거나 자주 부족한 경우.
- 서버나 개발 환경에서 갑작스런 메모리 부족 상황에 대비할 때.
- 램 확장이 불가능한 임베디드 시스템 등에서 가상 메모리로 활용.

<br>

## SwapFile 초기화


```shell
# 스왑 활성화 여부 확인
$ swapon --show
NAME       TYPE SIZE   USED PRIO
/swapfile2 file   2G   1.5G   -2
/swapfile  file   2G 548.5M   -3

# 현재 스왑 파일 비활성화
$ sudo swapoff -v /swapfile2
swapoff /swapfile2

# 기존 스왑 파일 삭제
$ sudo rm -f /swapfile2
rm: remove 일반 파일 '/swapfile2'? y

# 새 스왑 파일 생성
$ sudo fallocate -l 2G /swapfile2

# 스왑 파일 권한 설정
$ sudo chmod 600 /swapfile2

# 스왑 영역으로 초기화
$ sudo mkswap /swapfile2
Setting up swapspace version 1, size = 2 GiB (2147479552 bytes)
no label, UUID=3c4ef0a3-c8c8-4414-b210-b69c3c86e4a3

# 스왑 파일 활성화
$ sudo swapon /swapfile2

# 스왑 활성화 여부 확인
$ swapon --show
```

이제 다시 메모리를 확인해보자

```shell
# before
$ free -h
              total        used        free      shared  buff/cache   available
Mem:          9.5Gi       3.8Gi       4.3Gi       128Mi       1.4Gi       5.3Gi
Swap:         4.0Gi       2.1Gi       1.9Gi

# after
$ free -h
              total        used        free      shared  buff/cache   available
Mem:          9.5Gi       5.7Gi       2.3Gi       201Mi       1.5Gi       3.3Gi
Swap:         4.0Gi       548Mi       3.5Gi
```