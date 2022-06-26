---
tags: 
    - kotlin
    - test
    - junit
    - graphql 
    - http
    - GraphQlTester
title: Kotlin GraphQL Test
date: 2022/06/06
author: 김동환
description: Kotlin spring-graphql 테스트 (Graphql over HTTP, GraphQlTester)
disabled: false
categories:
  - kotlin
---

# GraphQL

> Graphql을 통한 API 테스트
>

- spring-boot-starter-web:2.7.0
- spring-boot-starter-graphql:2.7.0
- junit-jupiter:5.8.2

## Graphql over HTTP

`MockMvc` 를 활용하여 graphql의 요청 및 응답을 `JSON` 형식으로 처리하여 Spring MVC 테스트를 진행한다.

### graphql request & response

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

### example#1 shorthand syntax

Graphql Request Body의 `query`필드에 해당 syntax를 작성하여 요청한다.

```kotlin
/*
mutation {
    orderCreate(order: {
        items: [{
            itemId: "62946b347bfac12ff5f2ea56"
            quantity: 1
        }, {
            itemId: "62946b347bfac12ff5f2ea57"
            quantity: 2
        }]
    }) {
        id
        items {
            item {
                id
                name
                price
            }
            quantity
        }
    }
}
*/
@Test
@DisplayName("[GRAPHQL OVER HTTP|MUTATION] 주문 생성하기")
fun orderCreate() {
    /* GIVEN */
    val requestBody = GraphqlForm.Request(
        query = "mutation {\n" +
                "    orderCreate(order: {\n" +
                "        items: [{\n" +
                "            itemId: \"62946b347bfac12ff5f2ea56\"\n" +
                "            quantity: 1\n" +
                "        }, {\n" +
                "            itemId: \"62946b347bfac12ff5f2ea57\"\n" +
                "            quantity: 2\n" +
                "        }]\n" +
                "    }) {\n" +
                "        id\n" +
                "        items {\n" +
                "            item {\n" +
                "                id\n" +
                "                name\n" +
                "                price\n" +
                "            }\n" +
                "            quantity\n" +
                "        }\n" +
                "    }\n" +
                "}",
    )

    /* WHEN */
    val response = postGraphql(requestBody)
    val order = getGraphqlResponseBody("orderCreate", response.response.contentAsString, OrderCollection::class.java)

    /* THEN */
    MatcherAssert.assertThat(order?.items?.sumOf { it.item.price * it.quantity }, CoreMatchers.`is`(closeTo(26.65, 0.001)))
}
```

### example#2 operation syntax

Graphql Request Body의 `query`필드에 해당 syntax를 작성하며, `operationName`과 `variables`를 작성하여 요청한다.

```kotlin
/*
mutation OrderCreate($order: OrderInput){
    orderCreate(order: $order) {
        id
        items {
            item {
                id
                name
                price
            }
            quantity
        }
    }
}

# VARIABLES
{
    "order": {
        "items": [{
            "itemId": "62946b347bfac12ff5f2ea56",
            "quantity": 1
        }, {
            "itemId": "62946b347bfac12ff5f2ea57",
            "quantity": 2
        }]
    }
}
*/
@Test
@DisplayName("[GRAPHQL OVER HTTP|MUTATION] 주문 생성하기 with Operation")
fun orderCreateWithOperation() {
    /* GIVEN */
    val requestBody = GraphqlForm.Request(
        query = "mutation OrderCreate(\$order: OrderInput){\n" +
                "    orderCreate(order: \$order) {\n" +
                "        id\n" +
                "        items {\n" +
                "            item {\n" +
                "                id\n" +
                "                name\n" +
                "                price\n" +
                "            }\n" +
                "            quantity\n" +
                "        }\n" +
                "    }\n" +
                "}",
        operationName = "OrderCreate",
        variables = mapOf(Pair("order", OrderInput.builder()
            .withItems(
                listOf(
                    OrderItemInput.builder()
                        .withItemId("62946b347bfac12ff5f2ea56")
                        .withQuantity(1)
                        .build(),
                    OrderItemInput.builder()
                        .withItemId("62946b347bfac12ff5f2ea57")
                        .withQuantity(2)
                        .build()
                )
            )
            .build()))
    )

    /* WHEN */
    val response = postGraphql(requestBody)
    val order = getGraphqlResponseBody("orderCreate", response.response.contentAsString, OrderCollection::class.java)

    /* THEN */
    MatcherAssert.assertThat(order?.items?.sumOf { it.item.price * it.quantity }, CoreMatchers.`is`(closeTo(26.65, 0.001)))
}
```

## GraphQlTester

`org.springframework.graphql:spring-graphql-test` 의 `GraphQlTester` 를 활용하여 손쉽게 Graphql 테스트를 작성할 수 있다. 또한 GraphQL over HTTP인 `HttpGraphQlTester` , GraphQL over Websocket인 `WebSocketGraphQlTester` 를 지원한다.

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

### example#1 shorthand syntax

```kotlin
/*
mutation {
    orderCreate(order: {
        items: [{
            itemId: "62946b347bfac12ff5f2ea56"
            quantity: 1
        }, {
            itemId: "62946b347bfac12ff5f2ea57"
            quantity: 2
        }]
    }) {
        id
        items {
            item {
                id
                name
                price
            }
            quantity
        }
    }
}
*/
@Test
@DisplayName("[GRAPHQL|MUTATION] 주문 생성하기")
fun orderCreate() {
    /* GIVEN */
    val query = "mutation {\n" +
            "    orderCreate(order: {\n" +
            "        items: [{\n" +
            "            itemId: \"62946b347bfac12ff5f2ea56\"\n" +
            "            quantity: 1\n" +
            "        }, {\n" +
            "            itemId: \"62946b347bfac12ff5f2ea57\"\n" +
            "            quantity: 2\n" +
            "        }]\n" +
            "    }) {\n" +
            "        id\n" +
            "        items {\n" +
            "            item {\n" +
            "                id\n" +
            "                name\n" +
            "                price\n" +
            "            }\n" +
            "            quantity\n" +
            "        }\n" +
            "    }\n" +
            "}"

    /* WHEN */
    val response = graphQlTester
        .document(query).execute()
        .path("orderCreate").hasValue().entity(OrderCollection::class.java)

    /* THEN */
    MatcherAssert.assertThat(response.get().items?.sumOf { it.item.price * it.quantity }, CoreMatchers.`is`(Matchers.closeTo(26.65, 0.001)))
}
```

### example#2 operation syntax

```kotlin

/*
mutation OrderCreate($order: OrderInput){
    orderCreate(order: $order) {
        id
        items {
            item {
                id
                name
                price
            }
            quantity
        }
    }
}

# VARIABLES
{
    "order": {
        "items": [{
            "itemId": "62946b347bfac12ff5f2ea56",
            "quantity": 1
        }, {
            "itemId": "62946b347bfac12ff5f2ea57",
            "quantity": 2
        }]
    }
}
*/
@Test
@DisplayName("[GRAPHQL|MUTATION] 주문 생성하기 with Operation")
fun orderCreateWithOperation() {
    /* GIVEN */
    val query = "mutation OrderCreate(\$order: OrderInput){\n" +
            "    orderCreate(order: \$order) {\n" +
            "        id\n" +
            "        items {\n" +
            "            item {\n" +
            "                id\n" +
            "                name\n" +
            "                price\n" +
            "            }\n" +
            "            quantity\n" +
            "        }\n" +
            "    }\n" +
            "}"
    val variable = OrderInput.builder()
        .withItems(
            listOf(
                OrderItemInput.builder()
                    .withItemId("62946b347bfac12ff5f2ea56")
                    .withQuantity(1)
                    .build(),
                OrderItemInput.builder()
                    .withItemId("62946b347bfac12ff5f2ea57")
                    .withQuantity(2)
                    .build()
            )
        )
        .build()

    /* WHEN */
    val response = graphQlTester.document(query)
        .operationName("OrderCreate")
        .variable("order", variable.toMap()).execute()
        .path("orderCreate").hasValue().entity(OrderCollection::class.java)

    /* THEN */
    MatcherAssert.assertThat(response.get().items?.sumOf { it.item.price * it.quantity }, CoreMatchers.`is`(Matchers.closeTo(26.65, 0.001)))
}
```

# 예제

[spring-graphql-example](https://github.com/yearnlune/spring-graphql-example)