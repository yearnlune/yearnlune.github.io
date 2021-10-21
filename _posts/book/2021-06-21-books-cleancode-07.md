---
tags: 
    - clean code
    - 클린 코드
    - 예외 처리
title: 클린 코드(Clean Code) 리뷰 - 07
date: 2021/06/21
author: 김동환
description: 클린 코드(Clean Code) 리뷰 - 7장 예외 처리
disabled: false
categories:
  - general
---

　
# 오류 처리

## 오류 코드보다는 예외를 사용하라

예전에는 예외를 지원하지 않는 프로그래밍 언어가 많았다. 하지만 요새 대부분의 프로그래밍 언어들은 예외를 지원한다. 그렇기에 함수 호출자에게 오류 코드를 반환하는 방법은 자칫 복잡해질 수 있다. 또한, 비즈니스 로직과 함께 뒤섞일 수도 있다. 그렇기에 예외를 던지거나 처리하는 방법이 좋다.

## Try-Catch-Finally 문부터 작성하라

try블록에서 어느 시점에서든 중단되면 catch블록으로 넘어간다. 즉 try블록은 트랜잭션과 비슷하다. try블록에서 이슈가 생기면 호출자가 기대하는 상태를 catch블록에 정의하기 쉽다. 자연스럽게 try블록의 트랜잭션 범위부터 구현하게 되어 트랜잭션 본질을 유지하기 쉬워진다.

```java
@EventListener
public void handleOrder(OrderEvent event) {
    try {
        //...
        validate(event);
        //...
    } catch (UnAuthroizedTokenException e) {
        publisher.publish(new UnAuthroizedTokenEvent(e));
    } catch (OrderNotFoundException e) {
        publisher.publish(new OrderNotFoundEvent(e));
    }
}
```

## 미확인 예외를 사용하라

자바에서 확인된 예외(checked exception)를 활용하여 예외를 처리하는 것은 때로는 유용할지도 모른다. 하지만 확인된 예외를 던져 처리를 한다면 그 메서드를 호출하는 상위 메서드들은 확인된 예외들을 던져야한다. 만약 새로운 예외를 추가한다면 상위 메서드를 모두 고쳐야 한다. 최하위 메서드에서 던지는 예외를 상위 메서드들이 알아야 하므로 캡슐화가 깨진다.

C#, C++, 파이썬, 루비 등은 확인된 예외를 지원하지 않는다. 하지만 자바와 같이 안정적인 소프트웨어를 구현하기에 무리가 없다. 즉 확인된 오류를 사용하여 생기는 이익과 불이익을 고려해야 한다. 하지만 확인된 오류를 사용함으로써 생기는 의존성이라는 비용은 이익보다 대체로 크다.

## null을 반환하지 마라

메서드가 null을 반환한다면 호출하는 부분에서는 매번 호출한 값이 null인지를 판단해야 한다. 만약 null 확인을 빼먹는다면 NullPointerException이 발생할 것이다. 그렇기에 null보다는 예외를 던지거나 초기화된 값을 반환하는 것이 낫다.

```java
// BAD CASE
public void registerItem(Item item) {
    if (item != null) {
        ItemRegistry registry = persistentStore.getItemRegistry();
        if (registry != null) {
            Item existing = registry.getItem(item.getID());
            if (existing.getBillingPeriod().hasRetailOwner()) {
                existing.register(item);
            }
        }
    }
}

// GOOD CASE
public void getItems() {
    List<Item> items;
    // ... 
    return Optional.ofNullable(items).orElse(Collections.emptyList());
}
```

## null을 전달하지 마라

정상적인 인수로 null을 기대하는 API가 아니라면 최대한 메서드로 null을 전달하는 코드는 피해야 한다.

```java
public double xProjection(Point p1, Point p2) {
    return (p2.x - p1.x) * 1.5;
}
```

만약 인수가 null을 전달한다면 `NullPointerException` 이 발생한다.

```java
// CASE #1 Exception
public double xProjection(Point p1, Point p2) {
    if (p1 == null || p2 == null) {
        throw InvalidArgumentException("Invalid argument for MetricsCalculator.xProjection");
    }

    return (p2.x - p1.x) * 1.5;
}

// CASE #2 assert
public double xProjection(Point p1, Point p2) {
    assert p1 != null : "p1 should not be null";
    assert p2 != null : "p2 should not be null";
    
    return (p2.x - p1.x) * 1.5;
}
```

　
# 결론

프로그래밍 개발을 할 때 오류처리를 하는 것은 중요하다. 하지만 오류 처리로 인해 비즈니스 로직과 혼합되어 유지 보수하는 데 어려움을 겪을 수 있다. 최대한 비즈니스 로직과 오류 처리를 분리하여 클린 코드를 작성해야 한다.