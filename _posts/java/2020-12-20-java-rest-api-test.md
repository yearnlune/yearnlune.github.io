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

`ResultActions andDo(ResultHandler handler)` 

: `MockMvcResultHandlers`를 통해서 해당 요청 및 응답을 처리하며, 콘솔창 출력이 가능하다.

`ResultActions andExpect(ResultMatcher matcher)`

: `MockMvcResultMatchers`를 통해서 해당 요청 및 응답의 검증을 진행할 수 있다.

| *static import를 활용하면 [ **MockMvcRequestBuilders, MockMvcResultHandlers, MockMvcResultMatchers** ]생략 가능하다.*

### **MockMvcRequestBuilders**

---

> 기본적인 HTTP Method를 지원하며, `MockHttpServletRequestBuilder` 를 활용하여 상세한 정의를 할 수 있다.

**HTTP GET**

`get(String url, Object... uriVars)`

: HTTP GET을  정의

```java
// CASE #1
mockMvc.perform(
	get("/accounts")
)
	.andDo(print())
	.andExpect(status().isOk());

// CASE #2
mockMvc.perform(
	get("/account/{idx}", 1)
)
	.andDo(print())
	.andExpect(status().isOk());
```
　

**HTTP POST**

`post(String url, Object... uriVars)`

: HTTP POST를 정의

```java
// REQUEST BODY 정의
String content = objectMapper.writeValueAsString(
			AccountDTO.RegisterRequest.builder()
				.id("dreamfactory")
				.name("드림팩토리")
				.password("P@ssW0rd")
				.build());

mockMvc.perform(
	post("/account")
		.content(content)
		.contentType(MediaType.APPLICATION_JSON)
		.characterEncoding("UTF-8")
)
	.andDo(print())
	.andExpect(status().isCreated());
```
　

**HTTP PUT**

`put(String url, Object... uriVars)`

: HTTP PUT을 정의

```java
// REQUEST BODY 정의
String content = objectMapper.writeValueAsString(
			AccountDTO.RegisterRequest.builder()
				.id("dreamfactory")
				.name("꿈공장")
				.password("P@ssW0rd")
				.build());

mockMvc.perform(
	put("/account/{idx}", 1)
		.content(content)
		.contentType(MediaType.APPLICATION_JSON)
		.characterEncoding("UTF-8")
)
	.andDo(print())
	.andExpect(status().isOk());
```
　

**HTTP DELETE**

`delete(String url, Object... uriVars)`

: HTTP DELETE를 정의

```java
mockMvc.perform(
	delete("/account/{idx}", 1);
)
	.andDo(print())
	.andExpect(status().isOk());
```
　

**HTTP PATCH**

`patch(String url, Object... uriVars)`

: HTTP PATCH를 정의

```java
// REQUEST BODY 정의
String patchedContent = objectMapper.writeValueAsString(
			AccountDTO.PatchedRequest.builder()
				.name("object")
				.build());

mockMvc.perform(
	patch(ACCOUNT + "/{idx}", 1)
		.content(patchedContent)
		.contentType(MediaType.APPLICATION_JSON)
		.characterEncoding("UTF-8")
		.accept(MediaType.APPLICATION_JSON)
		.header("Authorization", jwtToken)
)
	.andDo(print())
	.andExpect(status().isOk());

```

### MockHttpServletRequestBuilder

---

`content(String content)` 

: RequestBody의 컨텐츠를 정의하며, `String`외 `Byte[]` 도 지원한다.

`cotentType(String contentType)` 

: RequestBody의 Type을 정의한다. `String`외 `MediaType`도 지원한다.

`characterEncoding(String encoding)` 

: 해당 요청의 문자열 인코딩을 정의한다. 

`accept(String... mediaTypes)` 

: 응답처리 Type을 정의한다. `String...`외 `MediaType...`도 지원한다.

`header(String name, Object... values)`

: 요청의 헤더를 정의할 수 있다.

`headers(HttpHeaders httpHeaders)`

: 여러 헤더들을 `HttpHeaders`를 통해서 정의할 수 있다.

### MockMvcResultHandlers

---

`print()`

: 요청 및 응답에 대한 메시지를 콘솔에 출력한다. `OutputStream`, `Writer`와 같은 출력 대상을 적용할 수 있다.

```java
mockMvc.perform(
	get("/accounts")
)
	.andDo(print())
	.andExpect(status().isUnauthorized());

/*
MockHttpServletRequest:
      HTTP Method = GET
      Request URI = /accounts
       Parameters = {}
          Headers = []
             Body = null
    Session Attrs = {SPRING_SECURITY_SAVED_REQUEST=DefaultSavedRequest[http://localhost/accounts]}

.
.
.

MockHttpServletResponse:
           Status = 403
    Error message = Access Denied
          Headers = [X-Content-Type-Options:"nosniff", X-XSS-Protection:"1; mode=block", Cache-Control:"no-cache, no-store, max-age=0, must-revalidate", Pragma:"no-cache", Expires:"0", X-Frame-Options:"DENY"]
     Content type = null
             Body = 
    Forwarded URL = null
   Redirected URL = null
          Cookies = []
*/
```

### MockMvcResultMatchers

---

`status()` 

: 응답의 대한 HttpStatus를 검증을 할 수 있다. `is()` 메소드를 활용하여 HttpStatus code를 입력할 수 있으며, `CASE #1`와 같이 human-friendly 메소드를 활용할 수 있다.

```java
// CASE #1
mockMvc.perform(
	get("/accounts")
		.header("Authorization", jwtToken)
)
	.andDo(print())
	.andExpect(status().isOk());

// CASE #2
mockMvc.perform(
	get("/accounts")
		.header("Authorization", jwtToken)
)
	.andDo(print())
	.andExpect(status().is(200));
```
　

`content()` 

: 응답의 대한 ResponseBody를 검증을 할 수 있다. `json()`, `string()`, `bytes()`등을 활용하여 ResponseBody에 대한 검증을 할 수 있으며, `encoding()`, `contentType()`등을 활용하여 메타정보를 검증할 수 있다.  

```java
mockMvc.perform(
	get("/account/{idx}", 1)
		.header("Authorization", jwtToken)
)
	.andDo(print())
	.andExpect(content().encoding("UTF-8"))
	.andExpect(content().json("{\"idx\":1,\"id\":\"object\",\"name\":\"오브젝트\",\"role\":\"ROLE_GUEST\"}"))
	.andExpect(status().isOk());
```
　

`header()` 

: 응답의 대한 Header를 검증 할 수 있다. 헤더의 대한 존재 유무 및 헤더의 값을 검증할 수 있다.

```java
mockMvc.perform(
	get("/accounts")
		.header("Authorization", jwtToken)
)
	.andDo(print())
	.andExpect(header().exists("X-XSS-Protection"))
	.andExpect(header().stringValues("Cache-Control", "no-cache, no-store, max-age=0, must-revalidate"))
	.andExpect(status().isOk());
```