# 트랜잭션 격리 수준

> 작성일: 2026-11-19

## 한 줄 요약

격리 수준은 동시에 실행되는 트랜잭션끼리 **서로의 변경을 얼마나 볼 수 있게 할지**를 정한다. 높을수록 안전하지만 동시성은 떨어진다.

## 내용

| 격리 수준 | Dirty Read | Non-Repeatable Read | Phantom Read |
| --- | --- | --- | --- |
| READ UNCOMMITTED | 발생 | 발생 | 발생 |
| READ COMMITTED | 막음 | 발생 | 발생 |
| REPEATABLE READ | 막음 | 막음 | 발생 (표준 기준) |
| SERIALIZABLE | 막음 | 막음 | 막음 |

- **Dirty Read**: 다른 트랜잭션이 **커밋하지 않은** 값을 읽음. 그 트랜잭션이 롤백되면 존재하지 않는 값을 본 셈이다.
- **Non-Repeatable Read**: 같은 행을 두 번 읽었는데, 사이에 다른 트랜잭션이 수정·커밋해서 **값이 다름**.
- **Phantom Read**: 같은 조건으로 두 번 조회했는데, 사이에 다른 트랜잭션이 INSERT해서 **행 개수가 다름**.

### DB별 기본값

| DB | 기본 격리 수준 |
| --- | --- |
| MySQL (InnoDB) | REPEATABLE READ |
| PostgreSQL | READ COMMITTED |
| SQL Server | READ COMMITTED |
| Oracle | READ COMMITTED |

- MySQL InnoDB는 MVCC와 갭 락으로 REPEATABLE READ에서도 팬텀 리드를 대부분 막는다.

### ACID에서의 위치

- 격리 수준은 ACID의 **I(Isolation)**를 얼마나 엄격하게 지킬지 조절하는 설정이다.
- 격리 수준을 올려도 "조회 후 수정" 패턴의 Lost Update가 자동으로 해결되지는 않는다. 락이나 조건부 UPDATE가 필요하다.

## 헷갈렸던 점 / 주의할 점

- REPEATABLE READ와 READ COMMITTED를 반대로 말한 적이 있다. 이름 그대로 기억한다.
  - READ **COMMITTED**: 커밋된 것만 읽는다 → Dirty Read만 막는다.
  - **REPEATABLE** READ: 같은 걸 다시 읽어도(repeat) 같다 → Non-Repeatable Read까지 막는다.

## 참고

- [Lost Update와 락 선택](./lost-update-and-locks.md)
