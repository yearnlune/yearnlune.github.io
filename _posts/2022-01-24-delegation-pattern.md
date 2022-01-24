---
tags: 
    - delegation
    - delegate
    - design pattern
title: Delegation Pattern
date: 2022/01/24
author: 김동환
description: delegation design pattern
disabled: true
categories:
  - general
---

# Delegation Pattern

> 자신이 아닌 다른 객체에 일을 위임(Delegation)시키고 처리하게 하는 패턴이다.
>

Delegation Pattern은  합성(Composition)관계를 이용하여 다른 객체에 일을 위임, 처리함으로써 코드의 재사용성과 확장성을 높일 수 있는 디자인 패턴이다. 코드의 재사용성과 확장성을 늘리는 방법인 상속(Inheritance)을 사용할 수 있지만 무분별한 상속은 오히려 객체의 유연성을 떨어트릴 수 있다.

## 예시

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