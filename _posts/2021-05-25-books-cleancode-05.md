---
tags: 
    - clean code
    - 클린 코드
title: 클린 코드(Clean Code) 리뷰 - 05
date: 2021/05/25
author: 김동환
description: 클린 코드(Clean Code) 리뷰 - 5장 형식 맞추기
disabled: true
categories:
  - general
---

> 코드의 형식은 일종의 의사소통이다.

# 세로 형식

## 개념은 빈 행으로 분리하라

일련의 행 묶음은 완결된 생각 하나를 표현한다. 생각 사이는 빈 행을 넣어 분리해야 마땅하다. import 선언문, 패키지, 함수 등 빈 행을 넣어 시각적 정보를 제공해야한다.

```java
package com.querydsl.core.types.dsl;

import java.sql.Time;
import java.util.*;

import com.google.common.collect.ImmutableList;
import com.querydsl.core.Tuple;
import com.querydsl.core.types.*;

public final class Expressions {
		public static final NumberExpression<Integer> ONE = numberTemplate(Integer.class, "1");
		
		public static final NumberExpression<Integer> TWO = numberTemplate(Integer.class, "2");
		
		private Expressions() { }
		
		public static <D> SimpleExpression<D> as(Expression<D> source, Path<D> alias) {
		    if (source == null) {
		        return as(Expressions.<D>nullExpression(), alias);
		    } else {
		        return operation(alias.getType(), Ops.ALIAS, source, alias);
		    }
		}
		
		//...
}
```

## 세로 밀집도

같은 파일에 속할 정도로 밀접한 코드들은 세로 밀집도는 연관성을 의미한다. 즉, 서로 밀접한 코드는 세로로 가까이 두어야한다.

### 로컬 변수

변수는 사용하는 위치에 가까이 선언해야 한다. 해당 변수가 어떻게 초기화되었는지, 어떻게 쓰이는지를 바로 알 수 있다.

```java
private static void failEquals(String message, Object actual) {
		String formatted = "Values should be different. ";
		if (message != null) {
		    formatted = message + ". ";
		}
		
		formatted += "Actual: " + actual;
		fail(formatted);
}
```

### 클래스 필드

클래스 필드(인스턴스 변수)는 클래스 맨 처음에 선언한다.(자바) 물론 일부 C++에선 클래스 마지막에 선언하는 `scissors rule`을 따르긴 한다. 중요한 것은 잘 알려진 위치에 필드들을 모으는 것이다.

```java
public class JPAQueryFactory implements JPQLQueryFactory  {
		
		@Nullable
		private final JPQLTemplates templates;
		
		private final Provider<EntityManager> entityManager;
		
		public JPAQueryFactory(final EntityManager entityManager) {
		    this.entityManager = new Provider<EntityManager>() {
		        @Override
		        public EntityManager get() {
		            return entityManager;
		        }
		    };
		    this.templates = null;
		}
		
		// ...
}
```

### 종속 함수

한 함수가 다른 함수를 호출 한다면 두 함수는 가까이 배치해야한다. 호출 되는 함수보다 호출 하는 함수가 더 먼저 배치한다. 호출하는 함수쪽에서는 호출되는 함수의 이름을 통해 로직을 대략적으로 이해할 수 있으며, 이후 호출되는 함수에서 자세한 로직을 이해할 수 있다.

```java
public class AccountService {

		@Autowired
		private AccountRepository accountRepository;
		
		public AccountDTO.CommonResponse saveAccountIfNotExist(AccountDTO.RegisterRequest registerRequest) {
			if (!hasAccount(registerRequest.getId())) {
				return convertToCommonResponse(accountRepository.save(
					ConvertObject.object2Object(registerRequest, Account.class).passwordEncode(passwordEncoder)));
			}
			return null;
		}
		
		private boolean hasAccount(String id) {
			return accountRepository.existsById(id);
		}
		
		private AccountDTO.CommonResponse convertToCommonResponse(Account account) {
			return ConvertObject.object2Object(account, AccountDTO.CommonResponse.class);
		}
}
```
