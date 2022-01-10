---
tags: 
    - java
    - test
    - junit
    - stomp
    - websocket
title: Java STOMP test
date: 2022/01/10
author: 김동환
description: 자바 spring stomp 테스트
disabled: false
categories:
  - java
---

　
# STOMP

- junit:5.8.2
- spring-boot:2.6.2

## 기본 구조

### Websocket Client

> 테스트를 하기 위한 Websocket Client 생성, 접속 연결 및 종료를 관리한다.
>

```java
@ExtendWith(SpringExtension.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class StompSupport {

  protected StompSession stompSession;

  @LocalServerPort
  private int port;

  private final String url;

  private final WebSocketStompClient websocketClient;

  public StompSupport() {
    this.websocketClient = new WebSocketStompClient(new SockJsClient(createTransport()));
    this.websocketClient.setMessageConverter(new MappingJackson2MessageConverter());
    this.url = "ws://localhost:";
  }

  @BeforeEach
  public void connect() throws ExecutionException, InterruptedException, TimeoutException {
    this.stompSession = this.websocketClient
      .connect(url + port + ENDPOINT, new StompSessionHandlerAdapter() {})
      .get(3, TimeUnit.SECONDS);
  }

  @AfterEach
  public void disconnect() {
    if (this.stompSession.isConnected()) {
      this.stompSession.disconnect();
    }
  }

  private List<Transport> createTransport() {
    List<Transport> transports = new ArrayList<>(1);
    transports.add(new WebSocketTransport(new StandardWebSocketClient()));
    return transports;
  }
}
```

### Message Handler

> Response를 처리하는 핸들러
>

```java
public class MessageFrameHandler<T> implements StompFrameHandler {

  @Getter
  private final CompletableFuture<T> completableFuture = new CompletableFuture<>();

  private final Class<T> tClass;

  public MessageFrameHandler(Class<T> tClass) {
    this.tClass = tClass;
  }

  @Override
  public Type getPayloadType(StompHeaders headers) {
    return this.tClass;
  }

  @Override
  public void handleFrame(StompHeaders headers, Object payload) {
    completableFuture.complete((T)payload);
  }
}
```

## 따라해보기

```java
@Test
public void findUsers() throws ExecutionException, InterruptedException, TimeoutException {
  /* GIVEN */
  MessageFrameHandler<UserDTO[]> handler = new MessageFrameHandler<>(UserDTO[].class);
  this.stompSession.subscribe("/user/queue/users", handler);

  /* WHEN */
  this.stompSession.send("/app/users", "");

  /* THEN */
  List<UserDTO> userList = List.of(handler.getCompletableFuture().get(3, TimeUnit.SECONDS));

  assertThat(userList, notNullValue());
  assertThat(userList.size(), greaterThan(0));
}
```

1. Message handler인 `MessageFrameHandler`를 Response 타입으로 초기화 해준다.
2. Response 받기 위해 subscribe을 한다.
3. Request를 요청한다.
4. 테스트를 검증한다.

## 예제

[https://github.com/yearnlune/stomp-unit-test-example](https://github.com/yearnlune/stomp-unit-test-example)