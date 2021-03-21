---
tags: 
    - clean code
    - 클린 코드
title: 클린 코드(Clean Code) 리뷰 - 04
date: 2021/03/21
author: 김동환
description: 클린 코드(Clean Code) 리뷰 - 4장 주석
disabled: false
categories:
  - general
---
　
# 주석보단 코드로 말하자

> 나쁜 코드에 주석을 달지 마라. 새로 짜라
- 브라이언 W. 커니핸, P.J. 플라우거

코드로만 의도를 표현하기 힘들 때 주석을 사용하곤 한다. 몇몇 주석은 훗날 코드가 바뀌었음에도 해당 주석을 유지 보수하지 않는 경우가 있다. 부정확할 수도 있는 주석은 없는 것이 훨씬 나을 수 있다. 주석이 어찌 됐던 코드만이 정확한 정보를 제공해준다. 깨끗한 코드 작성으로 주석 없이도 명확한 의미를 전달하는 것이 최고의 방법이다.
　
## 주석은 나쁜 코드를 보완하지 못한다

주석을 일반적으로 추가하는 경우는 해당 코드의 품질이 나쁘기 때문이다. 한마디로 알아먹기가 힘들어 주석을 다는 경우이다. 하지만 이 생각이 들때 코드를 정리를 해야한다.

　
## 코드로 의도를 표현하라

대부분의 의도는 코드로 표현 가능하다. 아래처럼 주석으로 표현하려는 것을 함수를 만들어 충분히 표현 가능하다.

```java
// 복지 혜택을 받을 자격 있는 직원 여부 확인
if ((employee.flags & HOURLY_FLAG) &&
    (employee.age > 65)) {}

if (employee.isEligibleForFullBenefits()) {}
```

　
## 좋은 주석

주석의 유지보수 측면에서는 주석이 주는 위험성을 지니고 있지만, 적절하게 주석을 통해서 의미를 명료하게 전달하는 것은 좋은 방법이다.

### 법적인 주석

```java
/** 
  * Copyright (c) 1994, 2010, Oracle and/or its affiliates. All rights reserved.
  * ORACLE PROPRIETARY/CONFIDENTIAL. Use is subject to license terms.
  */
```

　
### 의도를 설명하는 주석

```java
//스레드를 대량 생산하여 경쟁 상태를 만들기 위함
for (int i = 0; i < 10000; ++i) {
	WidgetBuilderThread widgetBuilderThread = new WidgetBuilderThread();
	Thread thread = new Thread(widgetBuilderThread);
	thread.start();
}
```

　
### TODO 주석

앞으로 해야할 일들을 표시하는 주석이다. 물론 이 주석들은 훗날 코드로 해결되어 없어질 주석들이다.

```java
public <T> String solution(String solutionName, List<T> paramList) {
	String output = "";
	SolutionMeta solutionMeta = solutionFactory.get(solutionName);
	
	try {
		Method method = solutionMeta.getSolutionMethod();
		// TODO param mapping 작성 필요
	
		output = toString(method.invoke(paramList));
	} catch (IllegalAccessException | InvocationTargetException e) {
		e.printStackTrace();
	}
	return output;
}
```

## 나쁜 주석

대부분의 주석은 여기에 속한다. 해당 주석들을 지우고 깨끗한 코드를 작성을 해야한다.

　
### 같은 이야기를 중복하는 주석

```java
/**
  * 이 컴포넌트의 프로세서 지연 값
  */
protected int backgroundProcessorDelay = -1;

/**
  * 컨테이너와 관련된 Manager 구현
  */
protected Manager manager = null;

/**
  * 컨테이너와 관련된 Cluster 구현
  */
protected Cluster cluster = null;
```

　
### 이력을 기록하는 주석

예전에는 해당 이력을 남겨 관리하는게 관례였으나, Git과 같은 소스 코드 관리 시스템의 등장으로 불필요해졌다.

```java
/**
  *  == 개정이력(Modification Information) ==
  *
  *   수정일            수정자        수정내용
  *  ------------   --------    ---------------------------
  *   2020. 11. 30.    김동환        최초 생성
  *   2021. 3. 16.     김동환        관리자 API 추가
  */
```

　
### 있으나 마나 한 주석

```java
public class ConvertObject {
	
	/**
	  * 기본 생성자
	  */
	public ConvertObject() {
	}
}
```

　
### 주석으로 처리한 코드

소스 코드 관리 시스템의 등장으로 주석으로 처리된 코드는 과감하게 지워야한다.

```java
public <T> String toString(T t) {
    StringBuilder stringBuilder = new StringBuilder();
    Class tClass = t.getClass();

    if (tClass.isPrimitive()) {
        stringBuilder.append(t);
    } else if (tClass.isArray()) {
        if (t instanceof Object[]) {
            stringBuilder.append(Arrays.deepToString((Object[])t));
        } else {
            int tLength = Array.getLength(t);
            Object[] output = new Object[tLength];

            for (int i = 0; i < tLength; ++i) {
                output[i] = Array.get(t, i);
            }

            stringBuilder.append(Arrays.toString(output));
        }
    } else {
//      System.out.println("OUTPUT TYPE: " + tClass.getSimpleName());
        stringBuilder.append(t.toString());
    }

    return stringBuilder.toString();
}

```

　
# 마치며...

지금까지도 많은 나쁜 주석을 달아왔다. 이해하기 난해한 코드를 이해시키기 위해 간편하게 주석을 달곤 했다. 하지만 해당 소스를 변경하거나 추가해야 할 때 주석에 의지한 채 어렵게 유지 보수할 수밖에 없었다. 위에서 제시해준 여러 가이드라인을 참고하여 여러 주석을 삭제하고 함수나 변수로 표현하려고 노력해야겠다.