---
tags: 
    - kotlin
    - test
    - junit
    - graphql 
    - http
title: Kotlin Graphql Test
date: 2022/06/06
author: 김동환
description: 코틀린 spring graphql 테스트
disabled: true
categories:
  - kotlin
---

# GraphQL

> Graphql을 통한 API 테스트
>

- spring-boot-starter-graphql:2.7.0
- junit-jupiter:5.8.2

## Graphql over HTTP

`MockMvc` 를 활용하여 graphql의 요청 및 응답을 `JSON` 형식으로 처리하여 Spring MVC 테스트를 진행한다.

### graphql request & response body

graphql over http 요청 시 **POST body** 이다. 해당 형식을 활용하여 graphql 테스트를 진행한다.

```kotlin
class GraphqlForm {

    data class Request(

        val query: String,

        val operationName: String? = null,

        val variables: Map<String, Any> = mapOf()
    )

    data class Response(

				val data: Map<String, *>

        val errors: List<*>?,
    )
}
```

### support

request의 content-type을 `application/graphql+json` 이나, `application/json`로 요청한다. 그리고, [spring-graphql](https://github.com/spring-projects/spring-graphql/blob/85ad5bbfb40006a39caa9f58395ab115e1d2e418/spring-graphql/src/main/java/org/springframework/graphql/server/webmvc/GraphQlHttpHandler.java)의 경우 `Mono` 를 통한 **비동기 요청 및 응답**을 하기 때문에 비동기 API 테스트로 진행한다.

```kotlin
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc
@ActiveProfiles("test")
abstract class ControllerTestSupport {

		companion object {
        protected val objectMapper: ObjectMapper = jacksonObjectMapper()
    }

    @Autowired
    protected lateinit var mockMvc: MockMvc

    @Value("\${spring.graphql.path}")
    private lateinit var graphqlUrl: String

    protected fun postGraphql(requestBody: GraphqlForm.Request): MvcResult {
        val requestResult = mockMvc
            .perform(
                MockMvcRequestBuilders.post(graphqlUrl)
                    .content(objectMapper.writeValueAsBytes(requestBody))
                    .contentType(MediaType.APPLICATION_GRAPHQL)
                    .characterEncoding("UTF-8")
            ).andExpect {
                MockMvcResultMatchers.request().asyncStarted()
                MockMvcResultMatchers.request().asyncResult(CoreMatchers.notNullValue())
            }.andReturn()

        return mockMvc
            .perform(
                MockMvcRequestBuilders.asyncDispatch(requestResult)
            )
            .andExpect(MockMvcResultMatchers.status().isOk)
            .andReturn()
    }

    protected fun <T> getGraphqlResponseBody(name: String, response: String, clazz: Class<T>): T? =
        getResponseBody(name, response)?.let { objectMapper.readValue(objectMapper.writeValueAsString(it), clazz) }

    private fun getResponseBody(name: String, response: String): Any? {
        val graphqlResponse: GraphqlForm.Response = objectMapper.readValue(response)
        return graphqlResponse.data[name]
    }
}
```

## GraphQlTester

`org.springframework.graphql:spring-graphql-test` 의 `GraphQlTester` 를 활용하여 손쉽게 Graphql 테스트를 작성할 수 있다.

```kotlin
testImplementation("org.springframework.graphql:spring-graphql-test:1.0.0")
```

### support

`@AutoConfigureGraphQlTester` 를 활용하여 `GraphQlTester` 빈을 주입받는다.

```kotlin
@SpringBootTest
@AutoConfigureGraphQlTester
@ActiveProfiles("test")
abstract class GraphQlTestSupport {

    @Autowired
    protected lateinit var graphQlTester: GraphQlTester
}
```