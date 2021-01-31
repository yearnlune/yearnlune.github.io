---
tags: 
    - clean code
    - 클린 코드
title: 클린 코드(Clean Code) 리뷰 - 03
date: 2021/01/31
author: 김동환
description: 클린 코드(Clean Code) 리뷰 - 3장 함수
disabled: false
categories:
  - general
---
## 프로그래밍은 글을 쓰는 것과 같다.

프로그래밍은 목적성을 띤 글을 작성하는 것과 같다. 어떠한 행위나 동작을 목표로 한 문장, 한 문단을 작성한다. 우리가 글을 쓸 때도 적절하게 문장을 끊어가며 작성한다. 그 이유로는 여러 이유가 존재하겠지만, 가장 큰 이유는 독자가 쉽게 읽고 이해할 수 있기 때문이다. 이러한 중심주제를 뒷받침하는 문장들이 모여 하나의 중심 주제를 지닌 문단이 된다. 하나의 문단을 작성하는 데 하나의 중심 주제를 지닌 문장으로 이뤄졌다고 생각해보자. 아무래도 적절하게 문장으로 나뉜 글 보다 읽고 이해하는데 어려울 것이다. 프로그래밍도 글을 쓰는 것과 같다고 볼 수 있다. 하나의 함수가 하나의 중심 주제(중심 동작, 행위)를 담당한다면, 중심 주제를 뒷받침하는 문장(동작, 행위)이 많을수록 코드의 줄 수는 몇백 줄, 몇천 줄이 될 것이며, 이를 읽는 동료나 혹은 미래의 자신은 이해하는 데 어려움을 겪을 것이다. 그렇기에 적절하게 함수로 나누어 술술 읽힐 수 있게 해야 한다.

### ◽ 작게 만들어라

함수의 크기를 줄일수록 좋다. 특히 함수의 중첩 구조가 2단을 넘어서면 안 된다.

```java
public static String renderPageWithSetupsAndTeardowns(
		PageData pageData,
		boolean isSuite
	) throws Exception {
		if (isTestPage(pageData)) {
			includeSetupAndTeardownPages(pageData, isSuite);
		}
		return pageData.getHtml();
}
```

### ◽ 한가지만 해라

> 함수는 한 가지를 해야한다. 그 한 가지를 잘 해야 한다. 그 한 가지만을 해야 한다.

여기서 한가지란 추상화 수준이 하나인 단계를 의미한다. 의미 있는 다른 함수로 추출할 수 있다면 그 함수는 한 가지만 하는 것이 아니다.

### ◽ 함수당 추상화 수준은 하나로

함수당 한 가지의 일을 하기 위해서는 추상화 수준이 같아야 한다. 한 함수 내에 추상화 수준이 다르면 헷갈릴 수 있다.

### ◽ 내려가기 규칙

이야기처럼 위에서 아래로 읽혀야 좋다. 그렇기에 한 함수 다음에는 추상화 수준이 한 단계 낮은 함수가 와야 한다.

```java
private void includeTeardownPages() throws Exception {
    includeTeardownPage();
    if (this.isSuite) {
        includeSuiteTeardownPage();
    }
}

private void includeTeardownPage() throws Exception {
    include("TearDown", "-teardown");
}

private void includeSuiteTeardownPage() throws Exception {
    include(SuiteResponder.SUITE_TEARDOWN_NAME, "-teardown");
}

private void include(String pageName, String arg) throws Exception {
    //...
}
```

### ◽ 서술적인 이름을 사용하라

함수 이름을 같은 문구, 명사, 동사를 사용하여 읽기 쉽도록 해야 한다.

```java
void includeSetupAndTeardownPages();
void includeSetupPages();
void includeTeardownPages();
void includeSuiteSetupPage();
```

### ◽ 입력 인수가 적을 수록 좋다

IDE에서 입력 인수에 대한 정보를 제공해 주긴 하지만, 입력 인수가 많을수록 이해하기 어려울 수 있다. 특히 순서도 중요해지는 만큼 더더욱 어려워진다. 입력 인수를 항상 1개 이하로 구현하기 어렵겠지만, 입력 인수가 많은 것은 지양해야 한다.

### ◽ 중복하지마라

코드의 중복을 없애야 한다. 만약 그 코드를 수정해야 한다면 쓰인 N만큼 수정을 해야 한다.

## 마치며...

> 길이가 짧고, 이름이 좋으며, 추상화 정도가 일정한 함수를 구현하라

함수를 깨끗하게 짜는 방법에 관해서 얘기해보았다. 프로그래밍에서 많은 부분을 담당하는 것은 함수 구현 부분이라 할 수 있다. 위에서 살펴본 가이드들을 모두 만족하게 함수 구현을 하기엔 어려울 수 있지만, 유닛테스트와 함께 차근차근 고쳐나가야겠다.