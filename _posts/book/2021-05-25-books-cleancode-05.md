---
tags: 
    - clean code
    - 클린 코드
title: 클린 코드(Clean Code) 리뷰 - 05
date: 2021/05/25
author: 김동환
description: 클린 코드(Clean Code) 리뷰 - 5장 형식 맞추기
disabled: false
categories:
  - general
---

> 코드의 형식은 일종의 의사소통이다. 변수의 위치, 들여쓰기 등 다양한 형식 정보를 통해서 우리는 코드의 문맥을 더 쉽게 이해할 수 있다. 또한 팀 규칙으로 설정하여, 팀원이 작성한 코드도 더욱 쉽게 이해할 수 있다.

　
# 세로 형식

　
## 개념은 빈 행으로 분리하라

일련의 행 묶음은 완결된 생각 하나를 표현한다. 생각 사이는 빈행을 넣어 분리해야 마땅하다. import 선언문, 패키지, 함수 등 빈행을 넣어 시각적 정보를 제공해야 한다.

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

같은 파일에 속할 정도로 밀접한 코드들은 세로 밀집도는 연관성을 의미한다. 즉, 서로 밀접한 코드는 세로로 가까이 두어야 한다.

　
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

클래스 필드(인스턴스 변수)는 클래스 맨 처음에 선언한다(자바). 물론 일부 C++에선 클래스 마지막에 선언하는 `scissors rule`을 따르긴 한다. 중요한 것은 잘 알려진 위치에 필드들을 모으는 것이다.

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

한 함수가 다른 함수를 호출한다면 두 함수는 가까이 배치해야 한다. 호출되는 함수보다 호출하는 함수가 더 먼저 배치한다. 호출하는 함수 쪽에서는 호출되는 함수의 이름을 통해 로직을 대략 이해할 수 있으며, 이후 호출되는 함수에서 자세한 로직을 이해할 수 있다.

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

　
### 개념적 유사성

직접적인 종속성 외에도 개념적으로 비슷한 코드는 서로 가까이 배치하는 것이 좋다.

```java
public class JPAQueryFactory implements JPQLQueryFactory  {

    @Override
    public <T> JPAQuery<T> select(Expression<T> expr) {
        return query().select(expr);
    }
    
    @Override
    public JPAQuery<Tuple> select(Expression<?>... exprs) {
        return query().select(exprs);
    }
    
    @Override
    public <T> JPAQuery<T> selectDistinct(Expression<T> expr) {
        return select(expr).distinct();
    }
    
    @Override
    public JPAQuery<Tuple> selectDistinct(Expression<?>... exprs) {
        return select(exprs).distinct();
    }
    
    @Override
    public JPAQuery<Integer> selectOne() {
        return select(Expressions.ONE);
    }
    
    @Override
    public JPAQuery<Integer> selectZero() {
        return select(Expressions.ZERO);
    }
    
    @Override
    public <T> JPAQuery<T> selectFrom(EntityPath<T> from) {
        return select(from).from(from);
    }
}
```

　
# 가로 형식

우리 모두 우측으로 스크롤 하는 것을 원치 않는다. 요즘 IDE에서 알아서 이를 개행한 것처럼 보여주지만, 너무 긴 코드는 지양한다.

　
## 가로 정렬

불필요한 가로정렬은 지양한다. 실질적으로 큰 도움이 되지 않으며, 오히려 잘못된 곳이 강조되어 잘못된 의도를 일으킬 수 있다.

```java
// BAD CASE
public FitNesseExpediter(Socket          s,
                         FitNesseContext context) throws Exception {
    this.context =            context;
    socket =                  s;
    input =                   s.getInputStream();
    output =                  s.getOutputStream();
    requestParsingTimeLimit = 10000;
}

// GOOD CASE
public FitNesseExpediter(Socket s, FitNesseContext context) throws Exception {
    this.context = context;
    socket = s;
    input = s.getInputStream();
    output = s.getOutputStream();
    requestParsingTimeLimit = 10000;
}

```

　
## 들여쓰기

들여쓰기를 통해 코드의 계층구조를 표현한다. 이를 통해 한눈에 구조를 알아볼 수 있다. 일부 짧은 if, while문, 함수에서도 들여쓰기를 선호한다.

```java
@Service
public class AccountService {

    private boolean hasAccount(String id) {
        return accountRepository.existsById(id);
    }
        
    private boolean isCorrectPassword(String loginPassword, String accountPassword) {
        return passwordEncoder.matches(loginPassword, accountPassword);
    }
}
```

　
# 팀 규칙

일반적으로 자기 자신만의 규칙을 정하고 코드를 작성한다. 개인 프로젝트일 경우에는 큰 상관이 없을 수 있지만, 규모가 있는 프로젝트는 모두 팀과 함께 프로젝트를 진행한다. 이에 있어서 서로 다른 규칙으로 코드를 작성한다면 코드를 이해하는데 시간이 더 걸린다. 팀으로 혹은 회사에서 정한 규칙을 토대로 일관성 있는 코드를 작성한다면 어떤 프로젝트의 코드라도 좀 더 이해하기 쉽다. IDE에서 쉽게 규칙을 정하여 공유하여 적용할 수 있으며, 일부 플러그인(checkStyle) 등을 도입하여 적용할 수 있다.

　
# 마치며...

이번 장에선 형식에 대해서 서술하였다. 일부 내용은 이전 장과 일맥상통하는 부분도 있었다. 가장 중요시 생각되었던 것은 팀 규칙이다. Java에서는 현재 [hackday-convention-java](https://github.com/naver/hackday-conventions-java)를 활용하여 적용하고 있으며, JS, TS의 경우 eslint와 prettier 등을 활용하여 팀 규칙으로 활용하고 있다.
