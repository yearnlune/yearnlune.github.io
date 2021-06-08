---
tags: 
    - java
    - exception
    - test
    - junit
    - private
    - method
title: Java private method test
date: 2021/06/08
author: 김동환
description: 자바 private method 테스트
disabled: false
categories:
  - java
---

　
# Private Method

> 클래스내 Private method에 대한 테스트가 필요한 경우가 있다. 하지만 해당 Method를 호출할 수 없어 일반적으로 테스트가 불가능하다. 이러한 경우 어떤 방식으로 테스트를 해야하는지 알아보도록 한다.

```java
// 예시 Class
public class Calculator {

	public int findMaxValue(int[] numbers) {
		int maxValue = Integer.MIN_VALUE;

		for (int i = 0; i < numbers.length; i++) {
			maxValue = max(maxValue, numbers[i]);
		}

		return maxValue;
	}

	private int max(int a, int b) {
		return a > b ? a : b;
	}
}
```

　
## 기본 구조

### Do not test

Private method는 따로 테스트를 진행하지 않는다. 해당 Private Method는 다른 접근가능 메소드에서 활용된다. 그렇기에 다른 접근 가능 메소드를 테스트하여 처리한다. JUnit의 창시자라 할 수 있는 `Kent Beck`은 [http://shoulditestprivatemethods.com](http://shoulditestprivatemethods.com/)를 트윗하기도 하였다.

```java
public class CalculatorTest {

	Calculator calculator = new Calculator();

	@Test
	public void findMaxValue_sourceGreaterThanTarget_ShouldBeReturnedMaxValue() throws Exception {
		/* GIVEN */
		int[] numbers = {5, 10};

		/* WHEN */
		int maxValue = calculator.findMaxValue(numbers);

		/* THEN */
		assertThat(maxValue, is(10));
	}
}
```

　
### Java Reflection

리플렉션을 활용하여 각종 클래스, 필드, 메소드 등을 찾아 임시적으로 `Private`을 접근 가능하게 하여 테스트를 진행한다.

```java

public class CalculatorTest {

	Calculator calculator = new Calculator();

	@Test
	public void max_sourceGreaterThanTarget_ShouldBeReturnedSource() throws Exception {
		/* REFLECTION */
		Method maxMethod = Calculator.class.getDeclaredMethod("max", int.class, int.class);
		maxMethod.setAccessible(true);

		/* GIVEN */
		int source = 10;
		int target = 5;

		/* WHEN */
		int maxValue = (int)maxMethod.invoke(calculator, source, target);

		/* THEN */
		assertThat(maxValue, is(source));
	}
}
```

　
### ReflectionTestUtils

Spring Framework에서는 `ReflectionTestUtils`를 지원해준다.

```java
public class CalculatorTest {

	Calculator calculator = new Calculator();

	@Test
	public void max_sourceGreaterThanTarget_ShouldBeReturnedSource() throws Exception {
		/* GIVEN */
		int source = 10;
		int target = 5;

		/* WHEN */
		int maxValue = ReflectionTestUtils.invokeMethod(calculator, "max", source, target);

		/* THEN */
		assertThat(maxValue, is(source));
	}
}
```

　
### @VisibleForTesting

Google에서 제공하는 자바 공통 라이브러리인 `guava`에서는 `@VisibleForTesting`을 지원해준다. 위에서 살펴본 방식과는 달리 테스트를 위해서 접근 가능한 메소드로 설정하고, 실제 Production일 땐 `Private` 형태로 제공된다.

```java
@VisibleForTesting
int max(int a, int b) {
	return a > b ? a : b;
}
```

```java
public class CalculatorTest {

	Calculator calculator = new Calculator();

	@Test
	public void max_sourceGreaterThanTarget_ShouldBeReturnedSource() throws Exception {
		/* GIVEN */
		int source = 10;
		int target = 5;

		/* WHEN */
		int maxValue = calculator.max(source, target);

		/* THEN */
		assertThat(maxValue, is(source));
	}
}
```