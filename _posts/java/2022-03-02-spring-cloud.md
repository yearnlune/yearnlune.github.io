---
tags: 
    - java
    - spring cloud
    - MSA
title: Spring Cloud
date: 2022/03/02
author: 김동환
description: 스프링 클라우드 주요 기능
disabled: false
categories:
  - java
---
# Spring Cloud ?

> 분산 시스템 환경에서 사용되는 여러 기능 및 공통 패턴을 제공하는 umbrella project이며, 빠르고 쉽게 적용할 수 있다.
>

마이크로 서비스의 시스템을 구축할 때 다양한 문제에 직면한다. 유연한 서비스 구축, 서비스의 발견, 서비스 간 통신 등 모노리틱 시스템과 다른 문제 해결 능력이 필요하다. 이러한 문제를 Spring cloud 프레임워크를 사용하여 쉽고 빠르게 해결할 수 있다.

## 주요 기능

### Distributed configuration

마이크로 서비스 환경에는 수많은 서비스와 설정 파일이 존재한다. 그 과정에서 다양한 설정 파일을 적용이 가능해야 하며, 쉬운 관리가 필요하게 된다. Spring cloud는 설정값 변경에 따른 불필요한 빌드와 배포를 줄이기 위해서는 애플리케이션과 설정 파일의 분리하고, 이를 중앙에 집중화하여 쉬운 관리를 제공해준다.

### Service discovery

시시각각으로 바뀌는 마이크로 서비스 환경에서는 호스트 이름과 포트 번호를 통한 접근이 유연하지 않을 수 있다. 그렇기에 운영 중 서비스를 쉽게 찾고 접근할 수 있어야 한다. Spring cloud는 `eueka`나 `zookeeper`, `consul` 을 통해 service discovery 기능을 제공한다.

### Routing

마이크로 서비스 환경에서 클라이언트와 해당 서비스와의 통신이 복잡해질 수 있다. 다양한 클라이언트의 API 처리를 위해 안전하고 유연한 게이트웨이를 사용한다. 게이트웨이를 통해서 외부로 노출되는 엔드포인트를 줄이고 통합하여 다수의 엔드포인트를 관리한다. `zuul` 혹은 `spring cloud gateway(SCG)`를 통해 게이트웨이 기능을 제공한다.



### Messaging

마이크로 서비스 환경에서 서비스간 통신을 HTTP 혹은 비동기식 분산 메시징을 사용한다. `ribbon` 혹은 `spring clound loadbalancer`를 통해 로드밸런서를 통한 서비스 인스턴스간 통신을 할 수 있다. 또한 스프링 클라우드에서는 `rabbitMQ` 혹은 `kafka` 와 같은 분산 메시징을 쉽게 적용할 수 있으며, `spring cloud stream` 을 통해서 메시지(이벤트) 중심의 서비스를 구축할 수 있다.

### Circuit breakers

서비스 장애 발생 시 이를 회피할 수 있는 circuit breaker를 제공한다. 마이크로 서비스 환경에서 일부 서비스에 이슈가 생기면 전체 서비스에 부하를 줄 수 있다. 그렇기에 장애가 발생한 서비스의 요청을 중단시켜야 한다. `hystrix` 혹은 `resilience4j` 등을 통해 쉽게 장애 회피를 할 수 있다.

　
# 참고자료

**[Spring Cloud](https://spring.io/projects/spring-cloud)**

**[The Beginner’s Guide To Spring Cloud - Ryan Baxter](https://youtu.be/aO3W-lYnw-o)**