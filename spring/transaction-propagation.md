# 트랜잭션 전파: REQUIRED vs REQUIRES_NEW, 그리고 @Async

> 작성일: 2026-11-19

## 한 줄 요약

REQUIRED는 바깥 트랜잭션에 **참여**하고, REQUIRES_NEW는 바깥을 잠시 멈추고 **독립된 새 트랜잭션**을 만든다. `@Async`는 다른 스레드라 바깥 트랜잭션이 **아예 전달되지 않는다.**

## 내용

| | REQUIRED (기본값) | REQUIRES_NEW |
| --- | --- | --- |
| 바깥 트랜잭션이 있을 때 | 참여 (하나로 묶임) | 바깥을 멈추고 새로 시작 |
| 바깥 트랜잭션이 없을 때 | 새로 시작 | 새로 시작 |
| 안쪽이 실패하면 | 전체 롤백 | 안쪽만 롤백 |
| 바깥이 실패하면 | 전체 롤백 | 안쪽은 이미 커밋된 상태로 남음 |

### 안쪽 예외를 바깥에서 catch해도 롤백되는 이유

```java
@Transactional
public void order() {
    orderRepository.save(order);
    try {
        pointService.addPoint(userId); // REQUIRED, 내부에서 RuntimeException
    } catch (Exception e) {
        // 포인트 실패는 무시하고 주문은 진행하려는 의도
    }
}
```

1. `addPoint()`는 REQUIRED라 바깥 트랜잭션에 참여한다.
2. 안쪽 프록시가 예외를 보고 공유 트랜잭션에 **rollback-only** 표시를 한다.
3. 바깥은 정상 종료해서 커밋을 시도하지만, 이미 표시가 있어 **전체 롤백**된다.
4. 그리고 `UnexpectedRollbackException`이 발생한다.

해결: 포인트를 `REQUIRES_NEW`로 분리하거나, 주문 커밋 후 이벤트(`@TransactionalEventListener(phase = AFTER_COMMIT)`)로 처리한다.

### REQUIRES_NEW를 남발하면

- **정합성**: 바깥이 롤백돼도 안쪽은 커밋돼 있어서, 같이 취소돼야 할 데이터가 남는다. 로그·이력처럼 남아도 되는 것만 분리한다.
- **커넥션 풀 고갈**: 바깥 트랜잭션은 끝난 게 아니라 멈췄을 뿐이라 커넥션을 계속 쥐고 있다. 요청 하나가 커넥션 2개를 쓰고, 동시 요청이 몰리면 서로 커넥션을 기다리다 풀이 바닥난다.

### @Async와 트랜잭션

- 트랜잭션은 ThreadLocal로 **스레드에 묶여** 있어서 다른 스레드로 넘어가지 않는다.
- `@Async` 메서드에 `@Transactional`이 없으면 트랜잭션이 없고, 있으면 그 스레드에서 별도로 시작된다.
- 바깥이 커밋되기 전에 비동기 작업이 먼저 돌면 방금 저장한 데이터를 **못 볼 수 있다**. → 커밋 이후에 실행되도록 한다.
- 비동기 쪽 예외는 호출한 쪽으로 올라오지 않으므로 로그·재시도를 따로 설계해야 한다.

## 헷갈렸던 점 / 주의할 점

- REQUIRED를 "외부 트랜잭션을 허용한다"고 말했는데, **"참여한다 / 독립한다"**가 정확한 표현이다.
- `@Async`는 REQUIRES_NEW와 같은 게 아니다. 트랜잭션 전파 자체가 없는 것이다.

## 참고

- Spring Framework Reference - Transaction Propagation
