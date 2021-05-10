---
tags: 
    - java
    - spring
    - event
title: Spring Events
date: 2021/05/11
author: 김동환
description: Spring Events (ApplicationEventPublisher, @EventListener, @Async)
disabled: false
categories:
  - java
---

　
# Event 처리

> 여러 비즈니스 로직을 이벤트를 활용하여 처리가능하다. 기존과 달리  Pub/Sub 형태이기 때문에 느슨하게 결합할 수 있다.

　
## Event

Spring 4.2 버전 이후 POJO 스타일로 사용가능하다. 이전 버전에서는 `ApplicationEvent`를 상속받아 구현해야 한다.

```java
@Getter
@Setter
@NoArgsConstructor
public abstract class BaseEvent {
}

@Getter
@Setter
@NoArgsConstructor
public class OrderEvent extends BaseEvent {
}

@Getter
@Setter
@NoArgsConstructor
public class UserEvent extends BaseEvent {
}
```

　
## Publish

`ApplicationEventPublisher`의 `void publishEvent(Object event)`를 활용하여 이벤트를 발생시킬 수 있다.

```java
@Component
public class EventPublisher {

	final
	ApplicationEventPublisher applicationEventPublisher;

	public EventPublisher(
		ApplicationEventPublisher applicationEventPublisher) {
		this.applicationEventPublisher = applicationEventPublisher;
	}

	public <T extends BaseEvent> void publish(T event) {
		applicationEventPublisher.publishEvent(event);
	}
}
```

　
## Subscribe

Spring 4.2 버전 이후 `@EventListener` 를 통해서 이벤트를 수신받을 수 있다. 이전 버전에서는 `ApplicationListener<E>`를 구현해야 한다.

```java
@Component
public class OrderListener {

	@EventListener
	public void handleOrder(OrderEvent orderEvent) throws Exception {
		Thread.sleep(2000);
		System.out.println(System.currentTimeMillis() + " handleOrder");
	}
}

@Component
public class UserListener {

	@EventListener
	public void handleUser(UserEvent userEvent) throws Exception {
		Thread.sleep(1000);
		System.out.println(System.currentTimeMillis() + " handleUser");
	}

	@EventListener
	public void handleUserOrder(OrderEvent orderEvent) throws Exception {
		Thread.sleep(3000);
		System.out.println(System.currentTimeMillis() + " handleUserOrder");
	}
}
```

　
## 예시

```java
@Test
public void orderEvent() throws Exception {
	OrderEvent orderEvent = new OrderEvent();

	System.out.println(System.currentTimeMillis() + " START");
	eventPublisher.publish(orderEvent);
	System.out.println(System.currentTimeMillis() + " END");

	Thread.sleep(5000);
}

/* Results 순서는 상이할 수 있음
1620656350473 START
1620656352486 handleOrder
1620656355499 handleUserOrder
1620656355499 END
*/

@Test
public void event() throws Exception {
	OrderEvent orderEvent = new OrderEvent();
	UserEvent userEvent = new UserEvent();

	System.out.println(System.currentTimeMillis() + " START");
	eventPublisher.publish(orderEvent);
	System.out.println(System.currentTimeMillis() + " DIVIDER");
	eventPublisher.publish(userEvent);
	System.out.println(System.currentTimeMillis() + " END");

	Thread.sleep(5000);
}

/* Results orderEvent의 순서는 상이할 수 있음
1620656315234 START
1620656317237 handleOrder
1620656320238 handleUserOrder
1620656320238 DIVIDER
1620656321248 handleUser
1620656321248 END
*/
```

　
# Event 순서 처리

> **동일 Event의 처리**에 있어 순서를 정하기 위해선 `@EventListener`에  `@Order`를 추가, 활용해야 한다. 숫자가 낮을 수록 우선순위가 높다.

```java
@Component
public class OrderListener {

	@Order(2)
	@EventListener
	public void handleOrder(OrderEvent orderEvent) throws Exception {
		Thread.sleep(2000);
		System.out.println(System.currentTimeMillis() + " handleOrder");
	}
}

@Component
public class UserListener {

	@Order(0)
	@EventListener
	public void handleUser(UserEvent userEvent) throws Exception {
		Thread.sleep(1000);
		System.out.println(System.currentTimeMillis() + " handleUser");
	}

	@Order(1)
	@EventListener
	public void handleUserOrder(OrderEvent orderEvent) throws Exception {
		Thread.sleep(3000);
		System.out.println(System.currentTimeMillis() + " handleUserOrder");
	}
}
```

　
## 예시

```java
@Test
public void orderEvent() throws Exception {
	OrderEvent orderEvent = new OrderEvent();

	System.out.println(System.currentTimeMillis() + " START");
	eventPublisher.publish(orderEvent);
	System.out.println(System.currentTimeMillis() + " END");

	Thread.sleep(5000);
}

/* Results
1620656219744 START
1620656222754 handleUserOrder
1620656224760 handleOrder
1620656224760 END
*/

@Test
public void event() throws Exception {
	OrderEvent orderEvent = new OrderEvent();
	UserEvent userEvent = new UserEvent();

	System.out.println(System.currentTimeMillis() + " START");
	eventPublisher.publish(orderEvent);
	System.out.println(System.currentTimeMillis() + " DIVIDER");
	eventPublisher.publish(userEvent);
	System.out.println(System.currentTimeMillis() + " END");

	Thread.sleep(5000);
}

/* Results
1620656258732 START
1620656261740 handleUserOrder
1620656263752 handleOrder
1620656263752 DIVIDER
1620656264760 handleUser
1620656264760 END
*/
```

　
# Event 비동기 처리

> Spring 설정 클래스에 `@EnableAsync`를 추가하고, `@EventListener`에  `@Async`를 추가, 활용해야 한다.

```java
@EnableAsync
@SpringBootApplication
public class EventExampleApplication {

	public static void main(String[] args) {
		SpringApplication.run(EventExampleApplication.class, args);
	}
}
```

```java
@Component
public class OrderListener {

	@Async
	@EventListener
	public void handleOrder(OrderEvent orderEvent) throws Exception {
		Thread.sleep(2000);
		System.out.println(System.currentTimeMillis() + " handleOrder");
	}
}

@Component
public class UserListener {

	@Async
	@EventListener
	public void handleUser(UserEvent userEvent) throws Exception {
		Thread.sleep(1000);
		System.out.println(System.currentTimeMillis() + " handleUser");
	}

	@Async
	@EventListener
	public void handleUserOrder(OrderEvent orderEvent) throws Exception {
		Thread.sleep(3000);
		System.out.println(System.currentTimeMillis() + " handleUserOrder");
	}
}
```

　
## 예시

```java
@Test
public void orderEvent() throws Exception {
	OrderEvent orderEvent = new OrderEvent();

	System.out.println(System.currentTimeMillis() + " START");
	eventPublisher.publish(orderEvent);
	System.out.println(System.currentTimeMillis() + " END");

	Thread.sleep(5000);
}

/* Results
1620656121067 START
1620656121070 END
1620656123098 handleOrder
1620656124091 handleUserOrder
*/

@Test
public void event() throws Exception {
	OrderEvent orderEvent = new OrderEvent();
	UserEvent userEvent = new UserEvent();

	System.out.println(System.currentTimeMillis() + " START");
	eventPublisher.publish(orderEvent);
	System.out.println(System.currentTimeMillis() + " DIVIDER");
	eventPublisher.publish(userEvent);
	System.out.println(System.currentTimeMillis() + " END");

	Thread.sleep(5000);
}

/* Results
1620656048705 START
1620656048707 DIVIDER
1620656048708 END
1620656049721 handleUser
1620656050718 handleOrder
1620656051715 handleUserOrder
*/
```