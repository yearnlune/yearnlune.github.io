---
tags: 
    - cache aside pattern
    - cache
title: Cache aside pattern
date: 2022/12/30
author: 김동환
description: cache aside pattern
disabled: false
categories:
  - general
---

# Cache aside pattern

> 필요할 때 캐시에 데이터를 적재하는 패턴이다.
>

![Cache-aside-pattern](/assets/images/cache-aside-pattern/cache-aside-pattern.png)

다양한 캐싱 전략 중 널리 사용되고 있는 패턴 중 하나이다. 데이터의 수요에 따라 캐싱하기에 효율적으로 캐시를 적재 및 유지할 수 있다. 또한 Cache에 장애가 발생하더라도 DB에서 데이터를 제공할 수 있다.

1. 애플리케이션에서 데이터의 수요가 발생하면 캐시에 먼저 데이터를 찾아 제공한다.
2. 캐시에 해당 데이터가 존재하지 않으면 DB에서 찾아 제공한다.
3. DB에서 찾은 데이터를 캐시에 적재한다.

# 고려사항

캐시에 이미 존재하는 데이터는 DB와 상관없이 제공하기에, DB와 캐시의 동기화가 정확하게 이뤄져야한다.

# 참고문헌

[https://learn.microsoft.com/en-us/azure/architecture/patterns/cache-aside](https://learn.microsoft.com/en-us/azure/architecture/patterns/cache-aside)

[https://docs.aws.amazon.com/whitepapers/latest/database-caching-strategies-using-redis/caching-patterns.html](https://docs.aws.amazon.com/whitepapers/latest/database-caching-strategies-using-redis/caching-patterns.html)

[https://hazelcast.com/blog/a-hitchhikers-guide-to-caching-patterns/](https://hazelcast.com/blog/a-hitchhikers-guide-to-caching-patterns/)