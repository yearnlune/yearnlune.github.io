---
tags: 
    - java
    - test
    - junit
    - restful
    - springBoot
title: Java REST API Test 
date: 2020/12/20
author: 김동환
description:  자바 REST api 테스트 
disabled: true
categories:
  - java
---

> API의 Request와 그에 따른 Response, HttpStatus를 테스트

- junit:4.12
- spring-boot:2.2.4.RELEASE

### 기본 구조

```java
/**
 * @SpringBootTest와 @AutoConfigureMockMvc를 활용하여 통합테스트 진행
 */
@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureMockMvc
public class AccountControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    public void registerAccount_correctAccount_shouldBeCreated() throws Exception {
        //...
    }
}
```

```java
/**
 * @WebMvcTest를 활용하여 컨트롤러만 테스트 진행
 */
@RunWith(SpringRunner.class)
@WebMvcTest
public class AccountControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private AccountService accountService;

    @Test
    public void registerAccount_correctAccount_shouldBeCreated() throws Exception {
        //...
    }
}
```

`@SpringBootTest` : @SpringBootApplication를 찾아 실제 구동되는 어플리케이션과 동일하게 구동되며, 기본으로 `WebEnvironment.MOCK`으로 설정되어있다. 

`@AutoConfigureMockMvc` : `MockMvc`를 주입받을 수 있으며, @Controller 외에도 @Service, @Repository를 로드하여 통합테스트를 진행 할 수 있다. 

`@WebMvcTest` : `MockMvc` 를 주입받을 수 있으며, @Controller, @ControllerAdvice, WebMvcConfigurer, filters 를 로드하여, 컨트롤러에 대한 테스트를 진행 할 수 있다. 그외 필요한 Bean들은 `@MockBean` 을 통해 주입할 수 있다.