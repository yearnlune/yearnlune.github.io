---
tags:
- graphql
- spring
- spring-boot
- spring-graphql
- kotlin
title: spring-graphql 
date: 2022/06/23
author: 김동환
description: spring-graphql를 활용한 웹서비스 개발하기 
disabled: false
categories:
- spring
---

　

기존에 graphql을 활용한 Spring boot 서비스를 개발하기 위해서 `com.graphql-java` 나 `com.graphql-java-kickstart` 에서 지원해주는 라이브러리를 사용하였다. 
이제 [스프링부트 2.7.0 버전](https://github.com/spring-projects/spring-boot/releases/tag/v2.7.0)이 릴리즈되면서 
[spring-graphql](https://github.com/spring-projects/spring-graphql/releases/tag/v1.0.0)도 정식으로 지원하게 되었다. 해당 spring-graphql 를 활용하여 백엔드 서비스를 개발해보자.

🔗 **사용 기술 및 라이브러리**

- JDK 11
- Kotlin 1.6.21
- Spring Boot 2.7.0
- Gradle 7.4.1
- WebSocket
- MongoDB

# Graphql schema

## *.graphqls

`*.graphqls` 파일을 생성하고 type, query, mutation, scalar 등을 정의한다.

```graphql
# schema.graphqls

interface Collection {
    id: String
}

input SortInput {
    property: String,
    isDescending: Boolean
}

input SearchInput {
    by: String
    operator: String
    type: String
    value: [String]
}

input PageInput {
    pageNumber: Long
    pageSize: Long
    sort: [SortInput]
    search: [SearchInput]
}

interface PageResponse {
    total: Long!
}

type Query {
    collectionGet: String
}

type Mutation {
    collectionUpdate: String
}
```

```graphql
# item.graphqls

type ItemCollection implements Collection {
    id: String
    name: String!
    price: Float
}

input ItemInput {
    name: String!
    price: Float
}

extend type Mutation {
    itemCreate(item: ItemInput): ItemCollection
}
```

## graphql_java_generator plugin

`*.graphqls` 에 정의한 type들을 DTO로 활용하기 위해 graphql로 정의한 타입을 클래스로 자동
생성해주는 [com.graphql_java_generator.graphql-gradle-plugin](https://github.com/graphql-java-generator/graphql-gradle-plugin-project)
를 적용한다.

```kotlin
// build.gradle.kts

plugins {
    ...
    id("com.graphql_java_generator.graphql-gradle-plugin") version "1.18.6"
}

dependencies {
    implementation("com.graphql-java-generator:graphql-java-common-runtime:1.18.6")
}
```

### generatePojoConf

클래스 자동 생성을 위한 configuration을 다음과 같이 정의 한다.

```kotlin
// build.gradle.kts
generatePojoConf {
    packageName = "$group.example.graphql"  -- 1
    setSchemaFileFolder("$rootDir/src/main/resources/graphql")  -- 2
    mode = com.graphql_java_generator.plugin.conf.PluginMode.server  -- 3
}

tasks {
    compileKotlin {
        dependsOn("generatePojo")
    }
}
```

1. 자동 생성되는 클래스의 **base package**를 설정 할 수 있다.
2. **graphql schema file path**를 설정 할 수 있다.
3. graphql server를 명시 한다. default는 `PluginMode.client`이다.

## custom scalar

graphql에서 `Int`, `Float`, `String`, `Boolean`, `ID`  총 다섯 가지의 기본 scalar를 제공하고 있다. 하지만 이외에 타입을 쓰려면 custom scalar를 정의하여
사용해야 한다. `graphql-java-extended-scalars` 를 활용하면 주로 사용될 수 있는 scalar를 손쉽게 사용할 수 있다.

```kotlin
// build.gradle.kts

dependencies {
    implementation("com.graphql-java:graphql-java-extended-scalars:18.1")
}
```

### custom scalar 선언

graphql에 custom scalar을 선언한다.

```graphql
# schema.graphqls

scalar Long
```

### graphql_java_generator에 등록

graphql_java_generator에서 custom scalar를 이해할 수 있게 등록해줘야 한다.

```kotlin
// build.gradle.kts
generatePojoConf {
    ...

    customScalars.push(
        com.graphql_java_generator.plugin.conf.CustomScalarDefinition(  -- 1
            "Long",
            "java.lang.Long",
            null,
            "graphql.scalars.ExtendedScalars.GraphQLLong",
            null
        )
    )
}
```

1. `CustomScalarDefinition` 생성자는 다음과 같다.

```java
CustomScalarDefinition(
    final String graphQLTypeName,  // custom scalar 이름
    final String javaType, // custom scalar와 매핑될 java 타입
    final String graphQLScalarTypeClass, // custom scalar 타입을 정의한 class
    final String graphQLScalarTypeStaticField, // custom scalar 타입의 instance
    final String graphQLScalarTypeGetter  // custom scalar 타입의 getter
)
```

### spring-graphql에 등록

spring-graphql에서 custom scalar를 이해할 수 있게 등록해줘야 한다.

```kotlin
@Configuration
class GraphqlWiringConfig : RuntimeWiringConfigurer {

    override fun configure(builder: RuntimeWiring.Builder) {
        builder
            .scalar(ExtendedScalars.GraphQLLong)
            .build()
    }
}
```

# Web Service

`spring-graphql`의 기본 프로퍼티를 쉽게 설정할 수 있다.

```yaml
# application.yaml
spring:
  graphql:
    path: /example/graphql  # graphql endpoint, default '/graphql'
  websocket:
    path: /example/graphql  # webSocket endpoint
    schema:
      locations: classpath:graphql/**/  # graphql schema path, default 'classpath:graphql/**/'
```

## Controller

spring-graphql에서는 graphql의 `query`, `mutation`, `subscription`를 어노테이션으로 지원해준다. 또한 `input` 의 경우 `@Argument`를 통해 매핑해 줄 수
있다.

### Query

`@QueryMapping` 을 통해 graphql의 `Query`와 매핑해 줄 수 있으며, 기본적으로는 해당 어노테이션을 선언한 메소드의 이름으로 매핑한다.

```graphql
# item.graphqls
extend type Query {
    itemsGet(page: PageInput): ItemPagination
}
```

```kotlin
// ItemController
@QueryMapping("itemsGet")
fun findItems(@Argument page: PageInput): ItemPagination {
    val items = itemService.findItems(page)

    return ItemPagination.builder()
        .withItems(items.content)
        .withTotal(items.totalElements)
        .build()
}
```

### Mutation

`@MutationMapping` 을 통해 graphql의 `Mutation`와 매핑해 줄 수 있으며, 기본적으로는 해당 어노테이션을 선언한 메소드의 이름으로 매핑한다.

```graphql
extend type Mutation {
    itemCreate(item: ItemInput): ItemCollection
}
```

```kotlin
@MutationMapping("itemCreate")
fun createItem(@Argument item: ItemInput) = itemService.createItem(item)
```

### Subscription

`@SubscriptionMapping` 을 통해 graphql의 `Subscription`와 매핑할 수 있으며, websocket과 publisher를 활용하여 처리한다.

```graphql
extend type Subscription {
    itemsSubscribe: ItemCollection
}
```

```kotlin
@SubscriptionMapping("itemsSubscribe")
fun subscribeItems(): Flux<ItemCollection> = itemService.findItems()
```

## Exception

서버에서 예외를 처리하여 graphql.errors에 해당 오류 정보를 클라이언트에게 전달해야한다. 일반적으로 서버의 에러는 따로 처리하지 않는 이상 `INTERNAL_ERROR` 로 처리하여 전달한다.

### GraphQLError

`GraphQLError` 인터페이스를 구현하여 Graphql의 예외를 처리할 수 있다. GraphQLError에는 크게 `message` , `locations` , `errorType` , `extentions`
가 존재한다.

- message : 에러의 기본 메시지
- locations : 에러가 발생한 소스 위치 (line, column)
- errorType : GraphQL의 에러 분류 (`graphql.ErrorType`)
- extensions : 예외 확장 정보

```kotlin
abstract class GraphqlException(

    private val errorCode: Int = 0,

    @JvmField
    @Suppress("INAPPLICABLE_JVM_FIELD")
    override val message: String?
) : GraphQLError, RuntimeException(message) {

    override fun getMessage(): String? = message

    override fun getLocations(): MutableList<SourceLocation>? = null

    override fun getErrorType(): ErrorClassification? = null

    override fun getExtensions(): MutableMap<String, Any> {
        return mutableMapOf(
            Pair("code", this.errorCode),
            Pair("exception", this.javaClass.simpleName)
        )
    }
}
```

```kotlin
class ValidationException(

    @JvmField
    @Suppress("INAPPLICABLE_JVM_FIELD")
    override val message: String? = "Validation failed"
) : GraphqlException(10, message) {

    override fun getMessage(): String? {
        return super.getMessage()
    }

    override fun getErrorType(): ErrorClassification {
        return ErrorType.ValidationError
    }
}
```

### DataFetcherExceptionResolverAdapter

서버에서 처리된 예외들을 GraphQLError로 처리할 수 있다. GraphQLError를 구현한 예외 뿐만 아니라 기본적인 예외도 GraphQLError로 처리 가능하다.

```kotlin
@Configuration
class GraphqlExceptionConfig : DataFetcherExceptionResolverAdapter() {

    override fun resolveToSingleError(ex: Throwable, env: DataFetchingEnvironment): GraphQLError? {
        return when (ex) {
            is ValidationException -> ValidationException(ex.message)
            is RuntimeException -> BadRequestException(ex.message)
            else -> super.resolveToSingleError(ex, env)
        }
    }
}
```

# 예제

[spring-graphql-example](https://github.com/yearnlune/spring-graphql-example)