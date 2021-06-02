---
tags: 
    - clean code
    - 클린 코드
    - 객체 지향
    - 절차 지향
title: 클린 코드(Clean Code) 리뷰 - 06
date: 2021/05/31
author: 김동환
description: 클린 코드(Clean Code) 리뷰 - 6장 객체와 자료구조
disabled: true
categories:
  - general
---

> 요새들어 함수형 프로그래밍이 떠오르고 있지만, 많은 부분이 객체 지향 프로그래밍이 사용되고 있다.

# 객체

## 자료 추상화

추상화 인터페이스를 제공하며, 실질적인 구현은 감춰야 한다. 사용자가 구현을 모른 채 자료의 핵심을 조작할 수 있어야 진정한 의미의 클래스라고 할 수 있다.

```java
// #1 구체적
public class Point {
	public double x;
	public double y;
}

// #2 추상적
public interface Point {
	double getX();
	double getY();
	void setCartesian(double x, double y);
	double getR();
	double getTheta();
	void setPolar(double r, double theta);
}
```

`#1` 해당 필드의 접근자를 `private` 으로 선언하고, `getter` , `setter` 를 제공한다해도 구현을 외부로 노출하는 셈이다. 계층을 통해서 완벽하게 감출 수 없으며, 추상화를 통해서만 가능하다. 그렇다면 단순히 `#2` 처럼 `get` 와 `set`를 제공하는 인터페이스가 좋은 추상화라고 생각 할 수 있을까? 다음을 살펴보자.

```java
// #3 조회
public interface Vehicle {
	double getFuelTankCapacityInGallons();
	double getGallonsOfGasline();
}

// #4 추상적 개념
public interface Vehicle {
	double getPercentFuelRemaining();
}
```

`#3` 은 총량과 남은양을 가져올 수 있다. 하지만 데이터를 세세하게 공개하게 된다. `#4` 처럼 더욱 추상적인 개념을 통해서 표현하는 것이 더 좋은 방법이다.

## 절차 지향 / 객체 지향

우리 모두 객체 지향적으로 프로그래밍을 하려고 노력하고 있다. 하지만, 아직도 객체 지향 프로그래밍 언어를 가지고 절차 지향적으로 프로그래밍을 하는 경우가 종종 있다. 그렇다면 항상 절차 지향적인 프로그래밍은 나쁜 방법일까? 간단한 예를 살펴보자.

```java
// #1 절차 지향적 프로그래밍
public class Square {
	public Point topLeft;
	public double side;
}

public class Rectangle {
	public Point topLeft;
	public double height;
	public double width;
}

public class Geometry {
	public double area(Object shape) throws NoSuchShapeException {
		if (shape instanceof Square) {
			Square s = (Square)shape;
			return s.side * s.side;
		} else if {
			Rectangle r = (Rectangle)shape;
			return r.height* r.width;
		}
	} else {
		throws new NoSuchShapeException();
	}
}
```

```java
// #2 객체 지향적 프로그래밍
public class Square implements Shape {
	private Point topLeft;
	private double side;
	
	public double area() {
		return side * side;
	}
}

public class Rectangle implements Shape {
	private Point topLeft;
	private double height;
	private double width;

	public double area() {
		return height* width;
	}
}
```

위의 `#1` 절차 지향적 프로그래밍과 `#2` 객체 지향적 프로그래밍의 예제를 살펴보자.

### 새로운 클래스

`#1`의 경우 새로운 도형이 생긴다면, 새로운 도형 클래스를 생성하고, 기존 함수 `Geometry.area(Object shape)` 수정이 불가피하다. `#2` 의 경우 새로운 클래스를 생성하고 `area()` 를 구현해주면 되며, 각 클래스의 기존 함수는 영향을 받지 않는다.

### 새로운 함수

`#1`의 경우 새로운 함수가 생긴다면, 새로운 `Geometry` 클래스에 새로운 함수를 구현해주면 된다. `#2`의 경우 새로운 함수가 생긴다면, `Shape`를 구현하는 모든 클래스에 새로운 함수를 구현해줘야 한다.

### 결론

위 처럼 절차 지향적인 코드, 객체 지향적인 코드를 살펴보았다. 일반 적으로 객체 지향 코드에서 변경이 어려운 것은 절차 지향 코드에선 쉬우며, 절차 지향 코드에서 변경이 어려운 것은 객체 지향 코드에서 쉽다.  때로는 절차적인 코드가 적합할 경우도 존재한다. 모든 절차적인 코드가 좋지 않다고 단정지을 수는 없다.

## 디미터 법칙

> 모듈은 자신이 조작하는 객체의 속사정을 몰라야한다. 즉, 객체는 자료를 숨기고 함수로 공개해야한다.

### 법칙

클래스 C의 메서드 f에 대해서 다음과 같은 호출 법칙을 따라야한다.

1. C 클래스 내에서의 `f()` 호출
2. 메소드 내에서의 생성한 c 객체의 `f()` 호출
3. 인수로 넘어온 c 객체의 `f()` 호출
4. D 클래스 필드에 선언된 c 객체의 `f()` 호출

```java
public class C {
	public void f() {}
	public void example() { 
		// 1. C 클래스 내에서의 f() 호출
		f(); 
	}
}

public class D {
	C fieldC;

	public void example(C paramC) {
		// 2. g() 내에서의 생성한 c 객체의 f() 호출
		C localC = new C();
		localC.f();

		// 3. 인수로 넘어온 c 객체의 f() 호출 
		paramC.f();

		// 4. D 클래스 필드에 선언된 c 객체의 f() 호출
		fieldC.f();
	}
}
```

### 기차 충돌

함수의 호출이 또다른 객체를 리턴하고 이를 또 함수를 호출하는 코드들을 말한다. 이러한 코드들은 지양해야한다.

```java
// BAD CASE | Train Wreck
String outputDir = ctxt.getOptions().getScratchDir().getAbsolutePath();

// GOOD CASE
Options opts = ctxt.getOptions();
File scratchDir = opts.getScratchDir();
String outputDir = scratchDir.getAbsolutePath();
```

### 기차 충돌과 메서드 체이닝

빌더 패턴에서 쓰이는 메서드 체이닝과 유사하게 보일 수 있다. 계속적으로 메서드를 호출한다. 하지만 근본적으로 다른 점이 있다. 메서드 체이닝을 구현할 때 우리는 `this` 를 리턴해준다.

**즉 메서드 체이닝은 하나의 객체에서 자기 자신을 리턴함으로써 자신의 메서드를 호출하는 것이다. 위에서 알아본 기차 충돌은 자기 자신이 아닌 다른 객체의 호출을 연결한 것이다.**