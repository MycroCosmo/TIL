# @Transactional 롤백 규칙과 readOnly

> 작성일: 2026-11-19

## 한 줄 요약

기본적으로 **RuntimeException과 Error만 롤백**되고, checked 예외나 catch로 삼킨 예외는 **커밋된다.**

## 내용

### 1. checked 예외는 롤백되지 않는다

```java
@Transactional
public void cancel(Long id) throws IOException {
    orderRepository.updateStatus(id, CANCELED);
    externalClient.notify(id); // IOException 발생 → 그래도 커밋됨
}
```

- 기본 규칙은 unchecked(`RuntimeException`, `Error`)만 롤백이다.
- checked 예외도 롤백하려면 `@Transactional(rollbackFor = Exception.class)`처럼 지정한다.

### 2. 예외를 catch해서 삼키면 롤백되지 않는다

```java
try {
    mapper.insert(item);
} catch (Exception e) {
    log.error("실패", e); // 예외가 밖으로 안 나감 → 정상 종료로 보고 커밋
}
```

- 프록시는 메서드 밖으로 던져진 예외를 보고 롤백을 결정한다.
- 트랜잭션 안에서 "안전하게 하려고" try-catch를 넣으면 **오히려 롤백이 안 되는** 위험한 코드가 된다.
- 꼭 잡아야 하면 다시 던지거나 `TransactionAspectSupport.currentTransactionStatus().setRollbackOnly()`로 직접 표시한다.

### 3. readOnly = true에서 쓰기를 하면

- **JPA(Hibernate)**: flush 모드가 MANUAL이 되어 변경 감지로 인한 UPDATE가 DB에 반영되지 않는다.
- **MyBatis·JDBC**: 영속성 컨텍스트가 없다. 커넥션에 읽기 전용 힌트만 전달되고, 쓰기가 막히는지는 DB와 드라이버에 따라 다르다.
- 즉 "readOnly니까 쓰기를 막아준다"고 기대하면 안 된다. 쓰기가 있는 메서드에는 붙이지 않는다.

## 헷갈렸던 점 / 주의할 점

- MyBatis 코드를 리뷰하면서 "readOnly라 영속성 컨텍스트가 동작하지 않는다"고 답했는데, MyBatis에는 영속성 컨텍스트가 없다. **어떤 기술을 쓰는 코드인지 먼저 보고** 설명해야 한다.
- 트랜잭션 안에서 외부 API를 호출하면 응답을 기다리는 동안 커넥션을 계속 잡고 있고, DB가 롤백돼도 외부 요청은 되돌릴 수 없다. 외부 호출은 커밋 이후(`@TransactionalEventListener(phase = AFTER_COMMIT)`)로 뺀다.

## 참고

- Spring Framework Reference - Rolling Back a Declarative Transaction
