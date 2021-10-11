---
tags: 
    - java
    - test
    - junit
    - MongoDB
    - springBoot
title: Java MongoDB Test 
date: 2021/01/20
author: 김동환
description:  자바 MongoDB 테스트 
disabled: false
categories:
  - java
---

# MongoDB

> MongoDB와 관련된 Query, Repository를 테스트

- junit:4.12
- spring-boot:2.4.2

## 기본 구조

```java
/**
 * IN-MOMORY
 */
@RunWith(SpringRunner.class)
@DataMongoTest
public abstract class MongoDbTestSupport {
}
```

```java
/**
 * `test` profile에 정의된 실제 MongoDB 사용
 */
@RunWith(SpringRunner.class)
@ActiveProfiles("test")
@DataMongoTest
public abstract class MongoDbTestSupport {
}
```

`@DataMongoTest` : MongoDB의 컴포넌트들을 중점적으로 테스트할 수 있으며, 기본적으로 in-memory DB를 사용한다. `@ActiveProfiles`와 같이 Configuration을 정의하면 실제 MongoDB 테스트에 연관된 정보를 활용한다.

## 따라해보기

### Repository

> `MongoRepository` 에서 지원해주는 메소드나, CustomRepository를 통해서 작성한 Query들을 테스트 할 수 있다.

```java
public class AccountTest extends MongoDbTestSupport {

	@Autowired
	AccountRepository accountRepository;

	Account mockAccount;

	String mockAccountId;

	@Before
	public void setUp() {
		mockAccount = accountRepository.save(Account.builder().build());

		mockAccountId = mockAccount.getId();
	}

	@Test
	public void findById_existentAccountId_shouldBeSuccess() {
		/* GIVEN */
		String mockExistentAccountId = mockAccountId;

		/* WHEN */
		Optional<Account> accountOptional = accountRepository.findById(mockExistentAccountId);

		/* THEN */
		assertThat(accountOptional.orElse(null), is(mockAccount));
	}
}
```

### MongoTemplate

> MongoTemplate을 의존성 주입받아 Criteria query를 작성하고, 이를 테스트 할 수 있다.

```java
public class AccountTest extends MongoDbTestSupport {

	@Autowired
	MongoTemplate mongoTemplate;

	Account mockAccount;

	String mockAccountId;

	@Before
	public void setUp() {
		mockAccount = accountRepository.save(Account.builder().build());

		mockAccountId = mockAccount.getId();
	}

	@Test
	public void findById_existentAccountId_shouldBeSuccess() {
		/* GIVEN */
		String mockExistentAccountId = mockAccountId;

		/* WHEN */
		Query query = Query.query(
			Criteria.where("_id").is(mockExistentAccountId)
		);
		
		Account account = mongoTemplate.findOne(query, Account.class);
		
		/* THEN */
		assertThat(account, notNullValue());
		assertThat(account, is(mockAccount));
	}
}
```