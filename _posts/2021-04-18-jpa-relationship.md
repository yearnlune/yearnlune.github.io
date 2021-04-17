---
tags: 
    - java
    - jpa
    - relationship
title: JPA Relationship
date: 2021/04/18
author: 김동환
description: JPA Relationship
disabled: false
categories:
  - java
---
　
# @JoinColumn

외래키를 매핑할 때 사용한다.

## Attribute

### name

매핑할 컬럼 이름을 설정할 수 있으며, 기본값은 `필드명 + _ + 타겟 테이블 기본 키 컬럼 이름`이다.

### referencedColumnName

외래 키가 참조하는 타겟 테이블의 컬럼 이름을 설정할 수 있으며, 기본값은 `타겟 테이블의 기본 키 컬럼 이름`이다.

### foreignKey

`@ForeignKey`를 활용하여 외래키 제약조건을 설정할 수 있다.

```java
@Getter
@Builder
@NoArgsConstructor
@AllArgsConstructor
@Entity
@Table(name = "order")
@SQLDelete(sql = "UPDATE `order` SET is_deleted = 1 WHERE id = ?")
@Where(clause = "is_deleted = false")
public class Order {
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long id;

	@ManyToOne
	@JoinColumn(name = "user_id", foreignKey = @ForeignKey(name = "fk_order_user_id"), nullable = false)
	private User user;

	@ManyToMany
	private List<Item> itemList;

	@Builder.Default
	@ColumnDefault("false")
	@Column(nullable = false)
	private Boolean isDeleted = false;
}
```

　
# 다중성

entity 간의 1:1, 1:N, N:N을 나타내는 `@OneToOne` , `@OneToMany` , `@ManyToOne` , `@ManyToMany` 를 의미한다.

## Attribute

### optional

참조된 entity의 필수 존재 여부를 설정한다. 기본값은 `true`로 nullable이기 때문에 일반적으로 join 시  outer Join을 한다. `false` 일 경우는 일반적으로 inner join을 한다. `@JoinColumn`에서 nullable 속성값을 준것과 동일하다.

### fetch

참조된 entity의 데이터를 가져오는 전략을 선택한다. 총 두 가지의 fetch 전략이 있으며, `FetchType.EAGER`는 참조된 entity를 즉시 조회하는 전략이다. 기본적으로 Join을 통해서 한 번에 조회한다. `FetchType.LAZY`는 참조된 entity가 실제 사용할 때 조회하는 전략이다.

```java
public enum FetchType {

    LAZY,

    EAGER
}
```

### cacade

해당 Entity와 연관된 entity의 상태를 함께 제어할 수 있다.

```java
public enum CascadeType { 

    ALL, 

    PERSIST, 

    MERGE, 

    REMOVE,

    REFRESH,

    DETACH
}
```

`CascadeType.ALL` : 위의 나열된 모든 종류를 적용한다.

`CascadeType.PERSIST` : 해당 entity의 `persist()` 발생 시 연관된 entity 모두 저장(영속화, `persist()`)한다.

`CascadeType.MERGE` : 해당 entity의 `merge()` 발생 시 연관된 entity 모두 수정하거나 저장(`merge()`)한다.

`CascadeType.REMOVE` :  해당 entity의 `remove()` 발생 시 연관된 entity 모두 삭제(`remove()`)한다.

`CascadeType.REFRESH` : 해당 entity의 `refresh()` 발생 시 연관된 entity 모두 DB로부터 다시 불러(`refresh()`)온다.

`CascadeType.DETACH` : 해당 entity의 `detach()` 발생 시 연관된 entity 모두 준 영속 상태(영속성 컨텍스트에서 삭제, `detach()`)로 바꾼다.

### targetEntity

연관된 entity의 타입을 정의할 수 있다. 기본 값은 해당 Field의 타입을 읽어드린다. 그렇기에 주로 사용되진 않는다.

```java
@ManyToMany
private List<Item> itemList;

@ManyToMany(targetEntity = Item.class)
private List itemList;
```

### mappedBy

외래키를 관리하지 않는 entity에 양방향 연관관계를 설정하는 데 사용된다. 외래키를 관리하는 연관관계의 주인(Owner)의 FK 필드 이름으로 설정한다.

```java
public class User {
	//...
	@OneToMany(mappedBy = "user", cascade = CascadeType.REMOVE)
	private List<Order> orderList;
}

public class Order {
	//...
	@ManyToOne(optional = false)
	@JoinColumn(name = "user_id", foreignKey = @ForeignKey(name = "fk_order_user_id"))
	private User user;
}
```

### orphanRemoval

부모 entity와 연관 관계가 사라진 자식 entity를 자동으로 삭제해준다. 기본값은 `false` 이다.

```java
public class User {
	//...
	@OneToMany(mappedBy = "user", cascade = CascadeType.REMOVE, orphanRemoval = true)
	private List<Order> orderList;
}

public class Order {
	//...
	@ManyToOne(optional = false)
	@JoinColumn(name = "user_id", foreignKey = @ForeignKey(name = "fk_order_user_id"))
	private User user;
}
```

　
## @OneToOne

1:1 형식의 연관 관계를 설정할 때 사용한다. FK가 생성되는 주체(Owner)에 `@JoinColumn` 을 추가하여 나타낼 수 있다. 기본 fetch 전략은 즉시 로딩(`FetchType.EAGER`)이다.

```java
public class Person {
	//...
	@OneToOne(mappedBy = "person")
	private Passport passport;
}

public class Passport {
	//...
	@OneToOne(optional = false)
	@JoinColumn
	private Person person;
}
```

　
## @ManyToOne & @OneToMany

1:N, N:1 형식의 연관 관계를 설정할 때 사용한다. 기본 fetch 전략은 `@ManyToOne` 은 즉시 로딩(`FetchType.EAGER`), `@OneToMany` 는 지연 로딩(`FetchType.LAZY`) 이다.

```java
public class User {
	//...
	@OneToMany(mappedBy = "user")
	private List<Order> orderList;
}

public class Order {
	//...
	@ManyToOne(optional = false)
	@JoinColumn
	private User user;
}
```

　
## @ManyToMany

N:N 형식의 연관관계를 설정할 때 사용한다. `@JoinTable` 을 추가하여 나타낼 수 있다. 기본 fetch 전략은 지연 로딩(`FetchType.Lazy`)이다.

```java
public class Order {
	//...
	@ManyToMany
	private List<Item> itemList;
}

public class Item {
	//...	
	@ManyToMany(mappedBy = "itemList")
	private List<Order> orderList;
}
```