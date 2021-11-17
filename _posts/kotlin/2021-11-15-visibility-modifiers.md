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
disabled: false
categories:
  - kotlin
---


# **Visibility modifiers**

Java의 접근 제어자(Access Modifier)와 유사하게 Kotlin에서 사용되는 제어자이다.

Kotlin에서는 `private` , `protected` , `internal` , `public` 총 4가지의 가시성 제어자를 사용한다.

## private

해당 private를 선언한 범위에서만 사용가능하다. 클래스 내에서 사용하였다면 클래스 내에서만 사용가능하며, 파일 내에서 선언 하였다면 해당 파일 내에서만 사용 가능하다.

```kotlin
private const val privateVariable = 0;

class ExampleClass {

    private val privateField: Int = 0;
}

fun print() {
    println(privateVariable)    // privateVariable is visible

    val exampleClass: ExampleClass = ExampleClass()
    println(exampleClass.privateField)  // exampleClass.privateField is not visible
}
```

## protected

해당 protected를 선언한 범위 외에도 이를 상속받는 subclass에서도 사용할 수 있다.

```kotlin
open class Parent {

    protected fun protectedMethod() = print("PROTECTED");
}

class Child : Parent() {
    
    fun test() {
        protectedMethod()
    }
}
```

## internal

해당 internal을 선언한 모듈 내에서 사용할 수 있다.

### 모듈의 범위

- IntelliJ IDEA Module
- Maven, Gradle Project
- A set of files compiled with one invocation of the `<kotlinc>` Ant task.

![module_scope](/assets/images/kotlin/module_scope_example.png)

```kotlin
// ModuleA.ExampleA
class ExampleA {

    internal val internalPrint = print("A")
}

// ModuleA.Print
fun printA() {
    ExampleA().internalPrint;
}

// ModuleB.Print
fun printA() {
    ExampleA().internalPrint;  //Cannot access 'internalPrint': it is internal in 'ExampleA'
}
```

## public

모든 곳에서 사용 가능하며, 가시성 제어자의 default 값이다.

```kotlin
class ExampleClass {

    val publicField: Int = 0;
}

fun print() {
    print(ExampleClass().publicField)
}
```

# 참고문헌

[Kotlin doc](https://kotlinlang.org/docs/visibility-modifiers.html)