---
title: 우아한 테크코스 프리코스 회고
author: dnya0
date:   2024-10-23 01:44:00 +0900
categories: [Post, WoowaCourse]
tag: [Study, BootCamp, WoowaCourse, Java]
---

## 🥑 들어가며

나는 현재 회사를 다니고 있지만 부트캠프에서 여러 경험을 해보는 것이 꿈이다. 그래서 우아한테크코스를 지원하게 되었다. 우테코는 떨어지더라도 프리코스 동안 실력이 많이 향상되기에 손해보는 것이 없다. 또, 프리코스와 같은 문제를 받아 푸는 것이 즐겁기에 열심히 하게 되는 것 같다.

<br>

## ➕ 1주차: 문자열 덧셈 계산기

처음에 문제를 보고 환호했다. 계산기라니! 대학생 때 안드로이드 과목에서 시험 대체 과제로 계산기를 만든 경험이 있다. 나는 시험 대체 과제를 즐거워하는 편이었고, 계산기를 만든다는 것에 신이 나 열심히 개발한 기억이 난다. 살짝 아쉬웠던건 덧셈만 한다는 것.. 기본적인 사칙연산만 있었더라도 좋았을텐데.. 1주차에 너무 힘을 준게 되려나 싶기도 하다.

### 📌 프로젝트 구조

나는 먼저 계획한대로 기능 요구사항을 적으면서 전체적으로 프로젝트를 어떻게 개발할 지 고민했다. 출력, 입력 기능만 하는 클래스를 따로 만들고, 구분자를 나누는 클래스를 따로 두자고 결심하였다. 이후 구분자를 상속받는 커스텀 구분자 클래스를 계획했고 덧셈은 Integer로 변환하거나 덧셈하면서 예외가 발생할 시 BigInteger로 계산시키도록 계획하였다.

test 코드 작성하는 것을 기능을 개발하던 중 떠올려버렸다. 기능 단위별로 커밋하라고 되어있어서 그냥 기능이 완성되면 테스트 코드도 같이 짜서 올리는 방식으로 결정하였다. 테스트 코드를 커밋하고 나서야 패키지를 만들어서 나눌걸 싶었지만 패키지 이동을 하지 말라는 사항이 있어 그대로 진행했다.

```
.
├── main
│   └── java
│       └── calculator
│           ├── Application.java
│           ├── calulate
│           │   └── Plus.java
│           ├── delimiter
│           │   ├── CustomDelimiter.java
│           │   └── Delimiter.java
│           ├── exception
│           │   └── CalculatorException.java
│           └── view
│               ├── Input.java
│               └── Output.java
└── test
    └── java
        └── calculator
            ├── ApplicationTest.java
            ├── CustomDelimiterTest.java
            ├── DelimiterTest.java
            ├── InputTest.java
            └── PlusTest.java

```

#### 프로젝트 전체 구조 설계

![8152825D-614D-4B4E-A03A-67BEA0A88872_1_102_a](https://github.com/user-attachments/assets/cfd14b6b-58a5-4c57-b45d-7de7e6169bf7)

아무래도 공책에 적은걸 사진찍은 것이라 많이 아쉽다. 2주차부턴 아이패드로 적어야겠다.

### 📌 기능 설계 및 구현

```
## 커스텀 구분자

- 커스텀 구분자는 구분자를 상속받은 자식 객체로 구성한다.
- [x] 입력한 문자열에 커스텀 구분자가 존재할 경우 구분자와 커스텀 구분자로 문자열을 분리한다.
- [x] `//` `\n` 사이에 있는 문자의 경우 커스텀 구분자로 사용한다.
  - 예 1) "//;\n1;2;3" -> 커스텀 구분자 = `;`, 1 + 2 + 3 -> 6
- [x] `//`로 시작해서 `\n`로 닫히지 않고 안의 문자열에 `//`와 `\n`이 포함 되어있는 경우 `IllegalArgumentException`을 발생시킨다.
  - 예 1) "////;\n3\n"
- [x] 커스텀 구분자는 여러 개 작성할 수 있다.
  - 예 1) "// \n//temp\n1temp2 3" -> 커스텀 구분자: ` `, `temp`, 1 + 2 + 3 -> 6
- [x] 구분자와 양수로 구성된 문자열인지 확인한다.
```

기능 설계를 이런 식으로 짜봤는데 이 방식으로 하는게 맞는지 모르겠다. 보통 유저 로그인 기능을 만든다고 가정하면, `feature/user` 브랜치를 따로 만들어서 이슈나 pr로 유저 로그인 기능이라 적은 뒤 커밋(보통 `feat: 비밀번호 검증 기능 구현` 이런 식으로 커밋 메세지를 작성한다.)하고 머지하는 방식이었다. 그래서 기능 단위로 커밋하라는 것이 **커스텀 구분자 구현**을 커밋하는 것인지 **커스텀 구분자는 여러 개 작성할 수 있다**를 커밋하는 것인지 헷갈렸다. 나는 후자를 선택했지만 이게 맞는 것인가 고민이 많았다. 기능 명세서 양식이라도 있었으면 좋았을텐데 기능 명세서 작성 방법은 회사마다 프로젝트마다 다를 것이니..

### 📌 느낀점

생각보다 입력에서 시간을 많이 잡아먹었다. 전까진 잘 개발하고 있었는데 어느순간 `\\n`이 아닌 `\n`이 입력되는 것이라 착각을 한 것이다. 여기서 시간을 좀 깎아먹었다. 이 외에도 코드를 보면 아쉬운 점이 많다. 특히 구분자와 커스텀 구분자를 구현하는 클래스에서 느꼈다. 어떻게 잘 하면 깔끔하고 잘 동작하는 코드를 짤 수 있을 것 같은데.. 회사 다니느라 퇴근 후와 주말에만 시간이 있다보니 아름다운 코드를 위한 시간이 부족한 것 같아 아쉽다. 

그리고 메소드 이름 짓는데에 더 많은 노력을 기울여야겠다... 지금 다시 코드를 보니 메소드 이름이 가관이다..

1주차는 이렇게 끝이 났다. 2주차엔 어떤 과제가 나올지 기대가 많이 된다.

<br>

## 🚘 2주차: 레이싱 카

2주차는 중간회고가 있었다. 내가 제대로 하고 있는지, 신청서에 쓴 내용대로 잘 진행하고 있는지 다시 돌아보는 계기가 되었다. 벌써 프리코스의 반이 지났다니..

### 📌 프로젝트 구조

이번엔 Util을 담당하는 클래스를 생성하였다. 아직까지도 어떻게 구현해야할 지, 이게 맞는 구현방법인지 고민이 많다. 언젠가 답을 찾을 수 있겠지..

```
.
├── main
│   └── java
│       └── racingcar
│           ├── Application.java
│           ├── exception
│           │   └── RacingException.java
│           ├── io
│           │   ├── Input.java
│           │   └── Output.java
│           ├── racing
│           │   ├── Car.java
│           │   └── Racing.java
│           └── util
│               └── RacingUtils.java
└── test
    └── java
        └── racingcar
            ├── ApplicationTest.java
            ├── io
            │   └── InputTest.java
            └── util
                └── RacingUtilsTest.java

```

### 📌 기능 설계 및 구현

현실에서 Car는 이름과 이동 거리만 나타내는 것이 아닌 앞으로 나아가고 회전하고 정지하고 불을 비출 수 있다. 이 외에도 가격, 차의 색 등을 저장할 수도 있겠다. 그동안 배운 바에 따르면 객체는 각각 책임을 가지고 있으며, 스스로 행동할 수 있어야 한다. 다행히 이 과제의 *Car*는 크고 복잡한 기능 없이 이름과 이동 거리, 그리고 전진과 정지의 기능만 갖고있다. 

나는 우선 Car라는 객체를 만들어 name과 distance를 저장하도록 하였다. 또, car를 움직이게 하는 것은 `Random` 클래스를 사용하여 랜덤값이 4 이상일 때. Car 객체 외부에서 랜덤 값을 구한 다음 4 이상일 때만 이동거리를 +1 시키는 것은 살아있는 객체가 아니라고 느꼈다. 그래서 해당 로직을 Car 객체 안에 넣어 car가 스스로 판단하여 움직이도록 구현하였다.

### 📌 느낀점

2주차 공통 피드백을 보니 README를 고치면 안되는 것은 아닌 것 같다. 그런데 이 글을 보고 말았다. `기능 목록을 작성할 때 클래스 설계와 구현, 메서드 설계와 구현 같은 상세한 내용은 포함하지 않는다.` 맞는 말이다. 그러나 나는 목표에 프로젝트 설계를 간단히 하고 개발을 시작하겠다 했었는데,... 아무래도 목표가 많이 바뀔 것 같다.. 

<br>

## 🎱 3주차: 로또

우선 행위자가 누가 있을지 고민하였다. 로또를 사고 파는데에 객체가 누가 있고 어떤 책임을 져야 하는가. 우선 로또 판매자, 로또, 로또 구매자, 로또 번호 생성기 등이 있을 것이다. 그렇다면 이들은 각각 어떤 행위를 해야 하는가.

- 로또 판매자
  - 로또를 1장당 1000원의 가격으로 판매한다.
- 로또 구매자
  - 로또를 1장당 1000원의 가격으로 구매한다.
- 로또 번호 생성기
  - 로또 번호를 생성한다.

결국 로또 판매와 구매를 합쳐 Purchase라는 엔티티를 만들어 구현하였다.

### 📌 프로젝트 구조

스프링에서 빈을 관리하듯이 한 클래스에서 의존성 주입을 하려 노력했다.

```
.
├── ./Application.java
├── ./LottoApplication.java
├── ./config
│   └── ./config/LottoConfig.java
├── ./controller
│   ├── ./controller/LottoController.java
│   └── ./controller/dto
│       ├── ./controller/dto/BonusNumberSaveRequest.java
│       ├── ./controller/dto/LottoPurchaseResponse.java
│       ├── ./controller/dto/PrizeResultDto.java
│       ├── ./controller/dto/PrizeResultResponse.java
│       └── ./controller/dto/WinningNumberSaveResponse.java
├── ./domain
│   ├── ./domain/Lotto.java
│   ├── ./domain/LottoPurchase.java
│   ├── ./domain/WinningNumber.java
│   └── ./domain/value
│       └── ./domain/value/Standard.java
├── ./exception
│   ├── ./exception/ExceptionMessage.java
│   └── ./exception/LottoException.java
├── ./repository
│   ├── ./repository/LottoPurchaseRepository.java
│   ├── ./repository/LottoRepository.java
│   └── ./repository/WinningNumberRepository.java
├── ./service
│   └── ./service/LottoService.java
├── ./utils
│   └── ./utils/LottoUtils.java
└── ./view
    ├── ./view/Input.java
    └── ./view/Output.java

```

### 📌 느낀점

이번 과제는 마지막에 통계를 계산하는데 오래 걸렸던 것 같다. 그러나 꽤 자랑스러웠던 점도 있다. 이번엔 웹 형식으로 하려고 마음을 먹었는데, 그래서 최대한 스프링처럼 개발하려고 노력하였다. 모든 의존성 주입을 config 파일을 만들어 그 곳에서 해결하려 하였다.

처음에 어떤 객체가 있어야 할까 고민을 많이 했다. 객체는 각자 책임을 지고 스스로 행동을 하기 때문에 이 로또 프로그램에선 어떤 객체가, 어떤 책임을, 어떤 행동을 해야하는지 고민을 했었다. 처음엔 판매자, 구매자, 로또, 로또 기계 등등을 생각하였으나 최종적으로는 로또 구매, 로또, 당첨 번호 등으로 끝난 것 같다. 

아쉬운 점은 로또에 다른 변수를 더 추가하지 못한다는 점..

웹처럼 동작하게 하기 위해 repository를 만들어서 그 곳에 각각 객체의 List를 정의해두고 저장 및 삭제 로직을 넣어두었다. 리스트의 인덱스값을 id로 사용하는 방식으로 구현하였다.

드디어 마지막 4주차를 눈 앞에 두고 있다. 내가 마지막까지 잘 끝낼 수 있기를..

<br>

## 💰 4주차: 편의점

역시 우아한 형제들에서 주관하는 부트캠프라 그런지 마지막 프리코스는 결제와 관련됐다. 여러 일정이 겹쳐 하루만에 코드를 짤 수 밖에 없었다. 많이 아쉬웠지만 어쩔 수 없었다고 생각한다.

### 📌 프로젝트 구조

```
.
├── Application.java
├── context
│   └── BeanContext.java
├── controller
│   ├── ConvenienceStoreController.java
│   └── dto
│       ├── ExtraOrderDto.java
│       ├── ExtraOrderRequest.java
│       ├── MainProductListResponse.java
│       ├── MembershipRequest.java
│       ├── OrderDto.java
│       ├── OrderRequest.java
│       ├── ProductListDto.java
│       ├── PromotionQuantityDto.java
│       ├── PromotionQuantityResponse.java
│       ├── ReceiptEventDto.java
│       ├── ReceiptOrderDto.java
│       ├── ReceiptPaymentDto.java
│       ├── ReceiptResponse.java
│       └── ReceiptTotalPriceDto.java
├── domain
│   ├── common
│   │   ├── BaseEntity.java
│   │   ├── DomainFactory.java
│   │   └── value
│   │       ├── Domains.java
│   │       └── FilePath.java
│   ├── order
│   │   ├── Order.java
│   │   └── OrderRepository.java
│   ├── payment
│   │   ├── Payment.java
│   │   └── PaymentRepository.java
│   ├── products
│   │   ├── Products.java
│   │   ├── ProductsFactory.java
│   │   └── ProductsRepository.java
│   └── promotions
│       ├── Promotions.java
│       ├── PromotionsFactory.java
│       └── PromotionsRepository.java
├── exception
│   └── ConvenienceStoreException.java
├── service
│   ├── FileLoaderService.java
│   ├── OrderService.java
│   ├── PaymentService.java
│   └── ProductsService.java
├── utils
│   ├── FileReaderUtil.java
│   └── StoreUtil.java
└── view
    ├── InputView.java
    └── OutputView.java

```

### 📌 느낀점

우선 많은 시간을 들일 수 없었다는 점. 급하게 끝내느라 테스트 코드를 짜지 못했다. 역시 회사와 병행하는건 쉽지 않은 것 같다. 제일 크게 달라진 점은 역시 파일 입출력을 추상화하여 팩토리 패턴을 사용했다는 것? 최대한 여러 클래스를 넣어도 사용할 수 있도록 구현하고 싶었다.

처음엔 시간이 많아서 코드를 가독성 있고 이쁘게 짜고 싶었다. 그런데 뒤로 갈 수록 시간도 없고 구현하는데 급급하여 가독성 떨어지는 코드가 되어버렸다.. 정말 10분 남기고 겨우 구현한 뒤 제출했기 때문에 아쉬움이 많이 남는 코드였다.


dto도 따로 도메인 별로 정리했으면 좋았을텐데 하는 아쉬움도 있다.

<br>

최근 회사 일이 갑자기 바빠져서 퇴근하고 집에서 잠만 자고 있다.. 미루다가 2주나 지나서 회고를 쓰게 되었다. 3주차 회고는 작성해놓은 상태였는데 집에 와서 노트북을 열고 커밋 하나 하는게 왜이리 힘이 들던지.. 아마 이게 나의 마지막 프리코스가 되지 않을까.

전체적으로 작년보다 실력이 늘었다는 것을 깨달았다. 조금씩 코드를 보는 눈이 생기는 것 같다. 