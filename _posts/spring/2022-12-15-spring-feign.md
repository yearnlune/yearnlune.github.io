---
tags:
- feign
- spring-boot
- spring-cloud
- kotlin
title: Feign 
date: 2022/12/15
author: 김동환
description: openfeign 적용하기
disabled: false
categories:
- spring
---
# Feign

> Feign은 Netfilx에서 개발한 Http clinet binder로 보다 손쉽게 웹 서비스 클라이언트를 작성하고 호출 할 수 있다.
>

**사용 기술 및 라이브러리**

- JDK 11
- Kotlin 1.6.21
- Spring Boot 2.7.0
- Gradle 7.4.1
- Feign 3.1.5

# 따라해보기

```kotlin
dependencies {
    implementation("org.springframework.cloud:spring-cloud-starter-openfeign:3.1.5")
}
```

```kotlin
@EnableFeignClients
@SpringBootApplication
class FeignExampleApplication

fun main(args: Array<String>) {
    runApplication<FeignExampleApplication>(*args)
}
```

## 클라이언트

```kotlin
@FeignClient(
    name = "another",  // 서비스 이름
    url = "\${app.another-service}",  // 서비스 URL
    configuration = [FeignConfig::class]  // FeginClient에 적용할 configuration
)
interface AnotherFeignClient {

    @GetMapping(
        "/another/{anotherId}",
        headers = ["x-custom-header=just-test"]
    )
    fun findAnotherById(@PathVariable("anotherId") anotherId: String): AnotherDTO

    @GetMapping("/another/{anotherId}")
    fun findAnotherByIdWithHeaders(
        @RequestHeader header: HttpHeaders,
        @PathVariable("anotherId") anotherId: String
    ): AnotherDTO

    @GetMapping("/another")
    fun findAnotherList(): List<AnotherDTO>

    @PostMapping("/another")
    fun createAnother(another: AnotherDTO): AnotherDTO

    @PutMapping("/another/{anotherId}")
    fun updateAnotherStatus(@PathVariable("anotherId") anotherId: String): AnotherDTO
}
```

## 설정

Feign 클라이언트에 `configuration`  class로 정의하여 적용하거나 `yml` 을 통해 여러 설정을 정의할 수 있다.

```kotlin
// #1 Configuration class로 정의
class FeignConfig {

    /**
     * 요청 헤더 삽입
     */
    @Bean
    fun clientHeaderInterceptor(): RequestInterceptor {
        return RequestInterceptor {
            it.header("x-custom-header", "just-test")
        }
    }

    /**
     * 에러 처리
     */
    @Bean
    fun errorDecoder(): ErrorDecoder {
        return ErrorDecoder {_, response ->
            when (response.status()) {
                in 400..499 -> throw ResponseStatusException(HttpStatus.valueOf(response.status()))
                in 500..599 -> throw UnavailableServiceException()
                else -> throw RuntimeException()
            }
        }
    }

    /**
     * Feign 로깅 레벨 설정
     */
    @Bean
    fun loggerLevel(): Logger.Level = Logger.Level.FULL
}
```

```yaml
#2 yml로 정의
feign:
  client:
    config:
      another:  # FeignClient name
        request-interceptors:
          - io.github.yearnlune.feign.example.feign.FeignHeaderInterceptor
        error-decoder: io.github.yearnlune.feign.example.feign.FeignErrorDecoder
        logger-level: FULL
```

```kotlin
class FeignHeaderInterceptor : RequestInterceptor {

    override fun apply(template: RequestTemplate) {
        template.header("x-custom-header", "just-test")
    }
}

class FeignErrorDecoder : ErrorDecoder {

    override fun decode(methodKey: String, response: Response): Exception {
        when (response.status()) {
            in 400..499 -> throw ResponseStatusException(HttpStatus.valueOf(response.status()))
            in 500..599 -> throw UnavailableServiceException()
            else -> throw RuntimeException()
        }
    }
}
```

### 헤더

공통된 Header를 사용한다면 `RequestInterceptor`를 통해 공통 요청 헤더를 작성하여 적용한다. 그 외에 특정 API에 특정 헤더를 사용한다면 Annotation을 통해 적용할 수 있다.

- RequestInterceptor Interface
- Annotation

**RequestInterceptor**

`RequestInterceptor` 를 구현하고 이를 configuration나 yml에 적용한다.

```kotlin
class FeignHeaderInterceptor : RequestInterceptor {

    override fun apply(template: RequestTemplate) {
        template.header("x-custom-header", "just-test")
    }
}
```

**Annotation**

`@RequestMapping` 의 `header` 나 `@RequestHeader` 를 통해 적용한다.

> Feign의 @Headers를 통해 적용하려면 @RequestMapping이 아닌 Feign의 @RequestLine를 적용하여야 한다.
>

```kotlin
@FeignClient(
    name = "another",
    url = "\${app.another-service}",
    configuration = [FeignConfig::class]
)
interface AnotherFeignClient {

    @GetMapping(
        "/another/{anotherId}",
        headers = ["x-custom-header=just-test"]
    )
    fun findAnotherById(@PathVariable("anotherId") anotherId: String): AnotherDTO

    @GetMapping("/another/{anotherId}")
    fun findAnotherByIdWithHeaders(
        @RequestHeader header: HttpHeaders,
        @PathVariable("anotherId") anotherId: String
    ): AnotherDTO

}
```

### 오류 처리

`ErrorDecoder` 를 구현하여 적용할 수 있다. 적용된 FeignClinet의 API에 오류가 생기면 공통으로 처리 할 수 있다. 또한, 호출한 method(methodKey)에 따라 특정 API에 에러처리도 가능하다.

```kotlin
class FeignErrorDecoder : ErrorDecoder {

    override fun decode(methodKey: String, response: Response): Exception {
        when (response.status()) {
            in 400..499 -> throw ResponseStatusException(HttpStatus.valueOf(response.status()))
            in 500..599 -> throw UnavailableServiceException()
            else -> throw RuntimeException()
        }
    }
}
```

### 로깅

Feign을 통해 호출한 API의 로깅 수준을 정할 수 있다. 기본 로깅 정책은 NONE(로깅하지 않음) 이다.

- **BASIC** : 요청 메소드 및 URL, 응답 상태 코드 및 실행 시간
- **HEADERS** : BASIC + 요청 및 응답의 헤더
- **FULL** : 요청 및 응답의 본문, 헤더, 메타데이터

> Feign의 로깅은 `DEBUG` 레벨에서 동작한다.
>

```text
# LEVEL.BASIC
[AnotherFeignClient#findAnotherById] ---> GET http://localhost:8337/another/E001 HTTP/1.1
[AnotherFeignClient#findAnotherById] <--- HTTP/1.1 200 (9ms)

# LEVEL.HEADERS
[AnotherFeignClient#findAnotherById] ---> GET http://localhost:8337/another/E001 HTTP/1.1
[AnotherFeignClient#findAnotherById] x-custom-header: just-test
[AnotherFeignClient#findAnotherById] ---> END HTTP (0-byte body)
[AnotherFeignClient#findAnotherById] <--- HTTP/1.1 200 (10ms)
[AnotherFeignClient#findAnotherById] connection: keep-alive
[AnotherFeignClient#findAnotherById] content-type: application/json
[AnotherFeignClient#findAnotherById] date: Wed, 14 Dec 2022 06:44:14 GMT
[AnotherFeignClient#findAnotherById] keep-alive: timeout=60
[AnotherFeignClient#findAnotherById] transfer-encoding: chunked
[AnotherFeignClient#findAnotherById] <--- END HTTP (48-byte body)

# LEVEL.FULL
[AnotherFeignClient#findAnotherById] ---> GET http://localhost:8337/another/E001 HTTP/1.1
[AnotherFeignClient#findAnotherById] x-custom-header: just-test
[AnotherFeignClient#findAnotherById] ---> END HTTP (0-byte body)
[AnotherFeignClient#findAnotherById] <--- HTTP/1.1 200 (15ms)
[AnotherFeignClient#findAnotherById] connection: keep-alive
[AnotherFeignClient#findAnotherById] content-type: application/json
[AnotherFeignClient#findAnotherById] date: Wed, 14 Dec 2022 05:50:28 GMT
[AnotherFeignClient#findAnotherById] keep-alive: timeout=60
[AnotherFeignClient#findAnotherById] transfer-encoding: chunked
[AnotherFeignClient#findAnotherById] 
[AnotherFeignClient#findAnotherById] {"id":"E001","name":"홍길동","isActive":true}
[AnotherFeignClient#findAnotherById] <--- END HTTP (48-byte body)
```

# 마치며..

일반적으로 `RestTemplate` 을 활용하여 외부 API 호출 처리를 사용하곤 했다. RestTemplate을 활용하면서 API 호출 때마다 오류 처리 해줘야 하며, 다양한 외부 API 유지보수 측면에서 많은 불편함을 겪었다.

Feign은 이를 획기적으로 바꾸어 주었다. 인터페이스 선언과 간단한 설정으로  spring-cloud의 ribbon, hystrix등을 지원하고, 외부 API 호출의 통합 오류처리가 가능하고, 다양한 외부 API를 관리할 수 있게 되었다.

## 참고문헌

[feign github](https://github.com/OpenFeign/feign)

[spring-cloud-feign](https://cloud.spring.io/spring-cloud-netflix/multi/multi_spring-cloud-feign.html)