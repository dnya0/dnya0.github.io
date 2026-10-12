---
title: PostgreSQL AT TIME ZONE과 월 단위 연산의 함정
author: dnya0
date: 2026-10-02 18:00:00 +0900
categories: [TIL, PostgreSQL]
tag: [PostgreSQL, Timezone, Database]
math: false
mermaid: false
---

## 🥑 들어가며

회사 업무 중에 [이 글](https://news.hada.io/topic?id=34434)을 보고 "어 이거 우리도 당할 수 있겠는데?" 싶어서 코드를 뒤져봤다. 결론부터 말하면 지금 담당하고 있는 서비스들엔 해당 패턴이 없어서 당장 문제는 없었지만, 나중에 다른 프로젝트에서 raw SQL로 날짜 연산을 직접 짤 일이 생기면 바로 걸릴 수 있는 함정이라 TIL로 남겨둔다.

<br>

## 📌 문제: `AT TIME ZONE`은 "변환"이 아니라 "타입 변경"

`timestamptz`(타임존 있는 타임스탬프)에 `AT TIME ZONE 'UTC'`를 걸면, 단순히 타임존만 UTC로 바뀌는 게 아니라 **타입 자체가 `timestamp`(타임존 없는 타입)로 바뀐다.**

```sql
-- created_at이 timestamptz라고 가정
SELECT created_at AT TIME ZONE 'UTC';
-- 결과 타입: timestamp (timestamptz 아님!)
```

문제는 여기서 끝나지 않는다. PostgreSQL에서 **월 단위 연산(`+ interval '1 month'`)은 현재 세션의 타임존 설정에 영향을 받는다.** `timestamptz`를 유지한 채로 월을 더하면 PostgreSQL이 내부적으로 UTC 기준으로 계산해주지만, 위처럼 `AT TIME ZONE`을 걸어 `timestamp`로 바뀐 값에 월을 더하면 **세션 타임존 기준으로 날짜가 계산**된다.

예시: UTC로 3월 1일 00:00인 시각이 Pacific Time(UTC-8)으로는 2월 28일 16:00이다. 이 상태에서 한 달을 더하면, UTC 기준으로 "3월 1일 + 1개월 = 4월 1일"을 기대했는데 실제로는 "2월 28일 + 1개월 = 3월 28일" 기준으로 계산되어버릴 수 있다.

<br>

## 🚨 우회법

`timestamptz` 타입을 유지하면서 올바르게 비교·저장하려면, interval 연산 앞뒤로 `AT TIME ZONE 'UTC'`를 **두 번** 걸어야 한다.

```sql
-- 1) timestamptz -> timestamp (UTC 기준으로 "벗기기")
-- 2) 월 연산
-- 3) timestamp -> timestamptz (다시 UTC로 "씌우기")
SELECT (created_at AT TIME ZONE 'UTC' + interval '1 month') AT TIME ZONE 'UTC';
```

두 번째 `AT TIME ZONE 'UTC'`가 `timestamp`를 다시 `timestamptz`로 되돌려주는 역할을 한다.

<br>

## 📎 참고할 점

- PostgreSQL 16 이상부터는 `date_add()` 함수에 타임존을 명시적으로 넘길 수 있어서, 이 이중 변환 없이도 안전하게 월 단위 연산이 가능하다.
- 원문에 달린 HN 댓글들을 보면, 원문이 PostgreSQL wiki의 가이드를 과도하게 단순화했다는 지적도 있었다. 또한 애초에 타임존이 걸린 날짜 계산은 DB가 아니라 애플리케이션 코드에서 처리하라는 의견이 많았다 — 서비스 로직의 "한 달 뒤" 같은 개념은 비즈니스 규칙에 가깝지, DB 레벨에서 다룰 일이 아니라는 입장이다.

<br>

## ✏️ 정리

지금 담당하는 서비스들(Spring/JPA 기반 Orchestrator, Go 기반 Auth 서비스)은 `AT TIME ZONE`을 아예 쓰지 않는다. ORM이 `Instant`/`OffsetDateTime` 같은 타입으로 타임존을 투명하게 처리해주고 있어서 이 함정 자체를 피해가고 있었던 것 — 의도한 설계였다기보단 결과적으로 안전했던 셈이다.

다음에 raw SQL로 날짜 interval 연산을, 특히 월 단위로 직접 짜야 하는 상황이 오면:

1. 가능하면 애플리케이션 코드에서 타임존을 다루고 DB는 순수 저장소로만 쓴다.
2. 꼭 DB에서 해야 한다면 `AT TIME ZONE` + interval 조합을 쓰기 전에 PostgreSQL 버전부터 확인하고, 16+ 면 `date_add()`를 쓴다.
3. 16 미만이면 위 "두 번 걸기" 패턴을 기억해둔다.
