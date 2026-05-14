---
title: 우아한 프리코스 회고
author: dnya0
date:   2023-11-24 08:16:00 +0900
categories: [Post, WoowaCourse]
tag: [Post, WoowaCourse, Review]
---

<br>

캡스톤디자인 프로젝트를 하던 중 우아한 테크코스가 열려 신청하게 되었다. 신청 후 1달간 프리코스를 진행하였는데, 여러 요구사항과 제약사항에 맞춰 기능을 개발해야 했다. 캡스톤 팀원들도 같이 프리코스에 참여하였기에 모여서 새벽까지 프리코스 과제를 한 적도 많았다. 

사실 4주차가 끝나는 주 주말에 바로 회고글을 작성하려 했으나 4주차 과제가 끝난 다음 날 나의 소중한 맥북에 커피를 쏟아버려서.. 거의 3일간 코딩을 못했다. 프리코스가 끝난 다음 날이라 다행이었지만 그 때문에 캡스톤 개발도 올스탑이 되어버리는 바람에.. 다행히 바로 키캡 제거하고 닦아서 침수는 안됐다고 테크니션분이 말씀하셨다.. 보증기간 이내라 점검 비용이 하나도 들지 않았다. 내년 1월 말이면 보증기간이 끝나는데 이제 어떻게 해야할지 막막하다. 애플 케어 플러스를 들어둬야 했는데....

<br>

모든 과제는 JDK 17버전으로 통일해야 했으며, 입력받을 때와 랜덤값을 뽑아야할 때는 주최측에서 준비해준 `Randoms`와 `Console` api를 사용해야 했다.

<br>

## ⚾️ 1주차 - 숫자야구 게임

> [1주차 코드 살펴보기](https://github.com/dnya0/java-baseball-6/tree/dnya0)

![32개의 커밋](https://github.com/dnya0/dnya0/assets/84761609/321db30b-023a-4795-a979-3a743167a45d)

작년엔 1주차부터 바로 시작하는 것이 아닌 온보딩 기간이었다고 들었다. 그런데 이번엔 무슨 이유에선지 온보딩 기간을 갖지 않았다.

<br>

### 📌 기능 명세서 작성하기

개발하기 전 기능 명세서를 작성해야 했다. 명세서를 어떻게 작성할까 고민하다가 기능별로 작성하였다. 이게 어떤 기능이고, 무슨 일을 하는지 등을 적었다. api 명세서는 많이 적어봤지만 기능별로 명세서를 적는건 처음이라 어색했다. 체크박스를 이용해서 todo 리스트처럼 꾸며볼까 고민도 해봤지만 리스트 형식으로 결정하였다. 커밋 컨벤션과 코드 컨벤션을 지키기 위해 많이 노력했다.

![기능 명세서](https://github.com/dnya0/dnya0/assets/84761609/178ed297-d4c6-49d9-af65-ee606105209c)

<br>

### 🖥️ 구현

```
./
├── Application.java
├── Key.java
├── exception/
│   └── BaseballExceptionMassage.java
├── model/
│   ├── Computer.java
│   ├── Hint.java
│   └── User.java
└── play/
    ├── PlayGame.java
    └── Print.java
```

먼저 스프링을 이용해서 개발할 때는 항상 controller, service, repository로 구분해서 개발하였다. 그런데 막상 이렇게 개발하려하니 너무 애매하고 막막했다. 그래서 mvc 구조로 개발하자고 결정하였다. 이름에서 와닿지는 않지만 `Print.java`가 view, `PlayGame.java`가 controller, model 패키지 안에 있는 것들이 model들이다. PlayGame에서는 전체적인 게임을 제어한다.

<br>

### 🔖 느낀 점

숫자야구는 그렇게 어렵진 않았다. 그런데 내가 제출하는 시간에 사람이 몰려서 계속 빌드 오류가 뜨는 것이다. 그 때 많이 당황했다. 사실 코드리뷰도 하고싶었는데, 디스코드에 코드리뷰 해달라고 올려도 반응이 없고 잘 읽었다는 답변밖에 없어서 이 이후론 올리지 않았다. 많이 아쉬웠지만 코드리뷰는 캡스톤에서도 많이 하니까...

<br>

## 🚘 2주차 - 자동차 경주

> [2주차 코드 살펴보기](https://github.com/dnya0/java-racingcar-6/tree/dnya0)

![26개의 커밋](https://github.com/dnya0/dnya0/assets/84761609/17380ff9-3d9e-461d-96a7-4a63fcce12f7)

<br>

2주차 미션부턴 공통 피드백 제공되었다. 프로그래밍 요구사항도 추가되었는데 내용은 다음과 같다.

- indent(인덴트, 들여쓰기) depth를 3이 넘지 않도록 구현한다. 2까지만 허용한다.
- 3항 연산자를 쓰지 않는다.
- 함수(또는 메서드)가 한 가지 일만 하도록 최대한 작게 만들어라.
- JUnit 5와 AssertJ를 이용하여 본인이 정리한 기능 목록이 정상 동작함을 테스트 코드로 확인한다.

이 요구사항을 보고 좌절했다. 나는 3항 연산자를 즐겨 쓰는 사람이었기 때문에..

<br>

### 📌 기능 명세서

![기능 요구사항](https://github.com/dnya0/dnya0/assets/84761609/a721a66d-6417-4c12-8ddb-01e4b0e330a3)

먼저 추가 요구사항을 잊지 않기 위해 위에 적어두었다. 크게 Controller, View, Model로 나누어서 각각 기능을 적었다. 기능 명세서를 작성하면서 머리속으로 한 번 코딩이 되기 때문에 2번 구현하는 듯한 느낌이 들었다. 오히려 기능 명세서를 적으니 막히는 것이 별로 없어서 좋았다.


<br>

### 🖥️ 구현

```
./
├── Application.java
├── controller/
│   └── RacingController.java
├── message/
│   └── ExceptionMessage.java
├── model/
│   ├── Car.java
│   └── CarList.java
├── validator/
│   └── Validator.java
└── view/
    └── RacingView.java

```

controller, model, view로 나누어서 패키지를 생성하여 개발하였다. 또 Car 객체의 List를 일급 컬렉션으로 만들어 개발하였다.

<br>

### 🔖 느낀 점

2주차까지 큰 고비는 없었다. 2주차의 주제가 꽤나 흥미로웠고 출력물을 보는게 재미있었기 때문에 즐거운 기억만 남아있다. 오히려 1주차보다 쉬운 느낌..


<br>

## 🎱 3주차 - 로또

> [3주차 코드 살펴보기](https://github.com/dnya0/java-lotto-6/tree/dnya0)

![46개의 커밋](https://github.com/dnya0/dnya0/assets/84761609/d5f8542d-5ce3-41db-b8e5-20a8e9a96d03)


이번 주제는 내가 깜빡하고 처리못한 예외가 있었다. 수가 45 이하인 자연수가 아니면 예외를 발생시켜야 했는데 깜빡하고 그걸 안한 것이다. 제출하고 다음 날 공통 피드백을 보며 깨달았다. 아쉬움이 많이 남았다.

요구사항이 추가되었는데 추가된 요구사항은 다음과 같다.

- 함수(또는 메서드)의 길이가 15라인을 넘어가지 않도록 구현한다.
- else 예약어를 쓰지 않는다.
- Java Enum을 적용한다.
- 도메인 로직에 단위 테스트를 구현해야 한다. 단, UI(System.out, System.in, Scanner) 로직은 제외한다.

Enum을 어디서 사용해야할까 고민을 많이 했다. 결국 당첨 내역을 저장할 때 사용하는 것으로 결정하였다.

<br>

### 📌 기능 명세서 작성

![기능 명세서](https://github.com/dnya0/dnya0/assets/84761609/1a7ca368-34a5-4027-bc60-8c2b65091dae)

기능 명세서도 2주차와 비슷하게 적었다. 살아움직이는 문서?를 만들라고 해서 처음에 정한 기능 목록대로 무조건 만들어야되는게 아니라는걸 깨달았다.

<br>

### 🖥️ 구현

```
├── Application.java
├── contoller/
│   └── LottoController.java
├── exception/
│   └── ExceptionMessage.java
├── model/
│   ├── Lotto.java
│   ├── LottoList.java
│   ├── PurchaseAmount.java
│   ├── Result.java
│   ├── WinningNumber.java
│   └── value/
│       └── Matching.java
├── validator/
│   └── Validator.java
└── view/
    ├── LottoFormatter.java
    └── LottoView.java

```

Lotto의 리스트를 저장하는 LottoList를 만들었다. enum은 model의 value 폴더 안에 저장하였다. 개발하면서 알게된건데 enum은 dto나 entity와 구별해서 저장해둬야 나중에 찾기 편하다. 로또를 구현하면서도 따로 분류해두었다.

enum을 사용할 때도 고민이 많았다. 인터넷을 보다 알게된건데 저런 식으로 언더바를 추가해 가독성을 높일 수 있다는 것이다. 가령 `100000000000000`이런 숫자를 저장하면 한눈에 알아보기 힘들다. 그럴 때 언더바를 추가하여 `100_000_000_000_000` 이런 식으로 보기 편하게 저장할 수 있다.

```java
public enum Matching {

    THREE("3개 일치", 5_000, 3),
    FOUR("4개 일치", 50_000, 4),
    FIVE("5개 일치", 1_500_000, 5),
    FIVE_BONUS("5개 일치, 보너스 볼 일치", 30_000_000, 7),
    SIX("6개 일치", 2_000_000_000, 6);

    private static final Map<Integer, String> matchingMap = Collections.unmodifiableMap(
            Stream.of(values()).collect(Collectors.toMap(Matching::getCount, Matching::name))
    );

    private final String matchingNumber;
    private final int prizeMoney;
    private final int count;

    //...constructor

    public static Matching of(final int count) {
        return Matching.valueOf(matchingMap.get(count));
    }

    //...getter
}
```

또한 enum에서 어떻게 객체를 찾을까 고민했는데, 나는 map을 만들어서 찾도록 구현하였다. 

<br>

### 🔖 느낀 점

살짝 아쉬움이 많이 남는 코드였다. 예외처리를 놓쳐서 더 그렇게 느끼는 것 같다. 3주차까진 할만했다.


<br>

## 🎄 4주차 - 크리스마스 프로모션

> [4주차 코드 살펴보기](https://github.com/dnya0/java-christmas-6-dnya0)

![약 61개의 커밋](https://github.com/dnya0/dnya0/assets/84761609/6b538ce3-d6bb-4b29-832f-cf79e0371635)

커밋 수가 60개를 넘겼다. 많아야 3~40개였던 것 같은데.. 이번 과제는 꽤 많이 어려웠다. 가장 크게 바뀐 점은 그동안 레포지토리를 포크해서 본인의 아이디로 된 브랜치에서 개발한 후 pr을 보내는 형식이었다면, 4주차 과제는 템플릿을 가져와 비공개 레포지토리로 생성한 후 main 브랜치에 개발하는 것이다.

![기능 요구사항](https://github.com/dnya0/dnya0/assets/84761609/c6bf872a-e5f3-4788-b3e4-de3191d1c4fc)

요구사항부터 이메일 형식으로 왔다. 디스코드를 보니 다들 설렌다고 하던데.. 난 꽤나 당황했다. 그동안 리스트에 적혀있는 것을 구현만 하면 됐는데 이건 메일을 읽고 파악해야했기 때문이다. 그래도 밑의 메뉴를 보기 전까진 흥미로웠다. 메뉴를 보는 순간 db를 사용하는 것이 아닌데 저걸 어떻게 저장해서 갖고오나가 가장 머리 아픈 주제였다. 또, 각각의 메뉴에 따라 할인을 해 총 금액을 계산해야되는 것에서 자신감이 없어졌다.

<br>

### 📌 기능 명세서 작성

일단 어떻게 구현할 지 고민하기 위해 아래와 같이 이벤트에 대하여 날짜, 계획, 주의사항, 개발 요청 사항 등을 정리를 해놓았다.

![기능 명세서](https://github.com/dnya0/dnya0/assets/84761609/1aadd253-3dc5-4344-b3cd-0e573de61253)

이어서 구현할 기능 목록을 작성하였는데, 이제까지 작성했던 것 중에 가장 막막했다. 시간이 부족해보였고, 혜택 내역에 대해 고민이 많았기 때문에 이것은 어떻게 개발되는지에 따라 방향을 정하기로 결정하고 개발부터 시작했다.

<br>

### 🖥️ 구현

```
./
├── Application.java
├── controller/
│   └── ChristmasController.java
├── exception/
│   ├── ChristmasException.java
│   └── ExceptionMessage.java
├── model/
│   ├── Benefit.java
│   ├── BenefitList.java
│   ├── ChristmasEvent.java
│   ├── DecemberEvent.java
│   ├── Discount.java
│   ├── Event.java
│   ├── OrderDetails.java
│   ├── Reservation.java
│   └── value/
│       ├── Appetizer.java
│       ├── Beverage.java
│       ├── CourseMeal.java
│       ├── Dessert.java
│       ├── EventBadge.java
│       ├── EventDate.java
│       ├── MainDish.java
│       └── Menu.java
└── view/
    ├── InputView.java
    └── OutputView.java

```

우선 메뉴를 enum에 각각 저장하고 통합하여 가져와야했기 때문에 Menu 인터페이스를 구현하였다.

```java
public interface Menu {

    String getMenuName();
    int getPrice();

}
```

또 각각의 enum 객체들은 Menu를 상속받는 것으로 개발하였다.

```java
public enum Appetizer implements Menu {

    MUSHROOM_SOUP("양송이수프", 6_000),
    TAPAS("타파스", 5_500),
    CAESAR_SALAD("시저샐러드", 8_000);

    private final String appetizerName;
    private final int price;

    Appetizer(String appetizerName, int price) {
        this.appetizerName = appetizerName;
        this.price = price;
    }

    @Override
    public String getMenuName() {
        return appetizerName;
    }

    @Override
    public int getPrice() {
        return price;
    }
}

```

그리고 각각의 메뉴 객체들의 값을 저장하는 enum을 하나 더 만들었다. 여기서 가장 고민이 많았다.


```java
public enum CourseMeal {

    APPETIZER(Appetizer.values()),
    MAIN_DISH(MainDish.values()),
    DESSERT(Dessert.values()),
    BEVERAGE(Beverage.values());

    private final List<Menu> menus;

    CourseMeal(Menu[] menus) {
        this.menus = List.of(menus);
    }

    public static Menu findMenuItemByName(String itemName) {
        //... return menu
    }

}
```

<br>

이벤트는 Event 추상클래스를 만들어 DecemberEvent와 ChristmasEvent가 상속받도록 하였다.

```java
public abstract class Event {

    private final EventDate eventDate;

    public Event(EventDate eventDate) {
        this.eventDate = eventDate;
    }

    public EventDate getEventDate() {
        return eventDate;
    }
}
```

```java
public class ChristmasEvent extends Event {

    private final LocalDate reservationDate;

    public ChristmasEvent(LocalDate reservationDate) {
        super(EventDate.CHRISTMAS_EVENT);
        this.reservationDate = reservationDate;
    }

}
```

혜택 내역도 Benefit이라는 객체를 만들어 이름과 혜택 금액을 저장하고, BenefitList라는 일급 컬렉션을 만들어 혜택 내역 계산과 총 혜택 금액 계산 등을 하도록 했다.

<br>

### 🔖 느낀 점

그동안 프로그래밍을 배우고 나서부터 코드를 이쁘게 적기 위해 고민을 많이 하였는데.. 이번 4주차를 하면서 예외처리를 하는 로직이 꽤나 더럽게 나와.. 굴러가는 쓰레기라도 만들자는 마음으로 구현하였다. 너무 가슴이 아팠다.. 구현하는 내내 이대로 코딩테스트가 나온다면 5시간 안에 구현하기는 글렀다는 생각만 했다.

그래도 우아한 프리코스를 하면서 코드 보는 눈이 생긴 것 같고, 확실히 성장한 것 같다는 느낌이 들어서 좋았다. 코테에 붙지 않더라도 많이 배우고 경험한 것 같아 기뻤다. 최선을 다해서 후회는 없다. 다음에 우테코가 또 열린다면 다시 지원해볼 것 같다.

