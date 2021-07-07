---
tags: 
    - clean code
    - 클린 코드
    - 경계
title: 클린 코드(Clean Code) 리뷰 - 08
date: 2021/07/08
author: 김동환
description: 클린 코드(Clean Code) 리뷰 - 8장 경계
disabled: true
categories:
  - general
---

　
# 경계

소프트웨어를 개발할 때 우리는 모든 것을 직접 개발하지 않는다. 외부 라이브러리나 패키지를 사거나 오픈 소스를 이용한다. 또는 사내에서 개발한 컴포넌트를 사용한다. 어떤식이든 우리가 구현하는 코드와 외부에서 가져온 코드를 깔끔하게 통합시켜야만 한다.

## 외부코드 사용하기

우리가 사용하는 외부코드들은 많은 기능들을 제공해주고 있다. 하지만 이런 많은 기능들 모두 사용하지 않으며, 오히려 비즈니스 로직을 구현하고 제공할 때 해가 될 수 도 있다. 예를 들어 `java.util.Map`의 경우 다음과 같은 기능들을 제공해주고 있다.

```groovy
int size();
boolean isEmpty();
boolean containsKey(Object key);
boolean containsValue(Object value);
V get(Object key);
V put(K key, V value);
V remove(Object key);
void putAll(Map<? extends K, ? extends V> m);
void clear();
...
```

`java.util.Map` 을 활용하여 비즈니스 로직을 구현할 때 해당 기능들을 어느 누구나 사용가능하다

```groovy
// BookCategoryService.java

// 도서관 책의 청구기호와 카테고리를 가진 컬렉션 
Map<String, BookCategory> bookCategories = new HashMap<>();
```

해당 컬렉션은 `get()` 을 활용하여 읽기만을 제공하는 컬렉션으로 제공하고 싶지만 의도치 않은 기능들도 같이 제공될 수 있다. 어느 누구도 `clear()` 를 통한 삭제가 이뤄질 수 있다.

그렇기 때문에 필요한 인터페이스만 제공하기 위해서 **Wrapping을 통해서 기능들을 제한하여 제공**해야 한다.

## 외부코드 살피고 익히기

외부 코드를 바로 적용하기 이전에 외부코드를 테스트를 진행하여 익혀야 한다. 이를 **학습 테스트**라 부른다. 학습 테스트를 진행하여 외부 코드를 제대로 이해하는지 알아볼 수 있다. 이렇게 제대로 이해해야 우리쪽 코드에 적절하게 녹여낼 수 있다.

## 학습 테스트

필요한 API의 지식을 확보할 수 있는 손쉬운 방법 중 하나이다. 또한 새로운 버전이 나오더라도 테스트를 통해 새로운 버전의 외부 코드가 우리 코드의 이전과 다른 영향을 주는지도 확인할 수 있다. 이러한 학습 테스트를 진행하지 않는다면 새로운 버전의 외부 코드가 우리 코드에 끼치는 영향 및 문제가 파악하기 쉽지 않아 손쉽게 새 버전으로 이전하기 어렵다.

