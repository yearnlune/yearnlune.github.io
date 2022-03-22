---
tags: 
    - kotlin
    - companion object
    - 동반 객체
title: kotlin - Companion object
date: 2022/03/22
author: 김동환
description: companion object 선언 및 특징
disabled: false
categories:
  - kotlin
---

# Companion Object

> 클래스 내부에 선언하는 object
>

자바에서 static과 같이 코틀린에서 사용 가능하다. 자바에서 실질적 클래스 내에 정적 멤버나 함수로 선언되는 것과 달리 코틀린의 companion object는 클래스 내에 `object`(singleton object)로 정의한다. 자바의 static처럼 클래스를 통한 접근을 지원하는 것처럼 보이지만 실질적으로 object 내 멤버와 함수에 접근한다.

## 선언

```kotlin
class ClassA {

    companion object {  // Default Name 'Companion'
        val companionField = "COMPANION A FIELD"
        fun companionMethod() = println("COMPANION A METHOD")
    }
}

class ClassB {

    companion object Named {
        val companionField = "COMPANION B FIELD"
        fun companionMethod() = println("COMPANION B METHOD")
    }
}
```

## 특징
### 클래스 내 하나

클래스 내 하나의 companion object만 존재할 수 있다.

```kotlin
class ClassA {

    companion object A {
        val companionFieldA = "COMPANION A FIELD"
        fun companionMethodA() = println("COMPANION A METHOD")
    }

    // ERROR: [Only one companion object is allowed per class]
    companion object B {
        val companionFieldB = "COMPANION A FIELD"
        fun companionMethodB() = println("COMPANION A METHOD")
    }
}
```

### 클래스 이름을 통한 접근

companion object의 기본 이름인 `Companion` 이나 **지정한 이름**으로 접근 가능하다. 또한 companion object의 이름을 생략하고 사용할 수 있다.

```kotlin
@Test
fun companionObjectTest() {
		println(ClassA.Companion.companionField)    // "COMPANION A FIELD"
    ClassA.Companion.companionMethod()          // "COMPANION A METHOD"

		println(ClassB.Named.companionField)        // "COMPANION B FIELD"
    ClassB.Named.companionMethod()              // "COMPANION B METHOD"

    println(ClassA.companionField)              // "COMPANION A FIELD"
    ClassA.companionMethod()                    // "COMPANION A METHOD"
}
```

### 상속

객체의 상속이 이뤄지면 companion object의 필드 및 메소드 또한 상속이 이뤄진다. 부모와 자식의 companion object의 이름이 다를 경우 해당 이름으로 직접 접근할 수 있다.

```kotlin
open class ClassC {

    companion object Parent {
        val companionField = "COMPANION C FIELD"
        fun companionMethod() = println("COMPANION C METHOD")
    }

    fun fieldC() = println(companionField)
    fun methodC() = companionMethod()
}

class ClassD: ClassC() {

    companion object {
        val companionField = "COMPANION D FIELD"
    }

    fun fieldD() = println(companionField)
    fun parentFieldD() = println(Parent.companionField)
    fun methodD() = companionMethod()
}

@Test
fun companionTest() {
    val classC = ClassC()
    val classD = ClassD()

    classC.fieldC()          // "COMPANION C FIELD"
    classC.methodC()         // "COMPANION C METHOD"    

    classD.fieldD()          // "COMPANION D FIELD"
    classD.parentFieldD()    // "COMPANION C FIELD"
    classD.methodD()         // "COMPANION C METHOD"
}
```

# 참고문헌

[Object expressions and declarations](https://kotlinlang.org/docs/object-declarations.html#companion-objects)

[Companion Object (1) – 자바의 static과 같은 것인가?](https://www.bsidesoft.com/8187)