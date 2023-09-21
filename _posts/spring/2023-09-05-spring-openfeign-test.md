---
tags:
- spring
- spring-boot
- feign
- test
- kotlin
title: feign 테스트 - wireMock을 활용하여 mocking하기
date: 2023/09/05
author: 김동환
description: spring-boot, kotlin, feign, wiremock, junit5
disabled: false
categories:
- spring
---


# WireMock이란?

WireMock은 HTTP 기반 API 서비스를 테스트하기 위한 도구입니다. 서비스에서 발생하는 HTTP 요청에 대해 원하는 대로 응답을 설정할 수 있습니다.

# 따라해보기
--------
**사용 기술 및 라이브러리**

- JDK 17
- Kotlin 1.8.22
- Spring Boot 3.1.3
- Gradle 8.2.1
- Wiremock 4.0.4
- Feign 4.0.4



**예시 프로젝트**

[wiremock-example](https://github.com/yearnlune/wiremock-example)

## Gradle

```jsx
testImplementation("org.springframework.cloud:spring-cloud-contract-wiremock:4.0.4")
```

## HTTP API

feign을 통해 외부 API를 정의하였다.

```kotlin
@FeignClient(
  name = "another",
  url = "\${app.another.url}"
)
interface AnotherClient {

  @GetMapping("/api/another/nickname/{nickname}")
  fun hasNickname(@PathVariable("nickname") nickname: String): Boolean

  @PutMapping("/api/another/{id}")
  fun updateNickname(
    @PathVariable("id") id: String,
    @RequestParam("nickname") nickname: String
  ): AnotherDTO

  @PostMapping("/api/another")
  fun createAnother(
    @RequestBody another: AnotherDTO
  ): AnotherDTO
}
```

## Service

```kotlin
@Service
class ExampleService(
  private val anotherClient: AnotherClient
) {

  fun exampleFunctionA(nickname: String): Result<String> {
    return runCatching {
      /* Some codes... */
      if (anotherClient.hasNickname(nickname)) {
        "A"
      } else {
        "B"
      }
    }
  }

  fun exampleFunctionB(id: String, nickname: String): Result<AnotherDTO> {
    return runCatching {
      /* Some codes... */
      anotherClient.updateNickname(id, nickname)
    }
  }

  fun exampleFunctionC(anotherDTO: AnotherDTO): Result<AnotherDTO> {
    return runCatching {
      /* Some codes... */
      anotherClient.createAnother(anotherDTO)
    }
  }
}
```

## Mocking

HTTP request method와 정규식을 통한 URL 매핑(`urlPathMatching`)을 지원한다. 또한 mock의 우선순위(`atPriority`)를 정할 수 있다.

### 주요 메서드

`stubFor(mappingBuilder: *MappingBuilder*): StubMapping`

- mocking을 위해 미리 준비된 HTTP API를 작성을 하기 위한 메서드입니다.

`get(urlPattern: UrlPattern): MappingBuilder`

- HTTP 메서드 중 `GET`을 매핑하는 메서드, 이외에 `PUT`, `POST`, `DELETE`, `PATCH` 등 다양한 HTTP 메서드를 지원합니다.

`urlPathMatching(urlRegex: String): UrlPathPattern`

- **URL의 경로**가 정규식에 일치하는지를 확인합니다.

`urlMatching(urlRegex: String): UrlPattern`

- **URL의 경로와 쿼리 매개변수(query param)**까지 정규식에 일치하는지 확인합니다.

`MappingBuilder.atPriority(priority: Int)`

- URL 매칭의 우선순위를 정할 수 있습니다. 숫자가 작을 수록 우선순위가 높습니다. 해당 우선순위를 제공하지 않을 경우 기본적으로 `5`의 우선순위를 갖게 됩니다.

`MappingBuilder.withQueryParam(key: String, queryParamPattern: StringValuePattern)`

- 쿼리 매개변수를 설정합니다. 다수의 쿼리 매개변수가 필요한 경우 `withQueryParams` 메서드를 활용할 수 있습니다.
- 쿼리 매개변수 값에 `containing`, `matching`, `equalTo` 등 다양한 메서드를 활용하여 설정할 수 있습니다.

`MappingBuilder.*withRequestBody(bodyPattern: ContentPattern<?>)`

- 요청 본문을 설정할 수 있습니다.
- 본문 값에 `equalToJson` 와 `ObjectMapper`를 통해 객체를 설정할 수 있다.

`ResponseDefinitionBuilder.aResponse()`

- 응답을 정의하기 위한 빌더를 제공합니다.

`ResponseDefinitionBuilder.withStatus(status: Int)`

- 응답 코드를 설정할 수 있습니다.

`ResponseDefinitionBuilder.withHeader(key: String, values: String...)`

- 응답 헤더를 설정할 수 있습니다.

`ResponseDefinitionBuilder.withBody(body: String)`

- 응답 본문을 설정할 수 있습니다.

### GET

```kotlin
mockServer.stubFor(
  get(urlPathMatching("/api/another/nickname/\\w{4,12}$"))
    .atPriority(999) 
    .willReturn(
      aResponse()
        .withStatus(HttpStatus.OK.value())
        .withHeader("Content-Type", "application/json")
        .withBody(objectMapper.writeValueAsString(false))
    )
)
```

### PUT

```kotlin
mockServer.stubFor(
  put(urlPathMatching("/api/another/f0161e50-4428-4e36-a6e0-a35b63cdb3cf"))
    .atPriority(1)
    .withQueryParam("nickname", matching(".*"))
    .willReturn(
      aResponse()
        .withStatus(HttpStatus.OK.value())
        .withHeader("Content-Type", "application/json")
        .withBody(
          objectMapper.writeValueAsString(
            AnotherDTO(
              "f0161e50-4428-4e36-a6e0-a35b63cdb3cf",
              "홍길동",
              "yearnlune",
              LocalDateTime.of(2023, 9, 1, 0, 0, 0)
            )
          )
        )
    )
)

```

### POST

```kotlin
mockServer.stubFor(
  post(urlPathMatching("/api/another"))
    .atPriority(1)
    .withRequestBody(
      equalToJson(
        objectMapper.writeValueAsString(
          AnotherDTO(
            name = "raymond",
            nickname = "yearnlune"
          )
        )
      )
    )
    .willReturn(
      aResponse()
        .withStatus(HttpStatus.OK.value())
        .withHeader("Content-Type", "application/json")
        .withBody(
          objectMapper.writeValueAsString(
            AnotherDTO(
              "f0161e50-4428-4e36-a6e0-a35b63cdb3cf",
              "raymond",
              "yearnlune",
              LocalDateTime.of(2023, 9, 1, 0, 0, 0)
            )
          )
        )
    )
)
```

## Test

`@AutoConfigureWireMock`을 추가하여 WireMock을 적용한다.

```kotlin
@Suppress("SpringJavaInjectionPointsAutowiringInspection")
@SpringBootTest
@AutoConfigureWireMock
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
@ActiveProfiles("test")
class ExampleServiceTest @Autowired constructor(
  private val exampleService: ExampleService,
  private val wireMockServer: WireMockServer
) {

  @BeforeAll
  fun setup() {
    WireMockSupport.setupFeignClient(wireMockServer)
  }

  @Test
  @DisplayName("exampleFunctionA")
  fun exampleFunctionA_succeed() {
    /* GIVEN */
    val nickname = "yearnlune"

    /* WHEN */
    val result = exampleService.exampleFunctionA(nickname)

    /* THEN */
    assertThat(
      result.getOrThrow(),
      `is`("B")
    )
  }

  @Test
  @DisplayName("exampleFunctionB")
  fun exampleFunctionB_succeed() {
    /* GIVEN */
    val id = "f0161e50-4428-4e36-a6e0-a35b63cdb3cf"
    val nickname = "yearnlune"

    /* WHEN */
    val result = exampleService.exampleFunctionB(id, nickname)

    /* THEN */
    assertThat(
      result.getOrThrow(),
      `is`(
        AnotherDTO(
          "f0161e50-4428-4e36-a6e0-a35b63cdb3cf",
          "raymond",
          "yearnlune",
          LocalDateTime.of(2023, 9, 1, 0, 0, 0)
        )
      )
    )
  }
  
  @Test
  @DisplayName("exampleFunctionC")
  fun exampleFunctionC_succeed() {
    /* GIVEN */
    val another = AnotherDTO(
      name = "raymond",
      nickname = "yearnlune"
    )
  
    /* WHEN */
    val result = exampleService.exampleFunctionC(another)
  
    /* THEN */
    assertThat(
      result.getOrThrow(),
      `is`(
        AnotherDTO(
          "f0161e50-4428-4e36-a6e0-a35b63cdb3cf",
          "raymond",
          "yearnlune",
          LocalDateTime.of(2023, 9, 1, 0, 0, 0)
        )
      )
    )
  }
}
```

# 참고 문헌

[WireMock docs](https://wiremock.org/docs/)