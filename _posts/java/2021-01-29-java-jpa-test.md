---
tags: 
    - java
    - test
    - junit
    - jpa
    - springBoot
title: Java JPA Test 
date: 2021/01/29
author: 김동환
description:  자바 JPA 테스트 
disabled: false
categories:
  - java
---

> JPA와 관련된 테스트

- junit:4.12
- spring-boot:2.4.2

## 기본구조

```java
/**
 * IN-MOMORY
 */
@RunWith(SpringRunner.class)
@DataJpaTest
public abstract class JpaTestSupport {
}
```

```java
/** CASE #1
 * `test` profile에 정의된 실제 RDB 사용
 */
@RunWith(SpringRunner.class)
@DataJpaTest
@ActiveProfiles("test")
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
public abstract class JpaTestSupport {
}
```

```java
/** CASE #2
 * `test` profile에 정의된 실제 RDB 사용
 */
@RunWith(SpringRunner.class)
@DataJpaTest
@ActiveProfiles("test")
public abstract class JpaTestSupport {
}

////////////////////////application-test.yml///////////////////////////
spring:
  test:
    database:
      replace: none
////////////////////////////////////////////////////////////////////////
```

`@DataJpaTest` : JPA의 컴포넌트들을 중점적으로 테스트할 수 있으며, 기본적으로 `AutoConfigureTestDatabase` 에서 in-memory H2를 사용한다. `@ActiveProfiles` 을 통해 configuration을 정의하고, case #1처럼 `AutoConfigureTestDatabase.Replace.NONE` 을 통해 기본 H2가 아닌 `test` configuration에 정의된 DB를 사용한다. 또는  case #2처럼 configuration에 정의해도 된다.

## 따라해보기

### Repository

> `JpaRepository`에서 지원해주는 메소드나, CustomRepository를 통해서 작성한 Query들을 테스트 할 수 있다.

```java
public class AccountTest extends JpaTestSupport {

	@Autowired
	AccountRepository accountRepository;
	
	@Before
	public void setUp() {
	  Account accountMock = Account.builder()
	     .id("mock")
	     .name("nameMock")
	     .password("mock")
	     .role(AccountRoleType.GUEST)
	     .build();
	
	  accountRepository.save(accountMock);
	}

	@Test
	public void findById_existentAccountId_shouldBeSuccess() {
		/* GIVEN */
		String id = "mock";
	
		/* WHEN */
		Optional<Account> accountOptional = accountRepository.findById(id);
		Account account = accountOptional.orElse(null);
	
		/* THEN */
		assertThat(account, notNullValue());
		assertThat(account.getId(), is(id));
	}
}
```

### Criteria

> `EntityManager` 를 주입시키고 테스트 Criteria를 테스트 할 수 있다.

```java
public class AccountTest extends JpaTestSupport {
	
	@Autowired
	EntityManager entityManager;

	@Before
	public void setUp() {
	  Account accountMock = Account.builder()
	     .id("mock")
	     .name("nameMock")
	     .password("mock")
	     .role(AccountRoleType.GUEST)
	     .build();
	
	  accountRepository.save(accountMock);
	}

	@Test
	public void findById_existentAccountId_shouldBeSuccess() {
		/* GIVEN */
		String id = "mock";

		/* WHEN */
		CriteriaBuilder criteriaBuilder = entityManager.getCriteriaBuilder();
		CriteriaQuery<Account> accountQuery = criteriaBuilder.createQuery(Account.class);
		Root<Account> accountRoot = accountQuery.from(Account.class);

		accountQuery
			.select(accountRoot)
			.where(
				criteriaBuilder.equal(accountRoot.get("id"), id)
			);

		Account account = entityManager.createQuery(accountQuery).getSingleResult();

		/* THEN */
		assertThat(account, notNullValue());
		assertThat(account.getId(), is(id));
	}
}
```

### QueryDSL

> `JPAQueryFactory`를 주입시키고 QueryDSL를 테스트 할 수 있다.

```java
public class AccountTest extends JpaTestSupport {
	
	@Autowired
	EntityManager entityManager;

	JPAQueryFactory jpaQueryFactory;
	
	@Before
	public void setUp() {
		queryFactory = new JPAQueryFactory(entityManager);

	  Account accountMock = Account.builder()
	     .id("mock")
	     .name("nameMock")
	     .password("mock")
	     .role(AccountRoleType.GUEST)
	     .build();
	
	  accountRepository.save(accountMock);
	}

	@Test
	public void findById_existentAccountId_shouldBeSuccess() {
		/* GIVEN */
		String id = "mock";

		/* WHEN */
		QAccount qAccount = QAccount.account;
		Account account = jpaQueryFactory.from(qAccount)
			.select(qAccount)
			.where(qAccount.id.eq(id))
			.fetchFirst();

		/* THEN */
		assertThat(account, notNullValue());
		assertThat(account.getId(), is(id));
	}
}
```