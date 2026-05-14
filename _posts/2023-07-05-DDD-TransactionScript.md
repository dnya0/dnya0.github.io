---
title: 비즈니스 로직 처리
author: dnya0
date:   2023-07-06 00:13:00 +0900
categories: [Backend, Spring] 
tag: [Study, Spring, Web]
math: true
mermaid: true
image:
  path: https://github.com/dnya0/dnya0.github.io/assets/84761609/f77182a2-6283-4315-b6bc-b92485a15882
  alt: Spring Web layer
---

> [스프링 부트와 AWS로 혼자 구현하는 웹 서비스](https://www.yes24.com/Product/Goods/83849117)를 읽고 작성한 글입니다.
{: .prompt-info }

여기서 많은 분들이 오해하고 계신 것이, Service에서 비지니스 로직을 처리해야 한다는 것입니다. 하지만, 전혀 그렇지 않습니다. Service는 트랜잭션, 도메인 간 순서 보장의 역할만 합니다.

“그럼 비지니스 로직은 누가 처리하냐?”라고 반문할 수 있습니다. 잠깐, 다음 그림을 보겠습니다.

![image](https://github.com/dnya0/dnya0.github.io/assets/84761609/f77182a2-6283-4315-b6bc-b92485a15882)

간단하게 각 영역을 소개하자면 다음과 같습니다. 이미 아시는 분들은 넘어가셔도 좋습니다.

- Web Layer
    - 흔히 사용하는 컨트롤러(`@Controller`)와 JSP/Freemarker 등의 뷰 템플릿 영역입니다.
    - 이외에도 필터(@Filter), 인터셉터, 컨트롤러 어드바이스(`@ControllerAdvice`) 등 외부 요청과 응답에 대한 전반적인 영역을 이야기합니다.

- Service Layer
    - `@Service`에 사용되는 서비스 영역입니다.
    - 일반적으로 Controller와 Dao의 중간 영역에서 사용됩니다.
    - `@Transactional`이 사용되어야 하는 영역이기도 합니다

- Repository Layer
    - Database와 같이 데이터 저장소에 접근하는 영역입니다.
    - 기존에 개발하셨던 분들이라면 Dao(Data Access Object) 영역으로 이해하시면 쉬울 것입니다.

- Dtos
    - Dto(Data Transfer Object)는 계층 간에 데이터 교환을 위한 객체를 이야기하며 Dtos는 이들의 영역을 얘기합니다.
    - 예를 들어 뷰 템플릿 엔진에서 사용될 객체나 Repository Layer에서 결과로 넘겨준 객체 등이 이들을 이야기합니다.

- Domain Model
    - 도메인이라 불리는 개발 대상을 모든 사람이 동일한 관점에서 이해할 수 있고 공유할 수 있도록 단순화시킨 것을 도메인 모델이라고 합니다.
    - 이를테면 택시 앱이라고 하면 배차, 탑승, 요금 등이 모두 도메인이 될 수 있습니다.
    - `@Entity`를 사용해보신 분들은 `@Entity`가 사용된 영역 역시 도메인 모델이라고 이해해주시면 됩니다.
    - 다만, 무조건 데이터베이스의 테이블과 관계가 있어야만 하는 것은 아닙니다
    - VO처럼 값 객체들도 이 영역에 해당하기 때문입니다

Web<sup>Controller</sup>, Service, Repository, Dto, Domain, 이 5가지 레이어에서 비지니스 처리를 담당해야 할 곳은 어디일까요? 

바로 **Domain** 입니다.

기존에 서비스로 처리하던 방식을 트랜잭션 스크립트라고 합니다. 주문 취소 로직을 작성한다면 다음과 같습니다.

<br>

## 수도 코드

```java
@Transactional
public Order cancelOrder(int orderId) {
    
    1) 데이터베이스로부터 주문정보 (Orders), 결제정보(Billing), 배송정보(Delivery) 조회
    
    2) 배송 취소를 해야 하는지 확인
    
    3) if(배송 중이라면) {
        배송 취소로 변경
    }
    
    4) 각 테이블에 취소 상태 Update
}
```

<br>

## 실제 코드

```java
@Transactional
public Order cancelOrder(int orderId) {

    //1)
    OrdersDto order = ordersDao.selectOrders(orderId);
    BillingDto billing = billingDao.selectBilling(orderId);
    DeliveryDto delivery = deliveryDao.selectDelivery(orderId);

    //2)
    String deliveryStatus = delivery.getStatus();
    
    //3)
    if("IN_PROGRESS".equals(deliveryStatus)) {
        delivery.setStatus("CANCEL");
        deliveryDao.update(delivery);
    }

    //4)
    order.setStatus("CANCEL");
    ordersDao.update(order);

    billing.setStatus("CANCEL");
    deliveryDao.update(billing);
    
    return order;
}
```

모든 로직이 **서비스 클래스 내부**에서 처리됩니다. 그러다 보니 **서비스 계층이 무의미하며, 객체란 단순히 데이터 덩어리** 역할만 하게 됩니다. 반면 도메인 모델에서 처리할 경우 다음과 같은 코드가 될 수 있습니다.

```java
@Transactional
public Order cancelOrder(int orderId) {

    //1)
    Orders order = ordersRepository.findById(orderId);
    Billing billing = billingRepository.findByOrderId(orderId);
    Delivery delivery = deliveryRepository.findByOrderId(orderId);

    //2-3)
    delivery.cancel();

    //4)
    order.cancel();
    billing.cancel();

    return order;
}
```

order, billing, delivery가 각자 본인의 취소 이벤트 처리를 하며, 서비스 메소드는 트랜잭션과 도메인 간의 순서만 보장해 줍니다. 이 책에서는 계속 이렇게 도메인 모델을 다루고 코드를 작성합니다

<hr>

해당 내용을 읽고 생각이 많아졌다. 그동안 서비스 계층에 비즈니스 로직을 넣는줄 알았는데! 

글을 여러 번 읽고 해당 내용에 대해 찾아보니 *서비스 계층*에 비즈니스 로직을 넣는 것은 **트랜잭션 스크립트**, *도메인(엔티티)*에 비즈니스 로직을 넣는 것은 **도메인 주도 설계**라는 것이다.

<br>

## 📌 트랜잭션 스크립트 패턴 vs 도메인 모델 패턴

### 📎 트랜잭션 스크립트 패턴

- 하나의 트랜잭션으로 구성된 로직을 **단일 함수 또는 단일 스크립트**에서 처리하는 구조
- Entity에는 비즈니스 로직이 거의 없고, **Service Layer에서 대부분의 비즈니스 로직을 처리**하는 구조
- 여러 도메인간의 경계가 되는 부분을 처리할 때 주로 사용
- 절차 지향 설계, 엔티티를 자료 구조로 사용함.

<br>

### 📎 도메인 모델 패턴

- **Entity에서 비즈니스 로직**을 가지고 **객체지향의 특성을 적극 활용**하는 것
- 대부분의 **비즈니스 로직이 Entity 안에 구성**되어 있다. **Service Layer는 Entity에 필요한 역할을 위임**하는 역할을 함.
- 객체 지향 설계, 엔티티를 객체로 사용함.


<br>

### 📎 장단점

> 서비스패턴 or 트랜잭션 스크립트 패턴 : 단순 CRUD 작업 <br> 도메인 모델 : 비즈니스 로직이 복잡할 때 적합

|  구분  | 트랜잭션 스크립트 | 도메인 모델 |
| :----: | ------------- | -------- |
| 장점 | ∙ 구현 방법이 단순함 <br> ∙ 얼마나 모듈화를 잘하느냐에 따라 높은 효율을 낼 수 있음. | 객체 지향에 기반한 재사용성, 확장성, 유지 보수의 편리함 |
| 단점 | ∙ 비즈니스 로직이 복잡해질 수록 난잡한 코드를 만들게 됨 <br> ∙ 도메인에 대한 분석 / 설계 개념이 약하기 때문에 코드의 중복 발생을 막기 어려움 | ∙ 하나의 도메인 모델을 구축하는 데 많은 노력이 필요함 <br> ∙ 객체를 판별하고 객체들 간의 관계를 정립해야 하며, 데이터베이스 사이의 매핑에 대해 고민해야 함 |

<br>

<hr>

해당 내용을 더 자세히 알기 위해 [도메인 주도 개발 시작하기: DDD 핵심 개념 정리부터 구현까지](https://www.hanbit.co.kr/store/books/look.php?p_code=B4309942517) 라는 책을 구매했다. 원래 DDD Start! 라는 책을 구매하려 했으나 저 책이 DDD Start!의 리메이킹 버전이라는 글, DDD Start!가 절판된 점을 고려하여 저 책을 구매하게 되었다. 개발은 정말 공부하면 할수록 끝이 없는 것 같다.

<br>

## 참고

- [스프링 부트와 AWS로 혼자 구현하는 웹 서비스](https://www.yes24.com/Product/Goods/83849117)
- [비지니스 로직은 어디에 작성해야하나요?](https://okky.kr/questions/1402613)
- [인프런 김영한 님의 답변 1](https://www.inflearn.com/questions/250279/service%EC%99%80-entity-%EB%B9%84%EC%A6%88%EB%8B%88%EC%8A%A4-%EB%A1%9C%EC%A7%81%EC%97%90-%EA%B4%80%ED%95%B4)
- [인프런 김영한 님의 답변 2](https://www.inflearn.com/questions/117315/%EB%B9%84%EC%A7%80%EB%8B%88%EC%8A%A4-%EB%A1%9C%EC%A7%81%EA%B5%AC%ED%98%84-entity-vs-service)
- [비즈니스 로직을 처리하는 패턴](https://da-y-0522.tistory.com/74#recentComments)
- [9. 스프링부트와 AWS - 트랜잭션 스크립트, 도메인 모델](https://sgcomputer.tistory.com/280)
