---
tags: 
    - hexagonal architecture
    - architecture
    - ddd
    - domain driven design
    - kotlin
    - spring boot
title: "내가 선호하는 헥사고날 아키텍처 구조: 실전 경험에서 얻은 인사이트"
date: 2025/12/10
author: 김동환
description: 실제 프로젝트를 통해 경험한 헥사고날 아키텍처 기반의 구조와 그 설계 원칙에 대해 공유
disabled: false
categories:
    - general
---

## 들어가며

프로젝트를 진행하다 보면 아키텍처에 대한 고민이 끊이지 않는다. 마이크로서비스가 과한가? 레이어드 아키텍처면 충분한가? DDD를 어디까지 적용해야 하는가? 

이번 글에서는 실제 프로젝트를 통해 경험한 헥사고날 아키텍처 기반의 구조와 그 설계 원칙에 대해 공유하고자 한다. 단순히 이론만 나열하는 것이 아니라, 실제 개발 과정에서 부딪힌 문제와 해결 방법을 담아보았다.

## 기본 구조: 헥사고날 아키텍처 + 도메인 모듈화

프로젝트는 헥사고날 아키텍처(Hexagonal Architecture)를 기반으로 하되, 도메인별로 모듈을 분리하는 방식을 채택했다.

### 레이어 구조

각 도메인은 다음 4개의 레이어로 구성된다:

```
{domain}/
├── presentation/    # 외부 인터페이스 (REST Controller, DTO)
├── application/     # 유스케이스, 비즈니스 플로우 조율
├── domain/          # 핵심 비즈니스 로직, 엔티티, Repository 인터페이스
└── infrastructure/  # 기술적 구현 (Repository 구현체, DB, 외부 API)
```

**핵심 원칙:**
- `domain` 레이어는 어떤 레이어에도 의존하지 않는다 (순수 비즈니스 로직)
- `infrastructure`는 `domain`의 인터페이스를 구현한다 (의존성 역전)
- `application`은 `domain`을 사용하여 유스케이스를 조율한다
- `presentation`은 `application`을 호출한다

### 도메인 모듈 분리

기능별로 도메인 모듈을 분리하는 것을 선호한다. 예를 들어:

- **product**: 상품 정보 관리
- **order**: 주문 관리
- **user**: 사용자 인증, 계정 관리
- **global**: 공통 설정, 보안, 예외 처리

각 도메인은 독립적으로 진화할 수 있으며, 서로 약한 결합으로 연결된다.

## 도메인 서브 구조: 복잡성을 관리하는 방법

도메인이 복잡해지면 `domain/` 하위에 서브 도메인으로 분리한다. 예를 들어:

```
order/
└── domain/
    ├── order/         # 주문 관련 엔티티 (Order, OrderItem)
    └── payment/       # 결제 관련 엔티티 (Payment, Refund)
```

이렇게 하면:
- 각 서브 도메인은 독립적인 엔티티와 Value Object를 가질 수 있다
- 코드 응집도가 높아진다
- 서브 도메인 간의 경계가 명확해진다

하지만 무작정 분리하지 말고, 실제로 개념적으로 분리되는 단위로 나누는 것이 중요하다.

## Repository 패턴: 실용적인 접근

Repository는 모든 구현을 `infrastructure` 레이어에 둔다. 이론적으로는 인터페이스를 `domain`에 두고 구현을 `infrastructure`에 두는 것이 이상적이지만, 실제로는 오버엔지니어링이 될 수 있다.

### 구조

```kotlin
// infrastructure/order/OrderRepository.kt
@Repository
class OrderRepository(
    private val jpa: JpaOrderRepository,
) : BaseRepository<Order> {
    override fun save(entity: Order): Order = jpa.save(entity)
    override fun findByIdOrNull(id: Long): Order? = jpa.findByIdOrNull(id)
    
    fun findByUserId(userId: Long): List<Order> = jpa.findByUserId(userId)
}
```

공통 기능은 `BaseRepository` 같은 인터페이스를 통해 재사용하고, 각 Repository 구현체는 모두 `infrastructure`에 둔다.

**이렇게 설계한 이유:**
- 구조가 단순하고 이해하기 쉽다
- Repository는 사실상 기술적 구현이므로 `infrastructure`에 두는 것이 자연스럽다
- 필요할 때만 별도 인터페이스를 도입해도 충분하다

## 애그리게잇 분리 전략: 트래픽 패턴을 고려한 설계

DDD에서 애그리게잇은 한 트랜잭션에서 일관성을 보장하는 단위다. 하지만 항상 하나의 애그리게잇으로 묶어야 하는 것은 아니다.

예를 들어 `Project`와 `Task`를 별도의 애그리게잇으로 분리할 수 있다. 이유는 다음과 같다:

### 1. 트래픽 패턴의 차이

- **Task 변경**: 매우 높은 트래픽, 빈번한 변경
- **Project 메타 변경**: 낮은 트래픽, 드문 변경

이 두 가지를 같은 애그리게잇으로 묶으면, Task 상태 변경이 많을 때 Project 조회까지 락이 걸릴 수 있다.

### 2. 트랜잭션 경계 분리

Project 이름 변경과 Task 생성은 실제로 같은 트랜잭션으로 묶을 필요가 없다. 각각 독립적으로 트랜잭션을 관리하는 것이 더 합리적이다.

### 3. 성능 최적화

- 락 경합 최소화
- 조회/저장 패턴을 독립적으로 최적화 가능
- 스케줄 관련 쿼리만 최적화해도 충분한 경우가 많음

### 약한 결합으로 연결

분리된 애그리게잇은 ID로 약하게 결합된다:

```kotlin
class Task(
    projectId: Long,  // Project.id 참조 (약한 결합)
    assigneeId: Long, // User.id 참조 (약한 결합)
    // ...
)
```

필요한 경우 서비스 레이어에서 Repository를 통해 조회한다:

```kotlin
val project = projectRepository.findByIdOrNull(task.projectId)
```

## 도메인 간 결합: 약한 결합의 실천

다른 바운더리 컨텍스트의 엔티티를 참조할 때는 항상 ID만 사용한다.

### 같은 애그리게잇 내부: 강한 결합

```kotlin
class OrderItem(
    order: Order,      // 같은 애그리게잇 (강한 결합)
    productId: Long,   // 다른 컨텍스트 (약한 결합)
)
```

### 다른 바운더리 컨텍스트: 약한 결합

```kotlin
class Order(
    userId: Long,      // User 도메인 참조 (약한 결합)
    paymentId: Long,   // Payment 애그리게잇 참조 (약한 결합)
)
```

**약한 결합의 장점:**
- 도메인 간 의존성 최소화
- 각 도메인의 독립적인 진화 가능
- 순환 참조 방지
- 테스트 용이성 향상

## 도메인 간 통신: Client 패턴

다른 도메인의 정보가 필요할 때는 Client를 통해 통신한다. 이는 개발 편의성과 관리 측면에서 선호하는 방식이다.

### 구조

Client는 해당 도메인의 `application` 레이어에 정의하고, 다른 도메인에서 사용한다:

```kotlin
// user/application/UserClient.kt
@Component
class UserClient(
    private val userRepository: UserRepository,
) {
    fun getUserById(userId: Long): UserDto {
        val user = userRepository.findByIdOrNull(userId)
            ?: throw AnyException(ErrorCode.USER_NOT_FOUND, mapOf("userId" to userId))
        
        return UserDto(
            id = user.id,
            name = user.name,
            email = user.email,
        )
    }
    
    fun existsUser(userId: Long): Boolean {
        return userRepository.existsById(userId)
    }
}

// user/application/dto/UserDto.kt
data class UserDto(
    val id: Long,
    val name: String,
    val email: String?,
)
```

다른 도메인에서 사용할 때:

```kotlin
// order/application/OrderService.kt
@Service
class OrderService(
    private val orderRepository: OrderRepository,
    private val userClient: UserClient,  // 다른 도메인의 Client 주입
) {
    fun createOrder(userId: Long, items: List<OrderItem>) {
        // Entity가 아닌 DTO로 반환, 없으면 예외 발생
        val user = userClient.getUserById(userId)
        
        // ...
    }
}
```

### 설계 원칙

1. **응답은 항상 DTO**: Entity를 직접 반환하지 않는다. 이는 더티 체킹 방지와 의존성 격리에 중요하다.

2. **Client는 application 레이어에 정의**: 해당 도메인의 application 레이어에서 인터페이스와 구현을 관리한다.

3. **다른 도메인에서 Client 사용**: 필요할 때 Client를 주입받아 사용한다.

4. **개발 편의성 우선**: 이론적으로 완벽한 구조보다는, 실제 개발과 관리가 편한 방식을 선호한다.

**이렇게 설계한 이유:**
- 도메인 간 통신 경로가 명확하다
- Entity 노출을 방지하여 더티 체킹 문제를 원천 차단한다
- 각 도메인의 Client를 한 곳에서 관리할 수 있어 유지보수가 쉽다
- 여러 도메인에서 재사용 가능하다

## 마무리: 균형 있는 설계

이 구조는 처음부터 완벽하게 설계된 것이 아니다. 개발하면서 부딪힌 문제들을 해결해 나가며 점진적으로 개선했다.

**중요한 것은:**
- 이론에 매몰되지 않기
- 프로젝트의 특성에 맞게 유연하게 적용하기
- 과도한 추상화는 피하기
- 실용성과 확장성의 균형 잡기

헥사고날 아키텍처는 좋은 원칙을 제시하지만, 모든 프로젝트에 똑같이 적용할 수는 없다. 프로젝트의 규모, 팀의 역량, 비즈니스 요구사항을 고려하여 적절한 수준에서 적용하는 것이 중요하다.

이 구조가 완벽하다고 생각하지 않는다. 앞으로도 계속 개선해 나갈 것이다. 하지만 지금까지 이 구조로 개발하면서 느낀 점은, **명확한 경계와 원칙이 있으면 코드를 이해하고 유지보수하는 것이 훨씬 수월하다**는 것이다.

---
