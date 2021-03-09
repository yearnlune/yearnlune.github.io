---
tags: 
    - java
    - jpa
    - mapping
    - entity
title: JPA Mapping
date: 2021/03/08
author: 김동환
description: JPA Mapping
disabled: false
categories:
  - java
---
　
# @Entity

테이블과 클래스를 매핑하기 위한 어노테이션으로 JPA에서 사용하기 위해선 필수다.

JPA에서 사용할 이름을 정할 수 있으며, 기본적으로는 클래스이름을 사용한다.

```java
@Entity
public class User {
	//...
}
```

- 기본 생성자는 필수다.
- final 클래스, enum, interface, inner 클래스는 사용할 수 없다.
- 필드에 final을 사용할 수 없다.

## **Attribute**

`name` : JPA에서 사용할 이름 정의 (기본은 클래스 이름)

　
# @Table

테이블과 엔티티를 매핑하기 위한 어노테이션으로 @Table을 생략하면 @Entity에서 사용한 이름을 테이블 이름으로 사용한다.

```java
@Entity
@Table(name = "user",
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

테이블의 기본 키(**Primary Key**) 매핑을 해주며, `@GeneratedValue`를 통해서 여러 매핑 전략을 사용할 수 있다.

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
@Table(name = "user")
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
@Table(name = "user")
public class User {
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long id;

	//...
}
```

### SEQUENCE 전략

기본 키를 유일한 값을 Sequence를 통해 순서대로 생성하는 전략으로, Oracle, PostgreSQL에서 사용한다.

```java
@Entity
@Table(name = "user")
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

`@SequenceGenerator`

- name: SequenceGenerator의 이름
- sequenceName: DB에 등록되어 있는 Sequence 이름
- initialValue: 시퀀스 최초 숫자로 기본은 `1`이다.
- allocationSize: 시퀀스 호출 시 증가하는 수로 시퀀스를 일일히 DB에서 호출해서 사용하기엔 효율적이지 않아 일정 수만큼 시퀀스를 미리 증가시키고 메모리에서 해당 시퀀스만큼을 활용한다.  기본은 `50`이다.

### Table 전략

기본 키 생성 전용 테이블을 통해 이를 통해 기본 키를 생성한다.

```java
@Entity
@Table(name = "user")
@TableGenerator(
	name = "SEQUENCE_GENERATOR",
	table = "SEQUENCER",
	pkColumnValue = "SEQUENCE_USER_ID",
	allocationSize = 50
)
public class User {
	@Id
	@GeneratedValue(strategy = GenerationType.TABLE, generator = "SEQUENCE_GENERATOR")
	private Long id;

	//...
}
```

![/assets/images/jpa/sequencer_table.png](/assets/images/jpa/sequencer_table.png)

`@TableGenerator`

- name: TableGenerator의 이름
- table: 기본 키 생성 테이블 이름
- pkColumnValue: 기본 키로 사용할 값의 이름으로 기본 값은 Entity의 이름이다.
- pkColumnName: 시퀀스 컬럼 이름으로 기본 값은 `sequence_name`이다.
- valueColumnName: 시퀀스 값의 컬럼 이름으로 기본 값은 `next_val` 이다
- initialValue: 시퀀스 최초 숫자로 기본은 `0`이다.
- allocationSize: 시퀀스 호출 시 증가하는 수로 일일히 DB에서 호출해서 사용하기엔 효율적이지 않아 일정 수만큼 시퀀스를 미리 증가시키고 메모리에서 해당 시퀀스만큼을 활용한다. 기본은 `50`이다.

### Auto 전략

데이터베이스에 따라 위에 언급한 전략 중하나로 자동 선택한다. 오라클은 SEQUENCE 전략을 MySQL은 IDENTITY 전략을 사용한다.

```java
@Entity
@Table(name = "user")
public class User {
	@Id
	@GeneratedValue(strategy = GenerationType.AUTO)
	private Long id;

	//...
}
```

　
# @Column

클래스의 필드와 테이블의 컬럼을 매핑한다. @Column을 생략할 경우에는 필드의 Primitive Type은 `not null` Object Type은 `null`이 적용된다.

```java
@Entity
@Table(name = "account")
public class Account {
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long idx;

	@Column(length = 64, nullable = false, unique = true)
	private String id;

	@Column(length = 128, nullable = false)
	private String name;

	@Column(columnDefinition = "varchar(64) DEFAULT 'ROLE_GUEST'", nullable = false)
	private String role;

	@Column(name ="created_at", updatable = false, nullable = false)
	private Timestamp createdAt;
}

/*
Hibernate: 

 create table `account` (
       `idx` bigint not null auto_increment,
        `created_at` datetime not null,
        `id` varchar(64) not null,
        `name` varchar(128) not null,
        `role` varchar(64) DEFAULT 'ROLE_GUEST' not null,
        primary key (`idx`)
    ) engine=InnoDB

    alter table `account` 
       add constraint UK_c2x33e7jys0m6l9h2cbc0il1a unique (`id`)
*/
```

## Attribute

`name` : 클래스 필드와 매핑할 컬럼 이름

`insertable` : 엔티티 저장 시 해당 필드도 저장한다. 기본 값은 `true` 이며, readonly로 사용할 경우 false로 설정한다.

`updatable` : 엔티티 수정 시 해당 필드도 수정한다. 기본 값은 `true` 이며, readonly로 사용할 경우 false로 설정한다.

`table` : 하나의 엔티티로 두개 이상의 테이블에 매핑을 할 경우, 다른 테이블에 매핑을 하기 위해 사용된다.

`nullable` : null 값의 허용을 설정한다. 기본 값은 `true` 이며, DDL 생성 시 `NULL` 관련 제약조건이 생성된다.

`unique` : 유니크 제약조건을 설정한다. 기본 값은 `false` 이다. 단일 컬럼에 사용 가능하며, 여러 컬럼에 유니크 제약조건을 설정하기 위해서는 `@Table.uniqueConstaints`나 `@Table.indexes` 를 활용해야한다.

`columnDefinition` : 데이터베이스의 컬럼 정보를 직접 줄 수 있다.

`length` : `String` 타입에서 사용하며, 문자의 길이를 설정할 수 있다.

`precision` : `BigDecimal` 타입에서 사용하며, 소수점을 포함한 전체 자릿수

`scale` : `BigDecimal` 타입에서 사용하며, 소수의 자릿수

　
# @Enumerated

자바의 `Enum` 타입과 매핑할 때 사용된다. `EnumType.ORDINAL` 와 `EnumType.STRING`

가 있으며 기본 값은 `EnumType.ORDINAL` 이다.

```java
@Getter
@AllArgsConstructor
public enum AccountRoleType {
	GUEST("ROLE_GUEST"),
	USER("ROLE_USER"),
	ADMIN("ROLE_ADMIN");

	private final String value;
}
```

```java
@Entity
@Table(name = "account")
public class Account {
	//...

	@Enumerated(EnumType.STRING)
	@Column(length = 64, nullable = false)
	private AccountRoleType role;
	
	//...
}

/*
값이 AccountRoleType.USER일 경우

CASE #1 EnumType.ORDINAL

| role |
|  1   |

------------------------- 

CASE #2 EnumType.STRING

| role |
| USER |

*/
```

## EnumType.ORDINAL

enum의 순서를 데이터베이스에 저장한다. 저장된 데이터의 크기가 작지만, enum의 순서를 변경할 수 없다.

## EnumType.STRING

enum의 이름을 데이터베이스에 저장한다. ORDINAL에 비해 데이터의 크기가 크지만, enum의 순서를 변경하여도 무방하다.

　
# @Temporal

날짜 타입(`java.util.Date` , `java.util.Calendar`)을 매핑할 때 사용한다. 생략할 경우 데이터베이스에서 자바의 Date타입과 비슷한 타입으로 매핑된다. MySQL의 경우는 `datetime`, Oracle, PostreSQL의 경우 `timestamp` 로 생성된다.

```java
@Entity
@Table(name = "account")
public class Account {
	//...

	@Temporal(TemporalType.DATE)
	private Date updatedAt;
	
	//...
}

```

## TemporalType.DATE

날짜 정보만을 취급하는 타입으로 MySQL의 경우 `date` 타입으로 생성되며, 값은 `2021-03-01` 형식으로 저장된다.

## TemporalType.TIME

시간 정보만을 취급하는 타입으로 MySQL의 경우 `time` 타입으로 생성되며, 값은 `15:30:01` 형식으로 저장된다.

## TemporalType.TIMESTAMP

날짜정보와 시간정보 모드 취급하는 타입으로 MySQL의 경우 `datetime` 타입으로 생성되며, 값은 `2021-03-09 15:30:01` 형식으로 저장된다.

　
# @Lob

데이터베이스의 BLOB, CLOB타입과 매핑할 때 사용된다. byte[], `java.sql.BLOB` 의 경우 BLOB으로 매핑되며, String, char[], java.sql.CLOB의 경우 CLOB으로 매핑된다.

```java
@Entity
@Table(name = "account")
public class Account {
	//...

	@Lob
	private byte[] data;
	
	//...
}
```

　
# @Transient

해당 필드는 데이터베이스 컬럼과 매핑되지 않는다.

```java
@Entity
@Table(name = "account")
public class Account {
	//...

	@Transient
	private String temp;
	
	//...
}
```

　
# @Access

JPA가 엔티티 데이터에 접근하는 방식을 지정할 수 있다. @Access를 생략하면 `@Id` 의 위치를 기준으로 접근방식이 기본 값을 설정된다.

```java
@Entity
@Table(name = "account")
@Access(AccessType.FIELD)
public class Account {
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long idx;

	//...
}
```

## AccessType.FIELD

필드에 직접 접근하는 방식으로 필드권한이 private이어도 접근 할 수 있다.

## AccessType.PROPERTY

Getter를 통해서 접근하는 방식이다.