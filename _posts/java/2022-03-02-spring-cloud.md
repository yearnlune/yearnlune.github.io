---
tags: 
    - java
    - spring cloud
    - msa
title: Spring Cloud
date: 2022/03/02
author: 김동환
description: 스프링 클라우드란
disabled: true
categories:
  - java
---
# Spring Cloud ?

> 분산 시스템 환경에서 사용되는 여러 기능 및 공통 패턴을 제공하는 umbrella project이며, 빠르고 쉽게 적용할 수 있다.
>

마이크로 서비스의 시스템을 구축할 경우 다양한 문제의 직면한다. 유연한 서비스 구축, 서비스의 발견, 서비스 간 통신 등 모노리틱 시스템과 다른 문제 해결 능력이 필요하다. 이러한 문제를 Spring cloud 프레임워크를 사용하여 쉽고 빠르게 해결할 수 있다.

## 주요 기능

### Distributed configuration

마이크로 서비스 환경에는 수많은 서비스와 설정 파일이 존재한다. 그 과정에서 다양한 설정 파일을 적용이 가능해야하며, 용이한 관리가 필요하게 된다. Spring cloud는 설정 값 변경에 따른 불필요한 빌드와 배포를 줄이기 위해서는 애플리케이션과 설정 파일의 분리하고, 이를 중앙에 집중화 하여 용이한 관리를 제공해준다.

### Service discovery

시시각각으로 바뀌는 마이크로 서비스 환경에서는 호스트 이름과 포트 번호를 통한 접근이 유연하지 않을 수 있다. 그렇기에 운영중 서비스를 쉽게 찾고 접근 가능해야 한다. Spring cloud는 `Eureka` 를 통해 service discvoery 기능을 제공한다.

# 참고자료

**[Spring Cloud](https://spring.io/projects/spring-cloud)**

**[The Beginner’s Guide To Spring Cloud - Ryan Baxter](https://youtu.be/aO3W-lYnw-o)**