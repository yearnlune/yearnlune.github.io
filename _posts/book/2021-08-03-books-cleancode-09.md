---
tags: 
    - clean code
    - 클린 코드
    - 단위 테스트
    - unit test
    - test
title: 클린 코드(Clean Code) 리뷰 - 09
date: 2021/08/03
author: 김동환
description: 클린 코드(Clean Code) 리뷰 - 9장 유닛 테스트
disabled: false
categories:
  - general
---

　
# 단위 테스트

프로그래머라면 한 번쯤은 들어보았을 단위 테스트(Unit test)는 최소 범위의 테스트이다. 제품이나 시스템의 유지보수, 확장성을 위해서라도 유닛 테스트는 선택이 아닌 필수가 되었다. 하지만 올바르지 않은 테스트 코드는 테스트도 오래 걸릴뿐더러 테스트 코드를 유지하고 보수하는 비용도 많이 부담된다. 또한 테스트 코드를 작성하는데 많은 시간을 할애해야 한다. 그렇기에 깨끗한 테스트 코드 작성이 필요하다.

## 깨끗한 테스트 코드

깨끗한 테스트 코드 작성의 핵심 키워드는 `가독성` 이다. 모든 코드가 가독성이 중요하지만, 테스트 코드에 있어서는 가장 중요하다고 생각된다. 테스트 코드를 보고 어떤 테스트를 하는지 한눈에 알아야 한다.

유닛 테스트의 의도를 흐리지 않고 테스트의 의도가 분명히 드러나야 한다. 일반적으로 테스트는 크게 세 부분으로 나뉜다. 테스트 자료 생성, 테스트 자료 조작, 테스트 결과이다. 이 세 부분을 명확히 나눠 작성한다면 좀 더 가독성을 높일 수 있다.

```groovy
@Test
public void registerAccount_correctAccount_shouldBeCreated() throws Exception {
	/* GIVEN */
	String content = objectMapper.writeValueAsString(
		AccountDTO.RegisterRequest.builder()
			.id("object")
			.name("오브젝트")
			.password("object")
			.build());
	
	/* WHEN */
	mockMvc.perform(
		post(AccountConstant.ACCOUNT)
			.content(content)
			.contentType(MediaType.APPLICATION_JSON)
			.characterEncoding("UTF-8")
			.accept(MediaType.APPLICATION_JSON)
			.with(csrf())
	)
	
		/* THEN */
		.andDo(print())
		.andExpect(content().encoding("UTF-8"))
		.andExpect(status().isCreated());
}
```

## 테스트 당 assert 하나

테스트당 하나의 테스트 결론은 이해하기 쉽고 빠르다. 하지만 이를 지키기엔 어렵다고 생각한다. 그래도 테스트당 assert 문의 개수를 최대한 줄여야 한다고 생각한다. 하지만 하나의 테스트에서는 하나의 개념(기능)만을 테스트 해야 한다. 그렇지 않다면 유닛 테스트라 할 수 없으며, 그 의미가 퇴색된다.

## F.I.R.S.T

깨끗한 테스트 코드 작성을 위한 5가지 규칙이 있다.

### Fast

테스트 코드는 빨리 돌아야 한다. 해당 테스트 코드는 자주 돌려야 하기에 빨리 돌아가야 한다.

### Independent

각 테스트 코드는 서로 의존해선 안 된다. 각 테스트는 독립적으로, 어떤 순서로 실행해도 괜찮아야 한다.

### Repeatable

어떠한 환경에서도 반복 가능해야 한다.



### Self-Validating

테스트는 성공이나 실패로 나타내야 한다.

### Timely

단위 테스트는 테스트하려는 실제 코드를 구현하기 이전에 구현한다. 실제 코드를 구현하고 테스트 코드를 작성한다면 테스트를 할 수 없게 실제 코드를 구현할지도 모른다.

# 결론

TDD, BDD, DDD 등 다양한 테스트 방법론을 활용하고 있다. 이 과정에서 많은 테스트 코드를 작성하고 있고 이를 테스트하고 있다. 깨끗한 테스트 코드 작성으로 가독성을 높여 유지보수성, 재사용성을 보존해야 한다.