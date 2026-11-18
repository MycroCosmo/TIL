# JPA N+1 문제

> 작성일: 2026-11-19

## 한 줄 요약

목록을 1번 조회한 뒤, 각 엔티티의 연관 데이터를 가져오느라 **N번의 추가 쿼리**가 나가는 문제다. LAZY 로딩은 이걸 해결하는 게 아니라 **나중으로 미룰 뿐**이다.

## 내용

```java
List<Order> orders = orderRepository.findAll();       // 쿼리 1번
for (Order order : orders) {
    order.getMember().getName();                      // 주문마다 1번씩 → N번
}
```

### LAZY로 바꾸면 해결될까?

- EAGER: 조회 시점에 바로 N번 추가 쿼리가 나간다.
- LAZY: 프록시로 두었다가 **실제로 접근하는 순간** 쿼리가 나간다. 루프에서 접근하면 결국 N번이다.
- 즉 LAZY는 "필요 없을 때 안 가져오는" 장점이 있을 뿐, N+1 자체를 없애지는 않는다. 기본은 LAZY로 두고 필요한 곳에서 한 번에 가져오는 게 원칙이다.

### 해결 방법

| 방법 | 설명 | 주의 |
| --- | --- | --- |
| fetch join | JPQL `join fetch`로 연관 엔티티를 한 쿼리로 가져옴 | 컬렉션 fetch join + 페이징 시 메모리에서 페이징함 |
| `@EntityGraph` | 어노테이션으로 함께 가져올 연관 관계 지정 | 내부적으로는 left outer join |
| batch size | `default_batch_fetch_size`로 지연 로딩을 `IN` 절로 묶음 | N번 → N/size번으로 줄어듦 |

```java
@Query("select o from Order o join fetch o.member")
List<Order> findAllWithMember();
```

```yaml
spring:
  jpa:
    properties:
      hibernate:
        default_batch_fetch_size: 100
```

### 컬렉션 fetch join과 페이징

- 일대다 컬렉션을 fetch join하면 행이 늘어나서 DB에서 페이징할 수 없다.
- Hibernate는 경고를 남기고 **전부 메모리로 가져와서 페이징**한다. 데이터가 많으면 메모리 장애로 이어진다.
- 이 경우 ToOne 관계만 fetch join하고, 컬렉션은 batch size로 푼다.

## 헷갈렸던 점 / 주의할 점

- 조회 대상이 여러 개면 연관마다 N이 생긴다. 주문 항목마다 상품·재고를 조회하면 쿼리가 2N+1개가 된다.
- 반복문 안에서 Repository·Mapper를 호출하는 코드는 JPA가 아니어도 같은 문제다. `IN` 절로 한 번에 조회해서 Map으로 매칭한다.
- 느리다고 캐시부터 붙이기 전에, **쿼리 로그로 몇 번 나가는지 먼저 측정**한다.

## 참고

- Hibernate ORM User Guide - Fetching
