---
title: Transactional을 걸어야하는 경우와 Lock
author: dnya0
date:   2024-09-17 15:47:00 +0900
categories: [Backend, Spring]
tag: [Spring, Java, Transactional, Lock]
math: true
mermaid: true
image:
  path: https://user-images.githubusercontent.com/84761609/220085479-19ee260a-d709-4a47-b788-416275d8a2d8.png
  alt: Spring Image
---

## 🥑 들어가며

최근 간단한 프로젝트를 시작했다. 친구와 함께 시작했던 팀프로젝트의 기간이 늘어지게 되면서 시작하게 되었다. 서버는 개발할게 아직 많이 남아있지만 TS와 Nest 공부겸 다른 팀프로젝트를 시작하기도 했고, 각자 부트캠프와 회사 일 때문에 사실상 개발기간이 1년 이상 늘어났다고 생각한다. 늘어난 기간과 함께 TS와 Nest가 어렵기도 하고 꽤나 공부할게 많아서 힘들 때마다 정신을 환기시킬 겸 간단한 개인 프로젝트를 시작하게 된 것이다.

그런데, 프로젝트 개발 중 습관적으로 `@Transactional`과 DB Lock을 걸던 중 갑자기 의문이 들었다. Service 계층에 이 함수는 그저 데이터를 저장하고 불러오는 것 밖에 없는데 `@Transactinal`을 거는게 맞을까? 내가 지금 정의하고 있는 메소드는 각각 저장과 불러오는 것 밖에 하지 않는다.

<br>

## 🤔 Transactional이란?

우선 **Transaction 트랜잭션**이란 **DB에서 여러 개의 데이터 읽기와 쓰기 동작을 하나의 동작처럼 논리적으로 묶는 것**이다. Spring의 `@Transactinal`은 이 DB의 트랜잭션과 같이 여러 동작을 하나로 묶어준다.

트랜잭션의 예시로 항상 나오는 것은 입출금이 나온다.

- A의 계좌에서 B의 계좌로 보낼 때
    - A의 계좌에서 잔고 확인
    - B의 계좌가 돈을 받을 수 있는지 확인
    - A 계좌에서 출금
    - B의 계좌에 입금

만약 A 계좌에서 출금된 상태로 멈춘다면 어떻게 될까? A의 계좌에선 돈이 빠져나간 상태지만 B는 돈을 받지 못한 상태가 된다. A만 억울하게 돈을 잃게 되는 것!

이와 같은 상황이 일어나는 것을 방지하기 위해 **모든 작업이 완료되었을 때 DB에 저장하고, 문제가 생길 경우 원상태로 복귀시킨다.**

<br>

## ✔️ 코드 살펴보기

그렇다면 평소에 내가 짰던 코드를 살펴보자.

```java
@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository;

    @Transactional
    public void saveUser(@NotNull UserSaveRequest request) {
        String filePath = FilePath.createFilePath(request.imageFile());

        User user = User.from(request, filePath);
        userRepository.save(user);

        if (!userRepository.existsById(user.getId())) {
            throw new CBNotSavedException("User not saved");
        }
    }

    @Transactional
    public User updateUser(@NotNull UserUpdateRequest request) {
        User user = userRepository.findByEmail(request.email())
            .orElseThrow(() -> new CBNotFoundException("cannot find user"));

        user.update(request);

        return UserDataResponse.of(user);
    }

    public User getUser(String userId) {
        return userRepository.findById(userId)
                .orElseThrow(() -> new CBNotFoundException("cannot find user"));
    }

}
```

> +) `@Transactinal`을 사용 시 Update를 해줄 때 `save()`를 호출하지 않아도 된다. **Dirty Checking 변경감지** 때문! <br> <br> **변경감지**는 트랜잭션 커밋 시 영속화된 Entity에서 가지고 있었던 최초 정보(스냅샷)와 바뀐 Entity 정보를 비교해서 바뀐 부분을 update 해주는 기능이다.

이 로직에서 하나의 과정 중 에러가 발생해멈췄을 때 문제가 생길만한 부분이 있을까?

아까 예시로 들었던 입출금과 달리 로직이 중간에 멈춰도 커다란 문제가 되지 않는다. 그렇다면 `@Transactional`을 전체 로직에 걸 필요가 없지 않은가. **DB에 어떤 값도 들어가지 않았기 때문에 정합성을 보장할 이유가 없다.** 또한 Spring data JPA에서 데이터를 읽고 쓰는 일련의 모든 데이터 연산에는 트랜잭션이 기본적으로 적용된다.

사실 이런 질문을 할 수도 있다.

> *그래도 하나로 묶는게 낫지 않을까요? 로직이 한번에 작동하는게 낫지 않을까요?*

그렇지 않다. **트랜잭션은 길 수록 오버헤드가 발생하기 때문!**

<br>

## 🤔 트랜잭션은 길 수록 오버헤드가 발생한다?

우선 이미지를 보자

![image](https://github.com/user-attachments/assets/34e2c42b-0032-47b5-b576-d87db3aad12b)

사용자의 요청이 들어올 때마다 EntityManagerFactory에서는 각 요청 당 EntityManager를 생성해 발급한다. EntityManager가 영속성 컨텍스트를 관리하는 것이기 때문이다. 이때 각 EntityManager는 DB에 접근할 때 커넥션 풀을 점유해야 읽고 쓰는 작업을 진행할 수 있다.

> 📌 **영속성 컨텍스트란?** <br> <br>영속성 컨텍스트란 **엔티티를 영구저장하는 환경**이라는 것이다. 애플리케이션과 데이터베이스 사이에서 객체를 보관하는 가상의 데이터베이스 같은 역할을 한다. 엔티티 매니저를 통해 엔티티를 저장하거나 조회하면 엔티티 매니저는 영속성 컨텍스트에 엔티티를 보관하고 관리한다. <br> <br>`em.persist(member);`<br>엔티티 매니저를 사용해 회원 엔티티를 영속성 컨텍스트에 저장한다는 의미! <br> <br>`@Transactional`이 적용된 메소드에선 자동으로 트랜잭션이 적용되고, 그 안에서만 영속성 컨텍스트가 활성화된다. 영속성 컨텍스트는 트랜잭션이 시작된 동안에만 엔티티를 관리하며, 트랜잭션이 끝나면 영속성 컨텍스트도 종료된다.  <br> <br>자세한 것은 나중에 글로 다뤄보겠다.

커넥션 풀은 DB와 미리 연결해놓은 객체를 담아놓은 풀이다. 스레드 풀과 같다고 생각하면 된다. 미리 정의해놓고 재사용하는 것이 호출할 때마다 생성하는 것과 비교했을 때 비용 부담이 상당히 줄어든다.

이제 커넥션 풀에 대해 알았으니 위의 영속성 컨텍스트 설명 글에서 `@Transactional` 파트를 다시 읽어보자. 여러 작업을 트랜잭션으로 묶었다고 가정했을 때, 트랜잭션이 시작하면 영속성 컨텍스트가 활성화되며 트랜잭션이 끝날 때까지 지속된다. 이 영속성 컨텍스트가 활성화되면서 그 기간동안 하나의 DB 커넥션을 붙잡고 있어야 한다.

이 이야기는 무엇인가? **굳이 DB 커넥션을 붙잡고 있을 필요가 없는 작업인데도 붙잡고 있는 것은 큰 자원낭비가 되며 이는 곧 오버헤드가 된다**는 뜻이다. 따라서 **트랜잭션의 범위를 어디까지 허용해야 하는가**를 고민하며 개발하는 것이 좋다. 정합성이 필요한 경우에는 당연히 서비스단에 트랜잭션을 걸어야하겠지만, 그렇지 않은 경우 레포지토리 내로 한정하는 것이 좋다.

<br>

## ✔️ 코드 수정하기

``` java
@Service
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository;

    public void saveUser(@NotNull UserSaveRequest request) {
        String filePath = FilePath.createFilePath(request.imageFile());

        User user = User.from(request, filePath);
        userRepository.save(user);

        if (!userRepository.existsById(user.getId())) {
            throw new CBNotSavedException("User not saved");
        }
    }

    public User updateUser(@NotNull UserUpdateRequest request) {
        User user = userRepository.findByEmail(request.email())
            .orElseThrow(() -> new CBNotFoundException("cannot find user"));

        user.update(request);
        userRepository.save(user);
        
        if (!getUser(user.getId()).equals(user)) {
            throw new CBNotSavedException("User not saved");
        }

        return UserDataResponse.of(user);
    }

    public User getUser(String userId) {
        return userRepository.findById(userId)
                .orElseThrow(() -> new CBNotFoundException("cannot find user"));
    }

}
```

고치는 것은 어렵지 않다. 그저 `@Transactional`을 제거해주면 된다. Update의 경우 `@Transactional`로 관리되던 영속성 컨텍스트가 없으니 `save()`함수만 호출하면 된다. 제대로 추가되었는지 확인하는 코드도 추가하자.

<br>

## 🤔 Transaction 동시성 제어

우선 위에서 트랜잭션에 대해 적어놓은 것은 `DB에서 여러 개의 데이터 읽기와 쓰기 동작을 하나의 동작처럼 논리적으로 묶는 것`이라 적어놨다. 문득 이런 의문이 드는 것이다. *트랜잭션은 하나의 논리적 단위로 묶어 작업하는데, 굳이 DB Lock이 필요한걸까? 동시성 문제도 트랜잭션으로 해결할 수 있지 않을까?*

결론만 보자면 아니다.

우선 트랜잭션의 동시성 제어에 대해 알아보자.

### 동시성 제어

- 다중 사용자 환경에서 둘 이상의 트랜잭션이 동시에 수행될 때, 일관성을 해치지 않도록 트랜잭션의 데이터 접근 제어
- 다중 사용자 환경을 지원하는 DBMS의 경우, 반드시 지원해야 하는 기능

고립성은 상호 간의 트랜잭션을 독립적으로 만들어 준다. 그런데 2개 이상의 트랜잭션이 하나의 값에 접근하는 경우에는 어떻게 될까? 아래의 표를 확인하자.

| - | 트랜잭션1 | 트랜잭션2 | 발생 문제 | 동시 접근 |
| :---: | :---: | :---: | --- | --- |
| 상황 1 | 읽기 | 읽기 | 읽음 | 허용 |
| 상황 2 | 읽기 | 쓰기 | 오손 읽기, 반복 불가능 읽기, 유령 데이터 읽기 | 허용 또는 <br>불가 선택 |
| 상황 3 | 쓰기 | 쓰기 | 갱신손실, 모순성, 연쇄 복구 | 허용 불가<br>(Lock 사용) |

조회수를 예를 들어 설명하겠다. 아래의 이미지는 유저가 방문할 때마다 카운팅을 하는 로직이다.

![image](https://github.com/user-attachments/assets/92509896-54d3-4ccb-97f1-41d9476e52b6)

User1과 User2가 비슷한 시점에 게시물을 방문하였을 때이다. User1이 카운트를 올리지 않은 시점에서 User2가 카운트를 가져온다. User1의 트랜잭션이 끝나지 않은 시점이기에 42를 가져온다. User1은 43으로 업데이트를 하고, User2는 이전에 42를 가져왔으니 역시 43으로 업데이트를 한다. 동시성 문제가 발생한 것이다.

이 문제를 해결하기 위해 DB는 트랜잭션이 커밋했을 때의 결과가 트랜잭션이 순차적(하나씩 차례로) 실행됐을 때의 결과와 동일하도록 보장해줄 필요가 있다.

그러나 위에서 작성한 것과 같이 이를 위해 한 작업을 하는 동안 하나의 풀을 점유하고 있거나 혼자 락을 걸고 작업하는 것은 큰 자원손실을 야기시킨다. 그렇다면 어느정도 선에서 격리성을 보장하는 것이 좋을까?

<br>

## 📌 Lock

**Locking 로킹** 기법은 트랜잭션들이 동일한 데이터 항목에 대해 임의적인 병행 접근을 하지 못하도록 제어하는 것을 말한다. 종류엔 낙관적 락과 비관적 락, 비관적 락은 다시 배타락과 공유락으로 구분된다.


### 낙관적 락

동시성 문제에서 **경쟁조건이 발생하지 않을 것**이라 보고 거는 락이다. DB에서 사용하는 실제 락을 사용하는 것은 아니고, JPA가 제공하는 버전 관리 기능을 사용한다. 구현 시 예외 처리를 반드시 해주어야 한다. 예외가 발생하는 경우 예외가 발생했던 로직을 재시도해야 한다.

- 회원정보 수정
    - 그 회원 외에 누군가가 작업할 경우가 낮음
    - 동시에 수정이 이루어진 경우를 감지해서 예외를 발생시키 더라도 실제 상황에선 예외가 발생할 경우가 낮음

### 비관적 락

동일한 데이터를 동시에 수정할 가능성이 높을 때 거는 락이다.

- 상품 주문
    - 상품은 동시에 여러 명이 주문할 가능성이 높으니 재고 관리의 정합성을 보장하기 위해서라도 반드시 충돌 가능성을 염두에 두고 Lock을 걸어야  한다.
    - 이 과정에서 정합성을 보장하는 만큼 성능은 떨어질 수 밖에 없다.  

- **Shared(S) Lock 공유락**
    - 다른 트랜잭션이 잠긴 객체를 읽고 다른 공유 락을 생성하는 것은 허용하지만, 쓰기나 베타 락을 생성하는 것을 허용하지 않는다.
    - 여러 트랜잭션이 동일한 행에 공유 락을 생성할 수 있다. 즉 다른 트랜잭션이 읽고 있는 행을 읽을 수 있다.
    - 공유 락이 걸려있는 행에 베타 락을 걸 수는 없음, 즉 해당 공유 락이 해제될 때까지 쓰기 불가능.

- **Exclusive(X) Lock 베타락**
    - 동일한 행에 다른 트랜잭션을 생성하는 것을 허용하지 않는 잠금.
    - 다른 트랜잭션에서 베타 락을 생성할 수 없기 때문에 쓰기가 불가능.
    - 다른 트랜잭션에서 공유 락을 생성할 수 없음. 단 락을 쓰지 않는 읽기는 가능.

공유 락은 같은 공유 락을 허용하기 때문에 읽기가 가능하고, 쓰기에 필요한 베타 락을 불허하기 때문에 쓰기가 불가능 한 것이며, 베타 락은 락을 사용하는 읽기(Locking Reads)인 경우에는 불가능하며 락을 사용하지 않는 읽기(Consistent Nonlocking Reads)인 경우에는 가능하다.

<br>

## ✔️ 동시성 제어 해결하기

위에서 예를 들었던 조회수 동시성 문제 (갱신손실)은 어떻게 해결해야 할까. 나라면 MVP는 우선 낙관적 락으로 구현 후 Redis 분산락으로 바꿔나갈 것 같다.

해답은 어떤 답변에서 얻을 수 있었다.

> **최상용 개발자님의 답변 요약**<br><br>먼저 개발하려는 서비스의 트래픽을 예상하는 것이 좋습니다. 순간적인 트래픽이 많지 않은 경우, 낙관적 락을 이용한 버전 관리로 개발한 후, 운영하면서 충돌이 많이 발생하는 경우 비관적 락이나 Redis락을 고려할 것 같습니다.<br><br>순간적인 트래픽이 많을 경우라고 생각되는 경우 설계 단계부터 Redis를 고려하여 설계할 것 같습니다.

<br>

> 📌 **Redis 분산락** <br> <br>**Redis 레디스**는 인메모리 DB로 빠른 성능과 간단한 API를 제공한다. 기본적으로 레디스는 싱글 스레드로 동작하기 때문에, 단일 레디스 노드를 구축해 사용해도 동시성 문제가 발생하지 않는다. **분산락**은 분산 서비스 환경에서 여러 요청, 작업이 동일한 자원(공유 자원)에 접근하여 경쟁상태가 발생하지 않도록 원자성을 보장 해준다.<br><br>[레디스 분산락 구현](https://f-lab.kr/insight/redis-distributed-lock-20240704)에 대해 flab의 글을 가져왔다.

<br>

## 🤔 트랜잭션에 비관적 락 걸기

트랜잭션에 비관적 락을 걸 경우 **락의 범위와 트랜잭션의 범위는 동등해야 한다.** 서비스 단에서 `@Transactional`을 제거시켜 트랜잭션의 범위를 레포지토리로 제한해줬다. 로직엔 낙관적 락도, 비관적 락도 걸 필요가 없었기 때문! 여기에 비관적 락이 걸린 메소드를 가져다 쓰면 어떻게 될까? `TransactionRequiredException`이 발생한다.

서비스 단에서 `@Transactional` 어노테이션을 제거함으로써 트랜잭션 범위는 서비스 범위가 아닌 리포지토리 범위로 줄어들었으나, 정작 비관적 락이 걸린 레포지토리 메소드를 서비스 로직에서 사용하면 트랜잭션 범위(리포지토리)와 락의 범위(서비스-리포지토리)가 달라진다. 따라서 `TransactionRequiredException: no transaction is in progress` 오류가 발생하는 것이다.

<br>

<br>

## 🎈 참고

- [@Transactional은 만능이 아닙니다 - 1: Transaction의 개념과 트레이드오프](https://woonys.tistory.com/238)
- [@Transactional은 만능이 아닙니다 -2 트랜잭션의 격리성과 lock](https://woonys.tistory.com/240)
- [공유 락(Shared Lock)과 베타 락(Exclusive Lock)](https://velog.io/@paki1019/%EA%B3%B5%EC%9C%A0-%EB%9D%BDShared-Lock%EA%B3%BC-%EB%B2%A0%ED%83%80-%EB%9D%BDExclusive-Lock)
- [낙관적 락과 비관적 락은 갱신 손실 문제를 어떻게 해결하는가](https://velog.io/@balparang/%EB%82%99%EA%B4%80%EC%A0%81-%EB%9D%BD%EA%B3%BC-%EB%B9%84%EA%B4%80%EC%A0%81-%EB%9D%BD%EC%9D%80-Race-Condition%EC%9D%84-%EC%96%B4%EB%96%BB%EA%B2%8C-%ED%95%B4%EA%B2%B0%ED%95%98%EB%8A%94%EA%B0%80)