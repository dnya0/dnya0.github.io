---
title: PostgreSQL CDC 파이프라인 톺아보기
author: dnya0
date: 2026-09-17 09:00:00 +0900
categories: [Backend, CS]
tag: [Study, PostgreSQL, CDC, WAL, LogicalDecoding, Debezium, Kafka]
math: true
mermaid: true
---

## 🥑 들어가며

PostgreSQL CDC에 대해 공부하다가 WAL이라는 용어를 발견했고, WAL을 이해하려니 Logical Decoding이 딸려왔고, Logical Decoding을 이해하려니 다시 CDC 전체 그림이 필요해졌다. 세 개념을 각각 따로 정리하다 보니 결국 같은 그림(`OLTP → WAL → Logical Decoding → CDC → Kafka/S3 → OLAP`)을 계속 반복해서 그리고 있다는 걸 깨달았다. 그래서 이번엔 세 개를 하나의 흐름으로 묶어서 정리해보려 한다.

<br>

## 1. CDC란, 그리고 왜 필요한가

**CDC(Change Data Capture)**는 데이터베이스에서 발생하는 `INSERT`, `UPDATE`, `DELETE` 같은 변경 사항을 감지해서 다른 시스템에서 쓸 수 있도록 전달하는 기술/패턴을 말한다.

```txt
Application
     │
     ▼
PostgreSQL
     │
     │ INSERT / UPDATE / DELETE
     ▼
    CDC
     │
     ▼
변경 데이터
```

서비스가 작을 때는 PostgreSQL 하나만 있으면 되지만, 서비스가 커지면 같은 데이터를 여러 시스템이 필요로 하게 된다.

```txt
PostgreSQL
    │
    ├── 데이터 분석
    ├── Data Warehouse
    ├── 검색 시스템
    ├── 추천 시스템
    └── Data Lake
```

이 시스템들이 전부 운영 PostgreSQL을 직접 조회하도록 만들 수도 있지만, OLTP DB는 실제 트랜잭션 처리가 최우선이다. 분석/배치 쿼리가 계속 붙으면 CPU, Memory, Disk I/O에 영향을 줄 수 있다. 그래서 운영 DB의 변경 사항만 별도 흐름으로 빼내는 게 CDC다.

```txt
                   OLTP
                PostgreSQL
                    │
                    │ 변경 데이터
                    ▼
                   CDC
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
        Kafka      S3        DW
```

### Polling으로는 안 될까?

가장 단순한 방법은 주기적으로 `SELECT`를 날리는 것이다.

```sql
SELECT *
FROM orders
WHERE updated_at > :last_checked_time;
```

구현은 쉽지만 한계가 뚜렷하다.

- DB에 지속적인 조회 부하 발생
- Polling 주기만큼의 지연
- `updated_at` 같은 별도 기준 컬럼이 필요
- `DELETE`는 애초에 감지하기 어려움 (Row가 사라져버리니까)
- Polling 사이의 중간 상태를 놓칠 수 있음

예를 들어 10초 주기로 Polling하는데 그 사이에 이런 일이 있었다고 해보자.

```txt
10:00:01  CREATED
10:00:03  PAID
10:00:05  CANCELLED

10:00:10  Polling
```

최종 Row만 조회하면 `CANCELLED`만 보이고 `PAID`라는 상태를 거쳤다는 사실 자체를 놓친다. CDC는 **현재 상태를 반복 조회**하는 대신 **발생한 변경 자체를 추적**하는 데 초점을 맞춘다. 그리고 PostgreSQL은 이미 모든 변경을 추적하고 있는 로그를 갖고 있다 — 바로 WAL이다.

<br>

## 2. WAL: 변경을 안전하게 기록하는 방법

**WAL(Write-Ahead Log)**은 데이터베이스의 실제 데이터 파일을 변경하기 전에, **변경 내용을 로그에 먼저 기록하는 방식**이다. 다음 쿼리를 생각해보자.

```sql
UPDATE accounts
SET balance = 500
WHERE id = 1;
```

단순하게 생각하면 PostgreSQL이 디스크의 데이터 파일을 바로 고칠 것 같지만, 실제로는 이렇게 동작한다.

```txt
Application
    │
    │ UPDATE
    ▼
PostgreSQL
    │
    ├── 1. 메모리의 데이터 페이지 변경
    │
    ├── 2. WAL Record 생성
    │
    ├── 3. COMMIT 시 필요한 WAL을 디스크에 기록
    │
    └── 4. 변경된 데이터 페이지는 이후 디스크에 기록
```

핵심 규칙은 하나다.

> **변경된 데이터 페이지가 디스크에 반영되기 전에, 그 변경을 복구할 수 있는 WAL 레코드가 먼저 안전하게 기록되어 있어야 한다.**

그래서 이름이 **Write-Ahead** Logging이다.

### 왜 데이터를 바로 디스크에 쓰지 않을까

PostgreSQL의 데이터는 8KB 단위의 **Page**로 관리되고, 필요한 Page를 메모리(**Shared Buffers**)에 올려서 사용한다. `UPDATE`가 발생할 때마다 Page 전체를 즉시 디스크에 기록한다면 I/O 비용이 커진다.

```txt
                    Memory
                Shared Buffers

Before
┌────────────────────────┐
│ Data Page #42          │
│ id=1, balance=1000     │
└────────────────────────┘
            │
            │ UPDATE
            ▼
After
┌────────────────────────┐
│ Data Page #42          │
│ id=1, balance=500      │
└────────────────────────┘
```

이 상태 — 메모리는 바뀌었지만 디스크에는 아직 반영되지 않은 Page — 를 **Dirty Page**라고 한다. 대신 PostgreSQL은 변경된 Page를 메모리에 잠시 들고 있고, **COMMIT에 필요한 WAL만 먼저 디스크에 기록**한다.

> 참고로 실제 PostgreSQL의 `UPDATE`는 기존 Row를 `1000 → 500`으로 덮어쓰는 게 아니라 MVCC에 따라 새 Tuple Version을 만드는 방식으로 동작한다. 여기서는 WAL과 Buffer의 관계를 설명하기 위해 단순화했다.

### WAL Flush와 Data Page Flush는 다른 타이밍이다

메모리의 내용을 디스크에 쓰는 작업을 **Flush**라고 하는데, WAL을 이해할 때는 두 종류의 Flush를 구분해야 한다.

```txt
Memory

┌──────────────────────┐
│ Shared Buffers       │
│ Dirty Data Page      │
└──────────────────────┘

┌──────────────────────┐
│ WAL Buffers          │
│ WAL Records          │
└──────────────────────┘
          │
          │ ① WAL Flush (COMMIT 시점)
          ▼
Disk
┌──────────────────────┐
│ WAL Files            │
└──────────────────────┘

                 ↓
            COMMIT 성공
                 ↓
             시간이 지나
                 ↓
        ② Data Page Flush (Checkpoint 등)
                 ↓
        ┌─────────────────┐
Disk    │ Data File       │
        └─────────────────┘
```

**COMMIT됐다는 것과 Data Page가 디스크에 반영됐다는 것은 같은 의미가 아니다.** WAL이 먼저 안전하게 기록되기만 하면, 실제 Data Page는 나중에 **Background Writer**나 **Checkpoint** 과정에서 디스크에 반영해도 된다. 이 덕분에 매 트랜잭션마다 Data Page 전체를 디스크에 쓸 필요가 없어진다.

### 서버가 죽으면?

WAL이 디스크에 있고 Data Page가 아직 Dirty 상태인 채로 서버가 꺼졌다고 해보자.

```txt
💥 Crash
   ↓
PostgreSQL Restart
   ↓
WAL 확인
   ↓
필요한 WAL Record 재생
   ↓
Data 복구
```

WAL에 변경 기록이 남아있기 때문에 재시작 시 그 기록을 재생해서 복구할 수 있다. 이게 WAL이 ACID의 **Durability(지속성)**를 보장하는 핵심 메커니즘이다. 그리고 이 복구 과정을 처음부터 끝까지 반복하지 않도록, 어느 시점까지의 Data Page는 이미 디스크에 반영됐다고 표시해두는 지점이 **Checkpoint**다.

한 가지 분명히 해둘 게 있는데, **WAL은 실행됐던 SQL을 그대로 저장하는 로그가 아니다.** PostgreSQL 내부에서 장애 복구와 복제를 위해 만드는 내부 표현이다. 그래서 외부 시스템이 WAL을 그대로 읽어도 `users 테이블 id=10이 수정됐다` 같은 의미를 바로 알아낼 수 없다 — 이걸 해석해주는 게 다음 섹션의 **Logical Decoding**이다.

<br>

## 3. WAL의 두 갈래: Physical과 Logical

PostgreSQL이 WAL을 활용하는 방식은 크게 둘로 나뉜다.

```txt
                   WAL
                 /     \
                ▼       ▼
          Physical     Logical
        Replication    Decoding
              │             │
              ▼             ▼
           Replica          CDC
```

- **Physical Replication**: WAL을 물리적인 변경 그대로 재생해서 Primary와 동일한 Replica를 만든다. "어떤 테이블의 어떤 Row가 바뀌었는가"보다 "Primary의 변경을 그대로 재현한다"에 가깝다.
- **Logical Decoding**: WAL을 테이블/Row 단위의 논리적인 변경으로 해석한다. 여기서부터 CDC로 이어진다.

| 구분 | Physical Replication | Logical Decoding 기반 |
| --- | --- | --- |
| 기준 | PostgreSQL 물리적 변경 | 테이블/Row 변경 |
| 주요 목적 | DB 전체 복제 | 선택적 데이터 복제, CDC |
| 대상 | PostgreSQL Replica | PostgreSQL 또는 외부 Consumer |
| 테이블 단위 선택 | 불가능 | 가능 |
| CDC 활용 | 부적합 | 적합 |

<br>

## 4. Logical Decoding: WAL을 이벤트로 해석하기

**Logical Decoding**은 WAL에 기록된 내부 변경 정보를 테이블/Row 단위의 논리적인 데이터 변경으로 해석하는 기능이다.

```txt
Application
    │
    │ INSERT / UPDATE / DELETE
    ▼
PostgreSQL
    │
    ▼
   WAL
    │
    ▼
Logical Decoding
    │
    ▼
논리적 변경 이벤트

table     = users
operation = UPDATE
id        = 10
name      = Kim
```

예를 들어 다음 변경이 순서대로 발생했다면,

```sql
INSERT INTO users(id, name) VALUES (10, 'Lee');
UPDATE users SET name = 'Kim' WHERE id = 10;
DELETE FROM users WHERE id = 10;
```

Logical Decoding은 이런 변경 스트림을 만들어낸다.

```txt
INSERT users id=10 name=Lee
↓
UPDATE users id=10 name=Kim
↓
DELETE users id=10
```

### Output Plugin: pgoutput vs wal2json

Logical Decoding이 해석한 결과를 어떤 형태로 내보낼지는 **Output Plugin**이 결정한다. PostgreSQL CDC를 실제로 구성할 때 가장 자주 마주치는 선택지가 이 둘이다.

| 구분 | pgoutput | wal2json |
| --- | --- | --- |
| 제공 방식 | PostgreSQL 10+ 기본 내장 | 별도 확장(extension) 설치 필요 |
| 출력 형식 | 바이너리에 가까운 Logical Replication 프로토콜 | JSON |
| 가독성 | 낮음 (사람이 직접 읽기 어려움) | 높음 (디버깅에 유리) |
| 성능/오버헤드 | 상대적으로 가벼움 | 상대적으로 무거움 |
| Debezium 기본값 | 최신 버전 기본 권장 | 과거 많이 쓰였으나 지금은 후순위 |

정리하면, 별도 설치 없이 바로 쓸 수 있고 오버헤드가 적은 `pgoutput`이 요즘은 사실상 기본 선택지고, `wal2json`은 사람이 읽는 JSON 결과가 필요하거나 pgoutput을 못 쓰는 구버전 환경에서 여전히 쓰인다.

### Replication Slot과 LSN

Logical Decoding을 실제 CDC에 쓰려면 **Replication Slot**이 필수다. PostgreSQL은 WAL 파일을 영원히 보관하지 않고 필요 없어지면 제거하는데, CDC Consumer가 아직 읽지 않은 WAL을 제거해버리면 데이터를 유실한다.

```txt
WAL

A ─ B ─ C ─ D ─ E

        ▲
        │
CDC Consumer는
여기까지 처리
```

Replication Slot은 "이 Consumer가 여기까지 처리했으니 그 이후 WAL은 아직 지우면 안 된다"는 정보를 PostgreSQL에게 알려준다. 이 위치를 나타내는 값이 **LSN(Log Sequence Number)**이다. Kafka의 Consumer Offset과 비슷한 역할이라고 생각하면 이해하기 쉽다.

```txt
Kafka      → Consumer Offset
PostgreSQL → WAL LSN
```

Replication Slot 덕분에 Consumer가 필요한 WAL이 너무 일찍 지워지는 걸 막을 수 있지만, 반대로 Consumer가 오래 멈추면 문제가 생긴다.

```txt
CDC Consumer 장애
   ↓
Replication Slot 진행 중단
   ↓
필요 WAL 계속 보존
   ↓
WAL 누적
   ↓
Disk 사용량 증가
```

이 문제는 11장 실전 케이스에서 다시 다룬다.

<br>

## 5. CDC를 구현하는 3가지 방법

CDC는 특정 제품의 이름이 아니라 패턴이기 때문에 구현 방법이 여러 가지다.

```txt
                CDC 구현 방법
        ┌───────────┼───────────┐
        ▼           ▼           ▼
  Timestamp      Trigger      Log-based
   Polling                    (WAL 기반)
```

- **Timestamp 기반 Polling**: 1장에서 다룬 방식. 구현은 쉽지만 한계가 많다.
- **Trigger 기반**: `UPDATE`가 발생하면 DB Trigger가 `change_log` 테이블에 변경을 기록한다. 다만 Trigger가 OLTP 트랜잭션 경로에 끼어들기 때문에 부하와 운영 복잡성이 늘어난다.
- **Log-based CDC**: DB가 원래 장애 복구를 위해 만드는 WAL(PostgreSQL), binlog(MySQL) 등을 그대로 활용한다. 애플리케이션 트랜잭션 경로에 영향을 주지 않기 때문에 현대적인 CDC 시스템에서 널리 쓰인다. 지금까지 다룬 WAL → Logical Decoding 흐름이 바로 이 방식이다.

### Full Load + CDC

CDC는 기본적으로 **새로 발생하는 변경**만 추적한다. 그런데 `orders` 테이블에 이미 1억 건이 쌓여있다면, 그 기존 데이터는 어떻게 옮길 것인가도 고려해야 한다. 그래서 실무에서는 흔히 **Full Load + CDC**를 함께 쓴다.

```txt
             Source DB
                 │
          ┌──────┴──────┐
          ▼             ▼
      Full Load         CDC
          │             │
          ▼             ▼
     기존 데이터      변경 데이터
          │             │
          └──────┬──────┘
                 ▼
               Target
```

1. 기존 데이터를 전체 적재하고
2. 적재 과정 중 발생한 변경을 놓치지 않도록 추적하다가
3. 이후로는 CDC로 변경만 지속 전달한다

AWS DMS도 Full Load Task와 CDC Task를 함께 구성하는 형태를 기본으로 지원한다.

<br>

## 6. CDC 운영에서 실제로 고민하게 되는 것들

- **중복 처리**: 장애/재시도로 같은 변경이 다시 전달될 수 있다. Consumer는 **Idempotency**를 고려해야 한다.
- **순서(Ordering)**: `CREATED → PAID → SHIPPED` 같은 변경이 잘못된 순서로 반영되면 최종 상태가 달라질 수 있다.
- **Schema Evolution**: `ALTER TABLE users ADD COLUMN phone VARCHAR(20);` 같은 변경을 Target이 어떻게 받아들일지 미리 정해둬야 한다.
- **CDC Lag**: Source의 변경이 Target에 즉시 반영된다는 보장은 없다. 대부분의 CDC 파이프라인은 엄밀한 실시간이 아니라 **Near Real-Time**으로 이해하는 게 맞다.
- **Replication Slot / WAL 누적**: 4장 마지막에서 본 문제. 운영 중이라면 지속적으로 모니터링해야 한다.

<br>

## 7. 실전 아키텍처: Debezium + Kafka vs AWS DMS

지금까지의 흐름을 실제 구현체 두 가지로 보면 이렇다.

**Debezium + Kafka 기반**

```txt
PostgreSQL
    ↓
   WAL
    ↓
Logical Decoding
    ↓
Debezium
    ↓
  Kafka
    ↓
 ┌──┴─────────────┐
 ▼                ▼
OpenSearch    Data Warehouse
```

**AWS DMS 기반**

```txt
Application
    ↓
PostgreSQL (OLTP)
    ↓
   WAL
    ↓
AWS DMS
    ↓
   S3
    ↓
Data Lake / OLAP
```

| 구분 | Debezium + Kafka | AWS DMS |
| --- | --- | --- |
| 전달 계층 | Kafka (Event Streaming) | S3 등 AWS 서비스 |
| 실시간성 | 상대적으로 강함, 여러 Consumer가 구독 가능 | 배치성 적재에 가까운 경우가 많음 |
| 운영 부담 | Kafka/Connect 클러스터 직접 운영 | 관리형 서비스라 운영 부담이 적음 |
| 커스터마이징 | Output Plugin, SMT 등 세밀한 제어 가능 | AWS가 정한 설정 범위 내에서만 |

여기서 중요한 건, **Kafka 자체는 CDC가 아니라는 점**이다. Kafka는 Event Streaming Platform이고, Debezium 같은 CDC 도구와 결합했을 때 CDC Pipeline의 전달 계층으로 쓰이는 것이다.

<br>

## 8. CDC vs Outbox Pattern

CDC를 실무에 적용하려고 하면 꼭 한 번은 비교하게 되는 대안이 **Outbox Pattern**이다. 얼핏 보면 둘 다 "DB 변경을 이벤트로 만들어 전달한다"는 목적이 같아 보이지만, 접근 방식이 다르다.

```txt
Outbox Pattern

Application
    │
    │ 하나의 트랜잭션 안에서
    ├── 비즈니스 테이블 UPDATE
    └── outbox 테이블 INSERT (이벤트)
                 │
                 ▼
          별도 프로세스가 outbox를 읽어
          메시지 브로커로 전달
```

Outbox는 **애플리케이션이 명시적으로** "이 비즈니스 이벤트를 발행하겠다"는 의도를 담아 `outbox` 테이블에 적는 패턴이다. 같은 트랜잭션 안에서 비즈니스 데이터와 이벤트를 함께 커밋하기 때문에 "DB에는 저장했는데 이벤트 발행은 실패했다" 같은 정합성 문제를 피할 수 있다. 반면 CDC는 애플리케이션이 이벤트를 만드는 걸 신경 쓸 필요 없이, **DB의 물리적인 변경 자체**를 인프라 레벨에서 캡처한다.

| 구분 | CDC | Outbox Pattern |
| --- | --- | --- |
| 이벤트를 만드는 주체 | DB 변경 자체 (WAL) | 애플리케이션 코드 |
| 적용 범위 | 테이블의 모든 Row 변경 | 명시적으로 기록한 비즈니스 이벤트만 |
| 이벤트 스키마 | Row의 컬럼 구조에 종속적 | 애플리케이션이 원하는 형태로 자유롭게 설계 |
| 트랜잭션 정합성 | DB 커밋 = 곧 캡처 대상 | 같은 트랜잭션에 outbox INSERT를 포함해야 보장 |
| 기존 코드 변경 | 필요 없음 (WAL만 읽으면 됨) | outbox 테이블/기록 로직 추가 필요 |
| 내부 구현 노출 | 테이블 구조가 그대로 이벤트가 됨 | 원하는 만큼만 노출 가능 |

실무에서는 이 둘을 배타적으로 고르기보다, **Outbox 테이블 + CDC**를 함께 쓰는 경우도 많다. `outbox` 테이블에만 CDC를 걸어서(Debezium의 Outbox Event Router가 이 용도다) 애플리케이션은 이벤트 스키마를 제어하면서도, 이벤트 발행 자체는 WAL 기반 CDC의 안정성에 맡기는 방식이다.

<br>

## 9. CDC vs Event Sourcing

비슷하게 자주 헷갈리는 개념이 Event Sourcing이다. 차이는 **무엇이 Source of Truth인가**에 있다.

```txt
CDC
Application → Database → 데이터 변경 → WAL → CDC Event
(DB에서 발생한 결과적인 변경을 캡처)

Event Sourcing
Application → Domain Event → Event Store → 현재 상태 재구성
(Event 자체가 원본 데이터)
```

CDC는 DB 상태 변경이 먼저 일어나고 그 결과를 캡처하는 반면, Event Sourcing은 Event 자체가 저장되는 원본이고 현재 상태는 그 Event들을 재생해서 만들어낸다.

<br>

## 10. DBMS별 비교

WAL이라는 개념 자체는 PostgreSQL 전용이 아니라, 여러 DBMS가 내구성과 장애 복구를 위해 쓰는 일반적인 기법이다. DB마다 이름과 구현이 다를 뿐이다.

| DBMS | 관련 로그 | 주요 용도 |
| --- | --- | --- |
| PostgreSQL | WAL | 복구, Replication, CDC |
| MySQL/InnoDB | Redo Log | 장애 복구 |
| MySQL | Binary Log(binlog) | Replication, CDC |
| Oracle | Redo Log | 장애 복구, 복제 |
| SQL Server | Transaction Log | 장애 복구, 복제, CDC |

PostgreSQL은 WAL 하나가 Crash Recovery, Physical Replication, Logical Decoding까지 전부 담당하는 반면, MySQL은 역할이 나뉘어 있다.

```txt
PostgreSQL                          MySQL / InnoDB

데이터 변경                          데이터 변경
   ↓                                    │
  WAL                        ┌─────────┴─────────┐
   ├── Crash Recovery        ▼                    ▼
   ├── Physical Replication  Redo Log           binlog
   └── Logical Decoding         ↓                  ↓
            ↓              Crash Recovery    Replication
           CDC                                     ↓
                                                   CDC
```

그래서 CDC 관점에서는 이렇게 기억하면 된다.

```txt
PostgreSQL: WAL → Logical Decoding → CDC
MySQL     : binlog → CDC
```

<br>

## 11. 실전 케이스 — Replication Slot이 방치되면 벌어지는 일

CDC를 운영하다 보면 한 번쯤 마주치는 사고 패턴이 있다. Debezium이나 Consumer가 배포/장애로 몇 시간 멈췄는데, 아무도 눈치채지 못하는 경우다.

```txt
Consumer 중단 (배포 실수, OOM 등)
        ↓
Replication Slot의 restart_lsn이 그대로 멈춤
        ↓
PostgreSQL은 그 지점 이후 WAL을 계속 보존
        ↓
평소엔 자동 삭제되던 WAL이 쌓이기 시작
        ↓
pg_wal 디렉터리 용량 증가
        ↓
디스크 풀 → 최악의 경우 PostgreSQL 자체가 쓰기 불가 상태
```

문제는 Consumer가 며칠씩 멈춰 있어도 PostgreSQL 자체는 겉으로 멀쩡해 보인다는 점이다. 그래서 Replication Slot과 WAL 누적량을 직접 모니터링해야 한다.

```sql
-- 현재 Slot 목록과 활성 상태 확인
SELECT slot_name, active, restart_lsn, confirmed_flush_lsn
FROM pg_replication_slots;

-- Slot 때문에 보존 중인 WAL 용량 확인
SELECT slot_name,
       pg_size_pretty(pg_wal_lsn_diff(pg_current_wal_lsn(), restart_lsn)) AS retained_wal
FROM pg_replication_slots;

-- 물리/논리 복제 Consumer의 실시간 지연 확인
SELECT application_name, state, sent_lsn, write_lsn, flush_lsn, replay_lsn
FROM pg_stat_replication;
```

`retained_wal` 값이 계속 증가한다면 Consumer가 멈췄거나 처리 속도가 Source의 변경 속도를 못 따라가고 있다는 신호다. 실무에서는 이 값에 알람을 걸어두거나, 비활성 상태(`active = false`)로 오래 방치된 Slot을 자동으로 감지하는 정도의 안전장치는 CDC 파이프라인을 처음 구축할 때부터 함께 준비해두는 게 좋다.

<br>

## 12. 전체 구조 정리

지금까지의 내용을 하나로 이어보면 다음과 같다.

```txt
                     Application
                          │
                          ▼
                    PostgreSQL
                       OLTP
                          │
                          ▼
                         WAL
                          │
                          ▼
                  Logical Decoding
                          │
                          ▼
                   Replication Slot
                          │
                          ▼
                         CDC
                     ┌────┴────┐
                     ▼         ▼
                Debezium     AWS DMS
                     │         │
                     ▼         ▼
                   Kafka       S3
                     │         │
                     ▼         ▼
                Consumers     OLAP
```

```txt
OLTP              = 서비스 트랜잭션 처리
WAL               = DB 변경을 안전하게 기록 (Durability)
Logical Decoding  = WAL을 논리적인 변경으로 해석
Replication Slot  = Consumer에게 필요한 WAL 보존 및 처리 위치 관리
CDC               = DB의 변경을 캡처해 외부에서 쓸 수 있게 함
Debezium / AWS DMS = CDC를 구현하는 도구/서비스
Kafka / S3        = 변경 데이터의 전달 또는 저장 계층
```

<br>

## 마치며

처음엔 "WAL이 뭐지"에서 시작했는데, 파고들다 보니 Crash Recovery, Replication, Logical Decoding, CDC까지 전부 하나로 연결되는 흐름이라는 걸 알게 됐다. 결국 한 문장으로 정리하면:

> **WAL은 DB의 변경을 안전하게 기록하는 메커니즘이고, Logical Decoding은 그 WAL을 외부가 이해할 수 있는 변경 이벤트로 해석하는 기능이며, CDC는 그렇게 해석된 변경을 Source에서 Target까지 실제로 옮기는 전체 파이프라인이다.**

다음에는 Debezium을 직접 띄워서 Kafka Connect로 실제 Change Event가 어떤 JSON 형태로 나오는지, Outbox Event Router 설정은 어떻게 하는지 실습해보고 싶다.
