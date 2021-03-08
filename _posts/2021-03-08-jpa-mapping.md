---
tags: 
    - java
    - jpa
    - mapping
title: JPA Mapping
date: 2021/03/08
author: 김동환
description: JPA Mapping 작성중
disabled: false
categories:
  - java
---
　
# @Entitiy

테이블과 클래스를 매핑하기 위한 어노테이션으로 JPA에서 사용하기 위해선 필수다.

JPA에서 사용할 이름을 정할 수 있으며, 기본적으로는 클래스이름을 사용한다.

```java
@Entity
public class User {
	//...
}
```

- 기본 생성자는 필수다.
- final 클래스, enum, interface, inner 클래스에는 사용할 수 없다.
- 필드에 final을 사용할 수 없다.

## **Attribute**

`name` : JPA에서 사용할 이름 정의 (기본은 클래스 이름)

　
# @Table

테이블과 엔티티를 매핑하기 위한 어노테이션으로 @Table을 생략하면 @Entity에서 사용한 이름을 테이블 이름으로 사용한다.

```java
@Entity
@Table(name = "USER",
	indexes = {
		@Index(name = "idx_user_name", columnList = "name", unique = true)
	})
public class User {
	//...
	private String name;
	//...
}
```

## Attribute

`name` : 매핑할 테이블 이름 정의 (기본은 Entity에서 사용한 이름)

`catalog` : DB catalog 매핑

`schema` : DB schema 매핑

`uniqueConstraints` : DDL 생성 시 제약조건을 생성한다. 2개 이상의 복합키에 대해서 제약조건을 설정 할 수 있다.

`indexes` : @Index를 통해 인덱스를 생성할 수 있으며, unique 인덱스를 생성하려면 `unique = true` 해주어야 한다.

　
# @Id

테이블의 기본 키(Primary Key) 매핑을 해주며, `@GeneratedValue`를 통해서 여러 매핑 전략을 사용할 수 있다.

## 기본 키 매핑 전략

- 직접 할당 전략
- IDENTITY 전략
- SEQUENCE 전략
- TABLE 전략
- AUTO 전략

### 직접 할당 전략

`@Id` 로 할당된 필드를 직접 기본 값을 할당한다.

```java
@Entity
@Table(name = "USER")
@Setter
public class User {
	@Id
	private String id;

	//...
}

/*
	User user = new User();
	user.setId("ID_0001");
*/
```

### IDENTITY 전략

데이터베이스에게 기본 키 생성을 위임하는 전략으로, 주로 MySQL, SQL Server, PostgreSQL에서 사용한다.

```java
@Entity
@Table(name = "USER")
public class User {
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Integer id;

	//...
}
```

### SEQUENCE 전략

기본 키를 유일한 값을 Sequence를 통해 순서대로 생성하는 전략으로, Oracle, PostgreSQL에서 사용한다.

```java
@Entity
@Table(name = "USER")
@SequenceGenerator(
	name = "SEQUENCE_USER_ID",
	sequenceName = "SEQ_USER_ID",
	initialValue = 1,
	allocationSize = 50)
public class User {
	@Id
	@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "SEQUENCE_USER_ID")
	private Long id;

	//...
}
```



**@SequenceGenerator**
- name: SequenceGenerator의 이름
- sequenceName: DB에 등록되어 있는 Sequence 이름
- initialValue: 시퀀스 최초 숫자로 기본은 1이다.
- allocationSize: 시퀀스 호출 시 증가하는 수로 시퀀스를 일일히 DB에서 호출해서 사용하기엔 효율적이지 않아 일정 수만큼 시퀀스를 미리 증가시키고 메모리에서 해당 시퀀스만큼을 활용한다.  기본은 50이다.