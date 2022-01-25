---
tags: 
    - delegation
    - delegate
    - design pattern
title: Delegation Pattern
date: 2022/01/24
author: 김동환
description: delegation design pattern
disabled: false
categories:
  - general
---

> 자신이 아닌 다른 객체에 일을 위임(Delegation)시키고 처리하게 하는 패턴이다.
>

Delegation Pattern은 합성(Composition)관계를 이용하여 다른 객체에 일을 위임, 처리함으로써 코드의 **재사용성**과 **확장성**을 높일 수 있는 패턴이다. 코드의 재사용성과 확장성을 늘리는 방법인 상속(Inheritance)을 사용할 수 있지만 무분별한 상속은 오히려 객체의 유연성을 떨어트릴 수 있다.

## Usecase

일반적으로 코드의 재사용과 확장을 위해 사용한다. 클래스간 관계가 `IS-A` 의 관계이면 상속을 통해 강결합하여 처리하는 것이 좋다. 하지만 `HAS-A` 관계 인 경우 상속의 경우로 처리한다면 부모와 강결합으로 인해 자식의 유연성이 떨어지게 된다. 이러한 경우 Delegation Pattern을 활용하여 처리하는 것이 유연하게 대처할 수 있다.

1. **특정 인터페이스를 다른 인터페이스에 적용을 하려는 경우에 사용**([Adapter Pattern](https://en.wikipedia.org/wiki/Adapter_pattern))할 수 있다.

   특정 인터페이스를 사용하는 작업에서 다른 인터페이스의 일을 위임하여 특정 인터페이스에 맞게 동작하게 만들 수 있다. 예를 들어 C-type 충전기에 아이폰을 충전하기 위해 사용하는 C-type to Lightning 어댑터가 필요하다. **해당 어댑터가 라이트닝 타입의 일을 위임하여 처리한다.**

    　
2. **복잡한 클래스를 숨기는 용도로 사용**([Facade Pattern](https://en.wikipedia.org/wiki/Facade_pattern))할 수 있다.

   예를 들어 컴퓨터를 부팅하기 위한 CPU, Memory, Drive의 작업이 있다면 이를 숨기고 Computer.startComputer()로 표현 할 수 있을 것이다. 즉 **Computer클래스에게 CPU, Memory, Drive의 일을 위임하여 처리한다.**

    　
3. 특정 객체에 **여러 기능을 추가하려 할 때 사용**([Decorator Pattern](https://en.wikipedia.org/wiki/Decorator_pattern)) 할 수 있다.

   예를 들어 커피를 만드는 커피머신이 있다고 하자. 스타벅스에서 커피를 마실 때 다양한 커피(바닐라크림 프라푸치노 + 카라멜 시럽 + 헤이즐넛 시럽 + 휘핑 크림 + 카라멜 드리즐)를 주문 할 수 있다. 한번에 이러한 조합의 커피를 만드는 커피머신을 만들었다고 하면, 위와 다른 조합의 커피는 그에 맞는 새로운 커피머신을 만들어야한다.

   즉 이러한 객체들의 조합을 하나의 클래스로 만들려고 할 경우에는 모든 경우의 수 만큼 클래스를 생성해야하는 문제가 생긴다. 그리하여, 각각의 기능들이 추가될 수 있는 커피머신(데코레이터)를 만들면 된다. **기본적인 커피머신(커피 만들기)의 일을 위임하여 추가적인 기능과 함께 일을 처리한다.**


    　
## Example

![/assets/images/general/payservice_example_diagram.png](/assets/images/general/payservice_example_diagram.png)

```java
public interface PayService {

  String pay(long amount);
}

public enum PayTypes {

  NAVER_PAY,
  KAKAO_PAY
}

public class NaverPayService implements PayService {

  @Override
  public String pay(long amount) {
    return "NAVER: " + amount;
  }
}

public class KaKaoPayService implements PayService {

  @Override
  public String pay(long amount) {
    return "KAKAO: " + amount;
  }
}

public class CashService implements PayService {

  @Override
  public String pay(long amount) {
    return "CASH: " + amount;
  }
}
```

　
```java
public class PayDelegator {

  // Composition
  @Setter
  private PayService payService;

  public void setPayCategory(PayTypes payCategory) {
    switch (payCategory) {
      case NAVER_PAY:
        this.payService = new NaverPayService();
        break;
      case KAKAO_PAY:
        this.payService = new KaKaoPayService();
        break;
      default:
        this.payService = new CashService();
        break;
    }
  }

  public String pay(long amount) {
    return this.payService.pay(amount);
  }
}
```

　
```java
@Test
void payWithNaverPay() {
  /* GIVEN */
  PayDelegator payDelegator = new PayDelegator();
  payDelegator.setPayCategory(PayTypes.NAVER_PAY);

  /* WHEN */
  String result = payDelegator.pay(15000L);

  /* THEN */
  assertThat(result, containsString("NAVER"));
}

@Test
void payWithKakaoPay() {
  /* GIVEN */
  PayDelegator payDelegator = new PayDelegator();
  payDelegator.setPayCategory(PayTypes.KAKAO_PAY);

  /* WHEN */
  String result = payDelegator.pay(15000L);

  /* THEN */
  assertThat(result, containsString("KAKAO"));
}
```

　

`PayService` 에서는 결제 관련 서비스(`String pay(long amount)`)를 제공하고 있다. `PayDelegator` 는 결제 관련 서비스를 위임 받고 처리한다.