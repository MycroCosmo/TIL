# 동시 주문 재고 차감: Lost Update와 락 선택

> 작성일: 2026-11-19

## 한 줄 요약

"조회 → 계산 → 저장" 사이에 다른 요청이 끼어들면 한쪽 변경이 사라진다(Lost Update). **비관적 락, 낙관적 락, 조건부 UPDATE** 중 상황에 맞게 고른다.

## 내용

### 문제

```java
Stock stock = stockRepository.findByProductId(id); // 두 요청 모두 재고 10을 읽음
stock.decrease(1);                                 // 각자 9로 계산
stockRepository.save(stock);                       // 둘 다 9로 저장 → 1개가 사라짐
```

- 확인하고 행동하는 사이에 상태가 바뀌는 Check-then-Act 문제다.
- 트랜잭션을 걸어도 기본 격리 수준(READ COMMITTED)에서는 막히지 않는다.

### 방법 1. 비관적 락

```sql
SELECT qty FROM stock WHERE product_id = ? FOR UPDATE;                -- MySQL, PostgreSQL
SELECT qty FROM stock WITH (UPDLOCK, ROWLOCK) WHERE product_id = ?;   -- SQL Server
```

- 읽을 때부터 행을 잠가서 다른 트랜잭션은 커밋될 때까지 기다린다.
- 확실하지만 대기가 생기고, 여러 행을 잠그면 **데드락** 위험이 있다.
- 충돌이 잦은 곳(인기 상품 재고)에 맞다.

### 방법 2. 낙관적 락

```java
@Version
private Long version;
```

- 저장할 때 `WHERE version = ?` 조건으로 바뀐 적이 없는지 확인한다. 바뀌었으면 예외가 나고 **재시도**해야 한다.
- 락을 잡지 않아서 빠르지만, 충돌이 잦으면 재시도가 폭증한다.
- 충돌이 드문 곳(회원 정보 수정)에 맞다.

### 방법 3. 조건부 UPDATE (원자적 UPDATE)

```sql
UPDATE stock
SET qty = qty - #{amount}
WHERE product_id = #{productId}
  AND qty >= #{amount}
```

- 계산과 저장을 **쿼리 하나**로 처리해서 끼어들 틈이 없다.
- 영향받은 행 수가 0이면 재고 부족(또는 이미 처리됨)으로 판단한다.
- 단순 증감이라면 가장 가볍고 확실하다.

## 헷갈렸던 점 / 주의할 점

- 애플리케이션의 `synchronized`는 **서버 한 대 안에서만** 동작한다. 서버가 여러 대면 소용없다.
- 조건부 UPDATE는 중복 요청 방지(멱등성)에도 같은 원리로 쓸 수 있다. → [조건부 UPDATE로 중복 통보 막기](./conditional-update-idempotency.md)
- 락을 잡는 순서가 트랜잭션마다 다르면 데드락이 난다. → [데드락](./deadlock.md)

## 참고

- JPA `@Version`, `LockModeType.PESSIMISTIC_WRITE`
