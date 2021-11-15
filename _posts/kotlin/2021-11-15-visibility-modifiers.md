---
tags: 
    - kotlin
    - visibility modifiers
    - 가시성 제어자
    - access modifier
    - 접근 제어자
title: kotlin - Visibility modifiers
date: 2021/11/15
author: 김동환
description: 가시성 제어자 (private, protected, internal, public)
disabled: true
categories:
  - kotlin
---


# **Visibility modifiers**

Java의 접근 제어자(Access Modifier)와 유사하게 Kotlin에서 사용되는 제어자이다.

Kotlin에서는 `private` , `protected` , `internal` , `public` 총 4가지의 가시성 제어자를 사용한다.

## private

자바와 동일하게 해당 패키지, 클래스 등 선언한 범위에서만 사용가능하다.

```kotlin
private class A {
    private val hello: String = "Hello"
}

```