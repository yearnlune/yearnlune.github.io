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

> graphql over http를 활용한 API 테스트
>

- spring-boot-starter-graphql:2.7.0
- junit-jupiter:5.8.2

## 기본구조

### Graphql over HTTP

grpahql over http 요청 시 POST Request의 형식이다. 해당 형식을 활용하여 graphql 테스트를 진행한다.

```kotlin
class GraphqlForm {

    data class Request(

        val query: String,

        val operationName: String? = null,

        val variables: MutableMap<String, *> = mutableMapOf<String, Any>()
    )

    data class Response(

				val data: Map<String, *>

        val errors: List<*>?,
    )
}
```

### MediaType.APPLICATION_GRAPHQL

content-type을 `application/graphql+json` 이나, `application/json`로 요청한다. 그리고, [spring-graphql](https://github.com/spring-projects/spring-graphql/blob/85ad5bbfb40006a39caa9f58395ab115e1d2e418/spring-graphql/src/main/java/org/springframework/graphql/server/webmvc/GraphQlHttpHandler.java)의 경우 `Mono` 를 통한 비동기 요청 및 응답을 하기 때문에 비동기 API 테스트로 진행한다.

```kotlin
@ExtendWith(SpringExtension::class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc  // MockMvc를 통해 HTTP 테스트 하기 위함
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
								// spring-graphql은 GraphQlHttpHandler를 활용하여 비동기 요청한다. 
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