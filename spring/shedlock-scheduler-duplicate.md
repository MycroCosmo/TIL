# ShedLock으로 다중 서버 스케줄러 중복 실행 막기

> 작성일: 2026-11-19

## 한 줄 요약

`@Scheduled`는 서버마다 따로 돌기 때문에 서버가 여러 대면 같은 배치가 동시에 실행된다. ShedLock은 **DB에 "지금 실행 중" 표시를 남겨서 한 서버만 실행**하게 한다.

## 내용

### 문제

- 서버 3대에 같은 애플리케이션을 배포하면 스케줄러도 3개가 돈다.
- 정산 배치처럼 한 번만 돌아야 하는 작업이 같은 시각에 3번 실행된다.

### 검토했던 방법

| 방법 | 장점 | 단점 |
| --- | --- | --- |
| 스케줄링 전용 서버 분리 | 구조가 명확함 | 서버·배포 구성을 바꿔야 함 |
| 테이블 상태 필드로 작업 선점 | 추가 라이브러리 없음 | 선점·해제·장애 처리를 직접 구현 |
| **ShedLock** | 이미 쓰는 DB로 해결, 적용이 간단 | 락 시간 설정을 잘못하면 중복 가능 |

### 사용 방법

```java
@EnableScheduling
@EnableSchedulerLock(defaultLockAtMostFor = "10m")
@Configuration
public class SchedulerConfig {

    @Bean
    public LockProvider lockProvider(DataSource dataSource) {
        return new JdbcTemplateLockProvider(dataSource);
    }
}
```

```java
@Scheduled(cron = "0 0 0 * * *")
@SchedulerLock(name = "settlementJob", lockAtMostFor = "30m", lockAtLeastFor = "1m")
public void settle() { ... }
```

- `shedlock` 테이블(`name`, `lock_until`, `locked_at`, `locked_by`)에 락을 기록한다.
- 먼저 락을 잡은 서버만 실행하고, 나머지는 **기다리지 않고 건너뛴다.**

### 설정값의 의미

- `lockAtMostFor`: 락을 최대 유지하는 시간. 실행 중인 서버가 죽어도 이 시간이 지나면 락이 풀린다.
  - 작업이 이 시간보다 오래 걸리면 **다른 서버가 또 실행할 수 있다.** → 작업 시간보다 넉넉하게 잡는다.
- `lockAtLeastFor`: 작업이 아주 빨리 끝나도 최소 이 시간은 락을 유지한다. 서버 간 시계 차이로 다른 서버가 직후에 다시 실행하는 것을 막는다.

### 검증

- 포트를 다르게 해서 로컬 인스턴스를 여러 개 띄우고 동시에 실행시켜, 한 곳에서만 실행되는지 확인했다.

## 헷갈렸던 점 / 주의할 점

- ShedLock은 "동시에 두 번"을 막는 것이지 **"정확히 한 번"을 보장하지 않는다.** 배치 자체를 여러 번 실행해도 안전하게(건 단위 완료 상태 저장 등) 만드는 게 더 근본적이다.
- 서버가 한 대라면 ShedLock의 역할이 크지 않다. 면접에서 쓴 이유를 물으면 실제 환경 그대로 말한다.

## 참고

- https://github.com/lukas-krecan/ShedLock
