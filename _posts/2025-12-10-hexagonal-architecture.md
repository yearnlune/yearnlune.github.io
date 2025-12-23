---
tags: 
    - hexagonal architecture
    - architecture
    - ddd
    - domain driven design
    - kotlin
    - spring boot
title: "헥사고날 아키텍처, 어디까지 적용해야 할까"
date: 2025/12/10
author: 김동환
description: 실무 프로젝트에서의 선택과 트레이드오프
disabled: false
categories:
    - general
---

> 이 글은 “헥사고날 아키텍처를 어떻게 완벽히 구현했는가”가 아니라,  
> **“현실적인 제약 속에서 어떤 원칙을 지켰는가”에 대한 기록이다.**

## 들어가며

프로젝트를 진행하다 보면 아키텍처에 대한 고민이 끊이지 않는다.  
마이크로서비스는 과한가? 레이어드 아키텍처면 충분한가?  
DDD는 어디까지 적용해야 하는가?

헥사고날 아키텍처(Hexagonal Architecture)를 완전하게 구현하기보다는,  
실제 서비스 환경에서 **어디까지 적용하는 것이 합리적인지** 고민하며 선택한  
구조와 설계 원칙을 공유하고자 한다.

단순히 이론을 나열하기보다는,  
개발 과정에서 실제로 부딪힌 문제와 그에 대한 선택,  
그리고 **왜 그렇게 설계했는지에 대한 이유**를 중심으로 정리했다.

## 기본 구조: 헥사고날 아키텍처 + 도메인 모듈화

본 프로젝트는 헥사고날 아키텍처를 **지향**하되,  
모놀리식 환경에서 과도한 추상화를 피하고  
**도메인 경계와 의존성 방향을 우선적으로 지키는 방식**을 선택했다.

이 설계에서 가장 중요하게 둔 기준은 다음 세 가지였다.

1. **도메인 규칙이 기술 변화에 휘둘리지 않도록 보호할 것**
2. **변경 가능성이 낮은 지점에는 불필요한 추상화를 두지 않을 것**
3. **구조 자체보다, 팀이 이해하고 유지할 수 있는 설명 가능한 구조일 것**

이 기준을 바탕으로 헥사고날 아키텍처의 모든 요소를 적용하기보다는,  
실제 프로젝트 맥락에 맞는 부분만 선택적으로 가져왔다.

### 레이어 구조

각 도메인은 다음 4개의 레이어로 구성된다.

```
{domain}/
├── presentation/    # 외부 인터페이스 (REST Controller, DTO)
├── application/     # 유스케이스, 비즈니스 플로우 조율
├── domain/          # 핵심 비즈니스 로직, 엔티티, 도메인 서비스
└── infrastructure/  # 기술적 구현 (DB, Repository, 외부 API)
```

이 구조는 전형적인 포트/어댑터를 모두 분리한 형태의 헥사고날은 아니다.  
대신 레이어드 아키텍처를 기본으로 하되,  
**의존성 방향과 도메인 보호라는 헥사고날의 핵심 원칙만을 선택적으로 적용**했다.

### 핵심 원칙

- `domain` 레이어는 어떤 레이어에도 의존하지 않는다
- `application`은 도메인을 조합해 유스케이스를 완성한다
- `presentation`은 application만 호출한다
- `infrastructure`는 기술적 세부 사항을 담당한다

헥사고날 아키텍처의 핵심은  
**완벽한 포트 분리가 아니라, 의존성 방향과 경계의 명확성**이라고 생각한다.

## 도메인 모듈 분리

기능 단위로 도메인을 분리하는 방식을 선호한다.

- **user**: 인증, 계정 관리
- **product**: 상품 정보 관리
- **order**: 주문 관리
- **global**: 공통 설정, 보안, 예외 처리

각 도메인은 독립적으로 진화할 수 있으며,  
서로는 가능한 한 **약한 결합**으로 연결된다.

## 도메인 서브 구조: 복잡성을 관리하는 방법

도메인이 커지면 `domain/` 하위에 **개념적으로 분리되는 서브 도메인**을 둔다.

예를 들어 주문 도메인:

```
order/
└── domain/
    ├── order/         # 주문 (Order, OrderItem)
    └── payment/       # 결제 (Payment, Refund)
```

이 구조의 장점은 다음과 같다.

- 개념적으로 다른 책임을 명확히 분리할 수 있다
- 코드 응집도가 높아진다
- 애그리게잇 경계를 자연스럽게 표현할 수 있다

단, 구조적 분리를 위한 분리는 지양하고  
**실제로 다른 비즈니스 규칙과 라이프사이클을 가지는 경우에만 분리**한다.

## Repository 패턴: 실용적인 접근

본 프로젝트에서는 **Repository 인터페이스를 domain 레이어에 두지 않았다.**

### 왜 domain에 Repository 인터페이스를 두지 않았는가?

인프라(DB, ORM)의 변경은 이론적으로는 가능하지만,  
실제 프로젝트에서는 **거의 발생하지 않는 변화**이기 때문이다.

특히 이 프로젝트는 단일 팀이 장기간 소유하며,  
DB 기술을 교체할 가능성이 매우 낮은 서비스라는 점도  
이 판단에 영향을 주었다.

DB 변경은 단순히 Repository 구현을 교체하는 문제가 아니라,

- 쿼리 구조
- 인덱스 전략
- 트랜잭션 경계
- 성능 튜닝

까지 포함한 **대규모 수정**을 수반한다.

실제로 구현 초반에는 DDD 교과서처럼 Repository 인터페이스를 domain에 두는 구조도 시도했지만,
몇 개의 유스케이스를 구현하는 과정에서 이 추상화가 실질적인 이점을 주지는 않는다고 느꼈다.

Repository 인터페이스 하나로 이 변경을 흡수할 수 있다고 보지 않았고,  
오히려 불필요한 추상화가 될 가능성이 높다고 판단했다.

따라서 Repository를 **비즈니스 규칙이 아닌 기술적 구현**으로 간주하고,  
`infrastructure` 레이어에 실용적으로 구현했다.

```kotlin
@Repository
class OrderRepository(
    private val jpa: JpaOrderRepository,
) {
    fun save(order: Order): Order = jpa.save(order)
    fun findByIdOrNull(id: Long): Order? = jpa.findByIdOrNull(id)
    fun findByUserId(userId: Long): List<Order> = jpa.findByUserId(userId)
}
```

이는 헥사고날 아키텍처를 부정한 것이 아니라,  
**모놀리식 환경에서 변경 가능성이 낮은 지점에 대한 의도적인 트레이드오프**다.

이 선택의 대가는,  
도메인 레이어가 영속성 기술(JPA)에 대해 완전히 무지하지는 않다는 점이다.  
하지만 이 프로젝트에서는 그 비용보다  
불필요한 추상화를 줄이고 구조를 단순하게 유지하는 이점이 더 크다고 판단했다.

## 애그리게잇 분리 전략: 트래픽 패턴을 고려한 설계

DDD에서 애그리게잇은 **하나의 트랜잭션으로 일관성을 보장하는 단위**다.  
그러나 모든 엔티티를 하나의 애그리게잇으로 묶을 필요는 없다.

### 트래픽 패턴의 차이

예를 들어 `Project`와 `Task`가 있다고 가정하면:

- Task 상태 변경: 매우 빈번
- Project 메타 변경: 드묾

이를 하나의 애그리게잇으로 묶으면  
불필요한 락 경합과 성능 저하가 발생할 수 있다.

### 트랜잭션 경계 분리

실제로 Project 이름 변경과 Task 생성은  
같은 트랜잭션으로 묶일 필요가 없다.

### 결론

- 서로 다른 트래픽 패턴
- 다른 트랜잭션 경계
- 독립적인 최적화 필요성

이런 경우 **애그리게잇을 분리하고 약한 결합으로 연결**하는 것이 합리적이다.

```kotlin
class Task(
    val projectId: Long,   // 약한 결합
    val assigneeId: Long,  // 약한 결합
)
```

## 도메인 간 결합: 약한 결합의 원칙

다른 바운더리 컨텍스트의 엔티티를 참조할 때는  
**항상 ID로만 참조**한다.

### 같은 애그리게잇 내부 (강한 결합)

```kotlin
class OrderItem(
    val order: Order,     // 같은 애그리게잇
    val productId: Long,  // 다른 도메인
)
```

### 다른 바운더리 컨텍스트 (약한 결합)

```kotlin
class Order(
    val userId: Long,
    val paymentId: Long,
)
```

## UseCase의 역할: 단일 진입점

처음에는 단순히 Service 계층으로 충분하지 않을까 고민했지만,
비즈니스 흐름이 길어질수록 호출 규칙을 강제할 필요성을 느끼게 되었다.

이 글에서의 UseCase는 
**비즈니스 흐름을 조율하는 오케스트레이션 단위**이며,  
CommandService / QueryService는  
**도메인 간 통신을 위한 제한된 인터페이스**에 가깝다.

이 프로젝트는 MSA 환경처럼 도메인이 강하게 분리된 구조가 아니라,  
모놀리식 구조 안에서 도메인 경계와 의존성 방향을 관리하는 것을 목표로 했다.  
그 과정에서 이벤트 기반 통신 역시 내부적으로 사용하고 있지만,  
이 글에서는 헥사고날 아키텍처의 구조적 설명에 집중하기 위해  
의도적으로 다루지 않았다.

이러한 맥락에서  
**명시적인 호출 구조를 유지하면서도 경계를 지킬 수 있는 방법**으로  
CQRS 패턴을 UseCase 중심의 진입 구조로 우선 선택했다.

- Controller / Scheduler / Listener → 항상 UseCase만 호출
- UseCase는 다음을 할 수 있음:
  - 같은 도메인의 다른 UseCase 호출
  - 다른 도메인의 QueryService / CommandService 호출
- ❌ 타 도메인의 UseCase 직접 호출 금지

## 도메인 간 통신: CQRS 패턴

다른 도메인의 정보가 필요할 때는 **CQRS 서비스만 허용**한다.

### 호출 경로

```
Controller → UseCase → (같은 도메인 다른 UseCase / 다른 도메인 QueryService/CommandService)
```

### QueryService 규칙

- 반환: Snapshot / View / Summary
- JPA 연결 ❌ (더티 체킹 방지)
- 엔티티 반환 ❌
- 외부 도메인 호출 허용 ✅

```kotlin
@Service
class UserQueryService(
    private val userRepository: UserRepository,
) {
    fun getUserById(userId: Long): UserSnapshot {
        val user = userRepository.findByIdOrNull(userId)
            ?: throw UserNotFoundException(userId)

        return UserSnapshot(
            id = user.id,
            name = user.name,
            email = user.email,
        )
    }
}
```

### CommandService 규칙

- 입력: ~Command
- 출력: ~Result / id / Unit
- 엔티티 파라미터 ❌
- 엔티티 반환 ❌
- 엔티티 조작은 내부에서만

```kotlin
@Service
class UserCommandService(
    private val userRepository: UserRepository,
) {
    fun createUser(command: CreateUserCommand): CreateUserResult {
        val user = User(command.name, command.email)
        val saved = userRepository.save(user)
        return CreateUserResult(saved.id)
    }
}
```

### 내부 도메인에서도 CQRS 사용 가능

같은 도메인 안에서도 CommandService / QueryService 분리 사용 가능하다.

## 마무리: 균형 있는 설계

돌이켜보면 모든 선택이 처음부터 명확했던 것은 아니다.  
구현을 진행하며 여러 번 구조를 되돌려보고,  
일부는 과감히 포기하기도 했다.

그 과정에서 중요하다고 느낀 원칙은 다음과 같다.

- 이론에 매몰되지 않기
- 변경 가능성이 높은 지점에만 추상화 적용하기
- 도메인 경계와 책임을 명확히 하기
- 실용성과 확장성의 균형 잡기

이 글에서는 메시지 기반 통신, 완전한 애그리게잇 간 이벤트 연동 등은  
복잡도에 비해 현재 단계에서의 필요성이 낮다고 판단하여  
의도적으로 다루지 않았다.

이 구조가 항상 옳다고 생각하지는 않는다.  
다만 이 시점의 요구사항과 팀 구성, 서비스 성격을 기준으로 볼 때  
가장 설명 가능하고 유지 가능한 선택이었다고 판단했다.
