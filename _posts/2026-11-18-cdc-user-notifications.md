---
title: DB 변경을 감지해서 사용자에게 알림을 보낼 수 있을까? — CDC로 알림 설계하기
author: dnya0
date: 2026-11-18 09:00:00 +0900
categories: [Backend, CS]
tag: [Study, CDC, PostgreSQL, Debezium, Kafka, Outbox, Notification]
math: false
mermaid: true
---

## 🥑 들어가며

프론트에서 저장 버튼을 누르면 API 서버가 요청을 받고 DB에 데이터를 저장한다. DB는 그 변경을 로그로도 기록한다. 그렇다면 다른 서버가 그 로그를 지켜보다가, 변경이 생기면 사용자에게 알림을 보낼 수도 있지 않을까?

**가능하다. CDC로 감지한 변경을 알림 처리의 출발점으로 쓰면 된다.**

이전에 쓴 [PostgreSQL CDC 파이프라인 톺아보기]({% post_url 2026-09-17-postgresql-cdc %})에서는 WAL, Logical Decoding, Replication Slot과 데이터 전달 구조를 정리했다. 이번에는 그 뒤에 붙는 **사용자 알림**에 집중해보려 한다. 기존 글에서 소개한 Outbox와 중복 처리도, 실제 알림이 빠지거나 두 번 오는 상황에 대입해본다.

아래는 배송 상태 변경을 예로 든 설계 글이다. 운영 환경에서 구현·검증한 결과가 아니라, 각 단계에서 어떤 보장이 필요한지 살펴보기 위한 예시다.

<br>

## 1. 로그를 지켜본다는 건 어떤 의미일까

여기서 로그는 API 서버의 `application.log`나 SQL 실행 내역을 출력한 텍스트 파일이 아니다. PostgreSQL에서는 WAL을 논리적인 변경으로 해석해서 외부 시스템에 전달한다. 다른 서버가 DB 디렉터리를 열고 `tail -f`를 실행하는 모습보다는, **DB가 제공하는 복제 연결로 변경 스트림을 받는 모습**에 가깝다. [PostgreSQL Logical Decoding 문서](https://www.postgresql.org/docs/18/logicaldecoding-explanation.html)

예를 들어 Debezium과 Kafka를 사용하면 다음과 같이 구성할 수 있다.

```mermaid
flowchart TD
    Front[프론트엔드] --> API[API 서버]
    API --> DB[(PostgreSQL)]
    DB --> CDC[Logical Decoding / Debezium]
    CDC --> Kafka[Kafka]
    Kafka --> Consumer[알림 Consumer]
    Consumer --> Store[(알림 DB / 발송 작업)]
    Store --> Worker[발송 Worker]
    Worker --> User[사용자: 앱 알림 / 푸시 / 메일]
```

알림 처리는 **커밋이 확정된 변경**을 기준으로 시작해야 한다. WAL 기록이 생겼다는 사실만으로 발송하면 안 된다. 트랜잭션이 롤백되면 사용자에게 알릴 변경도 성립하지 않기 때문이다.

또한 API의 저장 성공 응답과 알림 도착은 별개다. 저장 요청이 성공한 뒤에도 Consumer나 발송 Worker에 대기 시간이 있을 수 있다. 이 구조에서 저장 성공은 알림 도착 완료를 뜻하지 않는다.

<br>

## 2. Row 변경을 곧바로 알림으로 바꾸면 생기는 일

주문 테이블에 다음 변경이 발생했다고 해보자.

```sql
UPDATE orders
SET status = 'SHIPPED'
WHERE id = 42;
```

사용자에게는 “주문한 상품의 배송이 시작됐어요”라고 알려주고 싶다. 그런데 주문 테이블에는 배송 시작 외에도 여러 변경이 발생한다.

| 변경 내용 | 이 예시에서의 배송 시작 알림 |
| --- | --- |
| `PAID → SHIPPED` | 발송 |
| 배송 중인 주문의 내부 메모 수정 | 발송하지 않음 |
| 이미 `SHIPPED`인 주문에 같은 상태 재저장 | 발송하지 않음 |
| 운영자가 과거 데이터를 일괄 보정 | 별도 정책으로 판단 |

따라서 `orders`에 UPDATE가 생겼는지, 변경 후 상태가 `SHIPPED`인지만 확인해서는 부족하다. **어떤 상태 전이가 일어났는지**를 판단해야 한다.

아래는 변경 전후 값이 제공된다고 가정한 의사 코드다.

```python
def is_shipping_started(event):
    if event.operation != "UPDATE":
        return False
    return (
        event.before["status"] == "PAID"
        and event.after["status"] == "SHIPPED"
    )
```

이때 `before`에 모든 컬럼이 들어온다고 가정하면 안 된다. PostgreSQL의 이전 값 제공 범위는 `REPLICA IDENTITY` 설정 등에 영향을 받는다. 이전 상태가 필요한 설계라면 해당 값이 실제 이벤트에 포함되는지 확인해야 한다. `FULL` 설정도 로그량 증가를 함께 고려해야 한다. [Debezium PostgreSQL Connector 문서](https://debezium.io/documentation/reference/stable/connectors/postgresql.html)

이전 값이 없다고 그 시점에 주문 테이블을 다시 조회하는 것도 같은 결과를 보장하지 않는다. Consumer가 읽을 때는 주문이 이미 `DELIVERED`로 바뀌었을 수 있다. 현재 Row는 과거 변경의 전후 상태를 복원해주지 않는다.

<br>

## 3. 알림의 의미를 명시하고 싶다면 Outbox를 붙인다

Row 변경만 보고 사용자의 행동이나 업무 의도를 알아내기는 어렵다. 같은 `PAID → SHIPPED`라도 실제 배송 시작인지, 과거 데이터 보정인지 구분할 정보가 없을 수 있다.

애플리케이션을 수정할 수 있다면, 주문 변경과 함께 **`OrderShipped` 이벤트를 Outbox에 기록하는 방식**을 선택할 수 있다.

```txt
하나의 DB 트랜잭션
  ├─ orders: 배송 상태 변경
  └─ outbox: OrderShipped 이벤트 INSERT
              ↓ COMMIT
        CDC로 outbox 변경 전달
              ↓
        알림 Consumer
```

이렇게 하면 DB 저장 후 메시지를 직접 발행하다가 프로세스가 종료되는 빈틈을 줄일 수 있다. 주문 변경과 이벤트 기록이 함께 커밋되므로, 발행할 이벤트가 DB에 남는다. 이후 전달에는 별도의 재시도와 복구가 필요하다. [AWS Transactional Outbox 설명](https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/transactional-outbox.html)

예를 들어 애플리케이션이 정의하는 이벤트는 다음과 같다. Debezium의 원본 메시지 형식이나 기본 Outbox 테이블 스키마를 그대로 나타낸 예시는 아니다.

```json
{
  "eventId": "order-42-shipped-v7",
  "eventType": "OrderShipped",
  "aggregateId": "42",
  "aggregateVersion": 7,
  "recipientId": "user-100",
  "occurredAt": "2026-11-18T10:00:00+09:00",
  "schemaVersion": 1
}
```

`eventId`는 같은 이벤트가 재전달돼도 유지하고, 새로운 사건에는 새로운 값을 부여한다. `aggregateVersion`은 주문 안에서의 변경 순서를 판단하는 용도다. API 요청 자체가 재시도될 때도 실제 상태 전이가 성공한 경우에만 이벤트를 기록하도록 해야 한다.

Debezium의 Outbox Event Router는 Outbox 레코드를 메시지로 변환하는 기능을 제공한다. 이벤트 ID를 중복 제거에 활용하고, 주문 ID 같은 aggregate ID를 메시지 키로 사용할 수 있다. [Outbox Event Router 문서](https://debezium.io/documentation/reference/stable/transformations/outbox-event-router.html)

Outbox와 CDC는 함께 쓸 수 있다. **Outbox는 전달할 사건의 의미를 정하고, CDC는 저장된 이벤트를 외부로 옮기는 역할**을 맡는다.

<br>

## 4. 같은 이벤트를 두 번 받아도 알림은 하나만 만들기

Consumer가 알림을 저장한 직후, Kafka Offset을 커밋하기 전에 종료됐다고 해보자.

```txt
이벤트 수신 → 알림 저장 → 프로세스 종료
                              ↓
재시작 → 같은 이벤트 재수신 → 알림을 또 저장할까?
```

이 때문에 이벤트마다 무조건 알림을 INSERT하면 중복 알림이 만들어진다. 알림 DB에 다음과 같은 고유 제약을 두는 방법을 생각할 수 있다.

```sql
UNIQUE (event_id, recipient_id, channel)
```

같은 사건을 같은 사용자에게 같은 채널로 보내는 작업은 한 번만 만든다는 뜻이다. 하나의 사건에서 같은 채널로 여러 알림을 만들어야 한다면 알림 종류 등도 키에 포함해야 한다.

Consumer의 처리 순서는 다음과 같이 설계한다.

1. 이벤트를 받아 수신 대상과 알림 정책을 판단한다.
2. 알림 DB 트랜잭션에서 사용자 알림과 발송 작업을 함께 저장한다. 중복 키라면 기존 작업을 유지한다.
3. DB 커밋이 성공한 뒤 해당 메시지까지의 Kafka Offset을 커밋한다.
4. Worker가 저장된 발송 작업을 읽어 외부 채널로 보낸다.

DB 커밋 전에 실패하면 다시 처리하고, DB 커밋 후 Offset 커밋 전에 실패하면 고유 제약으로 중복 생성을 막는다. 처리 완료 여부를 별도 테이블에 기록한다면 그것도 알림·발송 작업과 **같은 트랜잭션**에 묶어야 한다.

파티션 내 메시지를 병렬 처리할 때는 앞선 메시지가 아직 저장되지 않았는데 뒤쪽 Offset부터 커밋하지 않도록 주의한다. 재시작 때 미처리 메시지를 건너뛸 수 있기 때문이다.

<br>

## 5. 알림 DB의 중복 제거와 실제 발송은 다른 문제다

알림 Row를 하나만 만들었다고 해서 푸시도 반드시 한 번만 도착하는 것은 아니다.

```txt
Worker → 외부 발송 서비스: 푸시 요청
외부 발송 서비스: 요청 접수 성공
Worker: 응답을 받기 전에 타임아웃
```

이때 Worker는 발송 서비스가 요청을 처리했는지 확신할 수 없다. 재시도하면 중복될 수 있고, 재시도하지 않으면 누락될 수 있다. 로컬 DB 트랜잭션만으로 외부 서비스 호출까지 원자적으로 묶을 수는 없다.

이 예시에서는 다음처럼 발송 정책을 정할 수 있다.

- 제공자가 멱등성 키를 지원하면 같은 발송 작업 ID를 재시도에도 사용한다. 보장 기간과 범위도 확인한다.
- 요청 ID로 처리 결과를 조회할 수 있다면, 결과가 불명확한 요청을 먼저 조회한다.
- 여러 Worker가 같은 작업을 동시에 가져가지 않도록 잠금이나 만료 시간이 있는 점유 처리를 둔다.
- 일시적 실패는 간격을 늘리며 재시도하고, 영구 실패나 재시도 한도 초과는 별도 보관해 조사한다.

발송 성공의 정의도 필요하다. 외부 서비스의 접수 성공, 사용자 기기 도착, 사용자의 읽음은 각각 다른 상태다. 알림 목록은 DB 조회로 복구할 수 있게 하고, 푸시나 SSE/WebSocket은 새 알림을 빠르게 알려주는 수단으로 두면 연결이 끊겼던 사용자도 다시 목록을 확인할 수 있다.

<br>

## 6. 알림에서 특히 조심해야 하는 초기 적재와 재처리

### 과거 주문을 오늘의 배송 알림으로 보내지 않기

CDC를 처음 연결하면 기존 데이터의 Snapshot을 읽는 구성이 있을 수 있다. Debezium의 Snapshot 읽기 이벤트는 일반 INSERT와 구분되는 `op = r`로 표현된다. 이를 새 배송 사건으로 해석하면 과거 주문에 대한 알림을 한꺼번에 보낼 수 있다. [Debezium Snapshot 및 이벤트 형식](https://debezium.io/documentation/reference/stable/connectors/postgresql.html)

이 글의 주문 테이블 직접 감지 예시에서는 Snapshot을 상태 초기화에만 사용하고 발송 대상으로 삼지 않는다. 다만 **Outbox Snapshot에는 아직 전달되지 않은 실제 이벤트가 있을 수 있다.** 모든 Snapshot을 무조건 버리는 방식은 위험하다. 도입 시점, 미발송 범위, 기존 처리 기록을 바탕으로 별도의 전환 정책을 정해야 한다.

### 순서가 바뀌거나 오래 지연된 알림

주문별 순서가 필요하다면 Kafka 메시지 키를 주문 ID로 두고 같은 파티션에서 처리하도록 설계할 수 있다. 이후 Worker 병렬 처리나 별도 재시도 경로에서도 순서가 유지되는지는 따로 확인해야 한다.

이미 배송 완료 알림이 나간 뒤 지연된 배송 시작 이벤트가 도착했다면 어떻게 할까? 이 예시에서는 주문 버전과 이벤트 발생 시각을 확인해 오래된 배송 시작 알림을 생략할 수 있다. 반면 결제 영수증처럼 늦어도 전달해야 하는 알림에는 같은 규칙을 적용하면 안 된다.

또한 운영자가 과거 이벤트를 재생할 때 알림까지 다시 보낼 것인지 정해야 한다. 상태 복원용 재처리와 실제 발송 경로를 분리하고, 중복 제거 기록의 보존 기간도 허용할 재처리 기간에 맞추는 편이 안전하다.

<br>

## 7. 어디까지 동작하는지 확인하려면

이 구조를 구현한다면 정상적인 배송 상태 변경뿐 아니라 다음 상황도 검증하고 싶다.

| 상황 | 확인할 결과 |
| --- | --- |
| 주문 변경 트랜잭션 롤백 | 알림이 생성되지 않음 |
| 같은 이벤트를 두 번 전달 | 동일한 수신자·채널의 발송 작업은 하나 |
| 알림 저장 직후 Consumer 종료 | 재수신해도 작업이 중복 생성되지 않음 |
| 발송 서비스 접수 후 응답 유실 | 불확실한 결과에 대한 조회·재시도 정책 적용 |
| 초기 Snapshot 또는 과거 이벤트 재생 | 정한 전환·재처리 정책대로만 발송 |
| 배송 완료 이후 배송 시작 이벤트 처리 | 알림 종류에 맞는 지연·순서 정책 적용 |

운영 지표도 단계별로 나누어야 한다. CDC 지연, Kafka Consumer Lag, 발송 대기 작업의 나이, 채널별 실패율은 서로 다른 병목을 보여준다. Kafka까지 전달됐어도 발송 Worker가 멈췄다면 사용자에게는 알림이 오지 않는다.

Replication Slot의 WAL 보존량도 함께 살펴야 한다. 다만 **Kafka를 읽는 알림 Consumer의 중단과 DB에서 읽는 CDC Connector의 중단은 다른 장애**다. 전자는 주로 Kafka 쪽 지연·보존 기간을, 후자는 DB 쪽 WAL 누적을 확인하게 된다. 두 구간의 장애가 연쇄적으로 번질 수도 있으므로 각각의 진행 상태를 관찰해야 한다.

<br>

## 마치며

“DB 로그를 지켜보다가 사용자에게 알림을 보낼 수 있지 않을까?”라는 생각에서 시작했는데, 알림까지 이어보니 결정할 것이 꽤 많다.

CDC로 변경을 감지한 다음에는 알림을 보낼 사건인지 판단하고, 수신자를 정하고, 작업을 안전하게 저장해야 한다. 외부 발송의 실패와 중복, 오래된 이벤트의 처리 정책도 필요하다.

이미 CDC를 사용하고 있거나 여러 경로에서 발생한 DB 변경을 포착해야 한다면 이 구조를 검토할 수 있다. 반면 알림만을 위해 처음부터 CDC와 Kafka를 운영하기 부담스럽다면, 같은 트랜잭션에 Outbox를 기록하고 Worker가 주기적으로 읽는 구조도 선택지다. 이때는 현재 주문 상태를 Polling하는 것과 달리 **전달할 사건 자체가 저장돼 있다.**

이번에 더 생각해보게 된 부분은 변경을 발견하는 순간보다 그 이후다. 사용자가 받아야 할 알림을 정의하고, 장애가 나도 그 의도가 유지되도록 만드는 것까지가 알림 시스템의 설계다.
