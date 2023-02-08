---
tags:
- spring-boot
- spring cache
- redis
- cache aside
- kotlin
title: redis를 활용한 간단한 스프링 캐싱 구현
date: 2023/02/08
author: 김동환
description: redis를 캐시로 활용하는 스프링부트 백엔드 서비스 구현하기
disabled: false
categories:
- spring
---

# 목표

**Redis를 캐시로 활용하는 스프링부트 백엔드 서비스**를 제공하는 것을 목표로 한다. 기본적인 캐시 정책은 [Cache-aside Pattern](https://yearnlune.github.io/general/cache-aside-pattern/) 를 활용한다. 이 프로젝트에서의 Redis는 서비스 품질에 영향을 끼치는 캐시의 역할만을 하기에 Redis에 이슈가 생기더라도 서비스 장애가 생기면 안된다. 그리하여, Redis에 이슈가 생기더라도 DB를 통한 서비스 제공을 기본으로 한다.

# 환경

- JDK 11
- spring-boot 2.7.8
- Kotlin 1.6.21
- Gradle 7.6


## 예시 프로젝트

[예시 프로젝트-github](https://github.com/yearnlune/redis-cache-example)

# Spring Cache

## Configuration

### Spring Cache 설정

`CACHE_REDIS_ENABLED`가 `true`인 경우에만 `@EnableCaching`을 통해 캐시를 사용할 수 있게 처리한다. 캐시 쪽에서 Exception이 생기면 로깅은 하되 전파하지 않게 처리한다.

```kotlin
@Configuration
@ConditionalOnProperty(name = ["cache.redis.enabled"], havingValue = "true")
@EnableCaching
class RedisCacheConfig(
  val redisConnectionFactory: RedisConnectionFactory
) : CachingConfigurerSupport(), CachingConfigurer {

  // Redis 캐시 정책 설정
  fun redisCacheConfiguration() = RedisCacheConfiguration.defaultCacheConfig()
    .disableCachingNullValues()  // 널 값은 캐싱하지 않음
    .serializeKeysWith(  // Redis key Serialize 정책 설정
      RedisSerializationContext.SerializationPair.fromSerializer(StringRedisSerializer())
    )
    .serializeValuesWith(  // Redis value Serialize 정책 설정
      RedisSerializationContext.SerializationPair.fromSerializer(
        GenericJackson2JsonRedisSerializer(objectMapper)
      )
    )
    .prefixCacheNameWith(CACHE_PREFIX)  // Redis key prefix 기본 값 설정
    .entryTtl(Duration.ofSeconds(CACHE_TTL))  // Redis TTL 기본 값 설정

  // 캐시 적용
  override fun cacheManager(): CacheManager = RedisCacheManager.RedisCacheManagerBuilder
    .fromConnectionFactory(redisConnectionFactory)
    .cacheDefaults(redisCacheConfiguration())
    .build()

  // Exception 정책 설정
  // Exception일 때 기본 정책은 `throw Exception`이나, Exception이 생기면 missed 처리하기 위함
  override fun errorHandler(): CacheErrorHandler {
    return object : CacheErrorHandler {
      override fun handleCacheGetError(exception: RuntimeException, cache: Cache, key: Any) {
        logger().warn("[REDIS_CACHE:GET:${cache.name}]: ${exception.message}")
      }

      override fun handleCachePutError(exception: RuntimeException, cache: Cache, key: Any, value: Any?) {
        logger().warn("[REDIS_CACHE:PUT:${cache.name}]: ${exception.message}")
      }

      override fun handleCacheEvictError(exception: RuntimeException, cache: Cache, key: Any) {
        logger().warn("[REDIS_CACHE:EVICT:${cache.name}]: ${exception.message}")
      }

      override fun handleCacheClearError(exception: RuntimeException, cache: Cache) {
        logger().warn("[REDIS_CACHE:CLEAR:${cache.name}]: ${exception.message}")
      }
    }
  }
}
```

### Redis 설정

Redis에 대한 기본 정보를 설정한다. 캐싱을 사용하지 않으면 Redis에 대한 연결도 필요 없기에 `@SpringBootApplication(exclude = [RedisAutoConfiguration::*class*])`를 처리하고, 캐싱을 사용할 때 `@Import(RedisAutoConfiguration::class)` 할 수 있게 처리한다.

```kotlin
@Configuration
@ConditionalOnProperty(name = ["cache.redis.enabled"], havingValue = "true")
@ConfigurationProperties(prefix = "spring.redis")
@Import(RedisAutoConfiguration::class)
class RedisConfig {
  var host: String = "redis"
  var port: Int = 6379
  var password: String? = null

  @Bean
  fun clientOptions(): ClientOptions = ClientOptions.builder()
    .disconnectedBehavior(ClientOptions.DisconnectedBehavior.REJECT_COMMANDS)
    .timeoutOptions(TimeoutOptions.enabled(Duration.ofSeconds(1)))
    .build()

  @Bean
  fun lettucePoolingClientConfiguration(clientOptions: ClientOptions) =
    LettuceClientConfiguration.builder()
      .clientOptions(clientOptions)
      .clientResources(DefaultClientResources.create())
      .build()

  @Bean
  fun redisStandaloneConfiguration() = RedisStandaloneConfiguration(host, port)
    .apply {
      if (this@RedisConfig.password != null) {
        this.password = RedisPassword.of(this@RedisConfig.password)
      }
    }

  @Bean
  fun redisConnectionFactory(
    redisStandaloneConfiguration: RedisStandaloneConfiguration,
    lettucePoolingClientConfiguration: LettuceClientConfiguration
  ): RedisConnectionFactory = LettuceConnectionFactory(redisStandaloneConfiguration, lettucePoolingClientConfiguration)

  @Bean
  fun redisTemplate(redisConnectionFactory: RedisConnectionFactory): RedisTemplate<String, Any> {
    val template = object : RedisTemplate<String, Any>() {
      override fun <T : Any?> execute(  // redisTemplate을 활용한 캐싱할 경우 Exception이 생기면 null을 통한 missed 처리하기 위함
        action: RedisCallback<T>,
        exposeConnection: Boolean,
        pipeline: Boolean
      ): T? {
        return runCatching {
          super.execute(action, exposeConnection, pipeline)
        }.getOrElse {
          logger().warn(it.localizedMessage)
          null
        }
      }
    }.apply {
      this.isEnableDefaultSerializer = false
      this.keySerializer = StringRedisSerializer()
      this.valueSerializer = GenericJackson2JsonRedisSerializer(CacheConfig.objectMapper)
    }

    template.setConnectionFactory(redisConnectionFactory)
    return template
  }
}
```

## Caching

### Annotation

`@Cacheable`, `@CachePut`, `@cacheEvict` 등 spring의 annotation을 활용하여 캐싱을 처리할 수 있다. 관련 문서는 [Spring Cache](https://docs.spring.io/spring-framework/docs/3.2.x/spring-framework-reference/html/cache.html)에서 확인할 수 있다. 단일의 데이터 경우 활용하기 좋으나, 다수의 데이터를 처리하면 의도치 않게 캐싱될 수 있다.

**@Cacheable**

```kotlin
// 1. Redis에 해당 Key가 존재 할 경우 바로 캐시에서 가져와 사용한다.
// 2. Redis에 해당 key가 존재 하지 않을 경우 DB를 통해 데이터를 가져와 캐싱한다.
@Cacheable(cacheNames = ["user:summary"], key = "#id")
fun findUserSummary(id: UUID36): UserDTO.Summary = 
	userRepository.findById(id).orElseThrow().toSummary()
```

**@CachePut**

```kotlin
// Redis에 해당 key 유무와 상관없이 캐싱한다.
@Transactional
@CachePut(cacheNames = ["user:summary"], key = "#id")
fun updateUser(id: UUID36, userSummary: UserDTO.Summary): UserDTO.Summary {
  val user = userRepository.findById(id).orElseThrow()
  return user.setWithUserSummary(userSummary).toSummary()
}
```

**@CacheEvict**

```kotlin
// Redis에 해당 Key를 제거한다.
@Transactional
@CacheEvict(cacheNames = ["user:summary"], key = "#id")
fun deleteUser(id: UUID36) {
  val user = userRepository.findById(id).orElseThrow()
  user.deleted = true
}
```

### RedisTemplate

RedisTemplate을 통해 직접 구현도 가능하다. 다수의 데이터를 캐싱 처리를 할 때 사용하면 좋다. 찾고자 하는 ID 목록을 우선 cache에서 확인하고 missed ID 목록을 DB에서 처리하고, AddCacheEvent 스프링 이벤트를 통해 캐싱을 처리한다. 아래 예시에서는 TTL 설정을 위해 파이프라인을 활용하여 저장하였다.

```kotlin
fun findUserSummariesWithUserIds(ids: Set<UUID36>) = 
  cacheService.findCollectionByIdsWithCache(
    ids, 
    CacheConfig.USER_KEY_PREFIX, 
    userRepository::findUsersByIdIn
  ).filterIsInstance<UserDTO.Summary>()
```

```kotlin
data class AddCacheEvent(
  val prefix: String,
  val values: Map<String, Any>
)

fun findCollectionByIdsWithCache(
  ids: Set<String>,
  cachePrefix: String,
  findFunction: (List<String>) -> List<CacheBase>
): List<CacheBaseDTO> {
  val cachedMap = runCatching {
    redisTemplate?.opsForValue()?.multiGet(ids.map { cachePrefix + it })
      ?.filterIsInstance<CacheBaseDTO>()
      ?.associateBy { it.id } ?: emptyMap()
  }.getOrElse {
    logger().error(it.localizedMessage)
    emptyMap()
  }

  val missedIds = ids
    .filter { cachedMap[it] == null }
    .also { logger().info("[$cachePrefix] Missed ID: [${it.size}]") }
  val missedCollection = if (missedIds.isNotEmpty()) findFunction(missedIds) else emptyList()

  return (cachedMap.values.toList() + missedCollection.map(CacheBase::toCacheBaseDTO))
    .also {
      if (missedIds.isNotEmpty()) {
        eventPublisher.publishEvent(
          AddCacheEvent.create(
            cachePrefix,
            missedCollection.map(CacheBase::toCacheBaseDTO).associateBy { it.id }
          )
        )
      }
    }
}

@Async
@EventListener
fun handleAddCache(addCacheEvent: AddCacheEvent) {
  runCatching {
    pushWithPipeline(addCacheEvent)
  }
}

private fun pushWithPipeline(addCacheEvent: AddCacheEvent) {
  val key = redisTemplate?.keySerializer as RedisSerializer<String>
  val value = redisTemplate?.valueSerializer as RedisSerializer<Any>
  redisTemplate?.executePipelined { connection ->
    addCacheEvent.values.forEach {
      connection.stringCommands().mSet(
        mapOf(key.serialize(addCacheEvent.prefix + it.key)!! to value.serialize(it.value)!!)
      )
      connection.expire(key.serialize(addCacheEvent.prefix + it.key)!!, CACHE_TTL)
    }
  }
}
```

# 추가 설정

## Health Check 제외

spring-actuator를 통해 health check를 할 때 Redis의 장애를 예외 처리 한다.

```yaml
management:
  health:
    redis:
      enabled: false
```

```json
{
  "status": "UP",
  "components": {
    "db": {
      "status": "UP",
      "details": {
        "database": "H2",
        "validationQuery": "isValid()"
      }
    },
    "diskSpace": {
      "status": "UP",
      "details": {
        "total": 1024208138240,
        "free": 342819975168,
        "threshold": 10485760,
        "exists": true
      }
    },
    "ping": {
      "status": "UP"
    }
  }
}
```