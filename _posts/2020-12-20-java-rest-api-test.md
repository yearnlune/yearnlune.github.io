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
disabled: false
categories:
  - java
---

> API의 Request와 그에 따른 Response, HttpStatus를 테스트

- junit:4.12
- spring-boot:2.2.4.RELEASE

## 기본 구조

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

`@SpringBootTest` 
: @SpringBootApplication를 찾아 실제 구동되는 어플리케이션과 동일하게 구동되며, 기본으로 `WebEnvironment.MOCK`으로 설정되어있다. 

`@AutoConfigureMockMvc` 
: `MockMvc`를 주입받을 수 있으며, @Controller 외에도 @Service, @Repository를 로드하여 통합테스트를 진행 할 수 있다. 

`@WebMvcTest` 
: `MockMvc` 를 주입받을 수 있으며, @Controller, @ControllerAdvice, WebMvcConfigurer, filters 를 로드하여, 컨트롤러에 대한 테스트를 진행 할 수 있다. 그외 필요한 Bean들은 `@MockBean` 을 통해 주입할 수 있다.

## MockMvc

> Spring의 MVC의 동작을 테스트 할 수 있는 클래스

```java
    @Autowired
    private MockMvc mockMvc;
    
    @Autowired
    private ObjectMapper objectMapper;
    
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
            post("/account")
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

## 주요 사용

`ResultActions perform(RequestBuilder requestBuilder)`

: `MockMvcRequestBuilders`를 통해서 구현한 Request를 테스트 한다.

### 공통

`content(String content)` 

: RequestBody의 컨텐츠를 정의하며, `String`외 `Byte[]` 도 지원한다.

`cotentType(String contentType)` 
: RequestBody의 Type을 정의한다. `String`외 `MediaType`도 지원한다.

`characterEncoding(String encoding)` 

: 해당 요청의 문자열 인코딩을 정의한다. 

`accept(String... mediaTypes)` 

: 응답처리 Type을 정의한다. `String...`외 `MediaType...`도 지원한다.

### HTTP GET

`get(String url, Object... uriVars)`

: HTTP GET을  정의할 수 있다. 

```java
// CASE #1
MockMvcRequestBuilders.get("/accounts");

// CASE #2
MockMvcRequestBuilders.get("/account/{id}", "7215776e-09f3-4e39-9fe8-6a6b1c70a366");
```

### HTTP POST

`post(String url, Object... uriVars)`

: HTTP POST를 정의할 수 있다.

```java
String content = objectMapper.writeValueAsString(
			AccountDTO.RegisterRequest.builder()
				.id("object")
				.name("Object")
				.password("object")
				.build());

MockMvcRequestBuilders
	.post("/account")
		.content(content)
		.contentType(MediaType.APPLICATION_JSON);
	
```