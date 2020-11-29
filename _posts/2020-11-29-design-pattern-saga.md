---
tags: 
    - saga
    - pattern
    - MSA
title: SAGA Pattern
date: 2020/11/29
author: 김동환
description:  SAGA Design Pattern
disabled: false
categories:
  - General
---

> MSA(Microservices architecture)에서 트랜잭션을 처리하는 방법으로 각 서비스당 로컬 트랜잭션을 진행하며 메시지 또는 이벤트를 통해 다음 트랜잭션 단계로 진행하며, 다음 단계가 실패하면 보정 트랜잭션(rollback) 실행

![/assets/images/design-pattern-saga/saga-overview.png](/assets/images/design-pattern-saga/saga-overview.png)

## Choreography Based SAGA

![/assets/images/design-pattern-saga/choreography-pattern.png](/assets/images/design-pattern-saga/choreography-pattern.png)

### 정의

중앙 제어 서비스 없이 각 서비스마다 로컬 트랜잭션을 처리하며 `이벤트` 혹은  `메시지`를 통해 통합 트랜잭션 처리를 통해 데이터 정합성을 처리

### 예시

![/assets/images/design-pattern-saga/saga-rollback-operation.png](/assets/images/design-pattern-saga/saga-rollback-operation.png)

### 장점

간단하게 구현할 수 있음

### 단점

서비스들이 어떠한 메시지나 이벤트를 수신 대기 하는 지 추적하기 어려움

트랜잭션의 진행 상태를 알기 어려움

## Orchestration Based SAGA

![/assets/images/design-pattern-saga/orchestrator.png](/assets/images/design-pattern-saga/orchestrator.png)

### 정의

중앙 오케스트레이터가 각 서비스들의 통합 트랜잭션을 관리 하는 것으로 트랜잭션 및 수행할 상태를 관리하여 통합 트랜잭션을 처리

### 장점

`Orchestrator`가 관리 하기 때문에 중앙 집중화 시킬 수 있다.

### 단점

`Orchestrator` 서비스가 필요함

트랜잭션 관련된 서비스들을 `Orchestrator` 가 관리하기 때문에 복잡해질 수 있음