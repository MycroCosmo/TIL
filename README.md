# TIL (Today I Learned)

매일 배운 것을 짧게 기록하는 저장소입니다.
길게 정리하는 것보다 **매일 한 가지라도 남기는 것**을 목표로 합니다.

## 작성 규칙

1. **한 파일에 한 개념만** 정리합니다. 길어지면 파일을 나눕니다.
2. 파일은 주제 폴더 아래에 `kebab-case` 이름으로 만듭니다.
   - 예: `spring/transactional-propagation.md`
3. [TEMPLATE.md](./TEMPLATE.md) 양식을 복사해서 씁니다.
4. 회사 코드나 내부 정보는 올리지 않고, **개념과 해결 방법만** 일반화해서 정리합니다.
5. 잘 정리된 글은 [블로그](https://0and24.tistory.com/)에 옮겨 다듬습니다.

## 커밋 메시지

```
docs(<폴더>): <정리한 내용>
```

- `docs(spring): @Transactional 전파 속성 정리`
- `docs(troubleshooting): Thymeleaf fragment 파라미터 누락 오류 해결`
- 기존 글을 고칠 때: `fix(jpa): N+1 해결 방법 예시 코드 수정`

## 카테고리

| 폴더 | 내용 |
| --- | --- |
| [java](./java) | Java 문법, 컬렉션, 동시성, JVM |
| [spring](./spring) | Spring / Spring Boot, 트랜잭션, 테스트 |
| [jpa](./jpa) | JPA, Hibernate, MyBatis 등 데이터 접근 |
| [database](./database) | SQL, 인덱스, 실행 계획, 트랜잭션 격리 수준 |
| [cs](./cs) | 네트워크, 운영체제, 자료구조, 컴퓨터 구조 |
| [frontend](./frontend) | Vue, React, Nuxt, Thymeleaf |
| [troubleshooting](./troubleshooting) | 실무에서 겪은 문제와 해결 과정 |
| [etc](./etc) | 도구, 개발 환경, 기타 |

## 글 목록

### java
- [HashMap 해시 충돌과 equals/hashCode](./java/hashmap-collision.md)
- [SimpleDateFormat은 스레드 안전하지 않다](./java/simpledateformat-thread-safety.md)
- [유틸리티 클래스 제대로 만들기 (JsonUtils 복기)](./java/utility-class-design.md)

### spring
- [ShedLock으로 다중 서버 스케줄러 중복 실행 막기](./spring/shedlock-scheduler-duplicate.md)
- [싱글톤 빈에 상태(필드)를 두면 안 되는 이유](./spring/singleton-bean-state.md)
- [트랜잭션 전파: REQUIRED vs REQUIRES_NEW, 그리고 @Async](./spring/transaction-propagation.md)
- [@Transactional 롤백 규칙과 readOnly](./spring/transactional-rollback-rules.md)
- [@Transactional 내부 호출과 private 메서드](./spring/transactional-self-invocation.md)

### jpa
- [MyBatis #{} vs ${}](./jpa/mybatis-dollar-vs-hash.md)
- [JPA N+1 문제](./jpa/n-plus-one.md)

### database
- [조건부 UPDATE로 결제 중복 통보 한 번만 반영하기](./database/conditional-update-idempotency.md)
- [데드락](./database/deadlock.md)
- [트랜잭션 격리 수준](./database/isolation-levels.md)
- [동시 주문 재고 차감: Lost Update와 락 선택](./database/lost-update-and-locks.md)
- [SQL Server 파라미터 스니핑과 OPTION (RECOMPILE)](./database/parameter-sniffing-option-recompile.md)

### frontend
- [Thymeleaf 3.1에서 #request, #session이 사라졌다](./frontend/thymeleaf-3-1-removed-utility-objects.md)

### troubleshooting
- [Redis 캐시에서 SliceImpl 역직렬화 실패](./troubleshooting/redis-sliceimpl-deserialization.md)
- [10년 범위 통계 조회 5,000ms → 300ms 개선](./troubleshooting/statistics-query-performance.md)
- [저장 프로시저를 MyBatis로 이관한 뒤 생긴 타임아웃](./troubleshooting/stored-procedure-migration-timeout.md)

### etc
- [첫 오픈소스 PR: Spring Boot 문서 수정과 DCO](./etc/first-open-source-pr.md)
- [GitHub 잔디(기여 그래프)가 찍히는 조건](./etc/github-contribution-graph.md)
