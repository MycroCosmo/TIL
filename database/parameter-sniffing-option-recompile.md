# SQL Server 파라미터 스니핑과 OPTION (RECOMPILE)

> 작성일: 2026-11-19

## 한 줄 요약

SQL Server는 처음 실행할 때의 파라미터 값으로 실행계획을 만들어 캐시하고 재사용한다. 그 계획이 **다른 파라미터에는 비효율적**일 수 있는데, 이것이 파라미터 스니핑 문제다. `OPTION (RECOMPILE)`은 **실행할 때마다 실제 값으로 계획을 새로 만든다.**

## 내용

### 파라미터 스니핑

- 파라미터화된 쿼리(저장 프로시저, `sp_executesql`, MyBatis `#{}`)는 SQL 문자열이 같아서 실행계획이 캐시에 하나만 남는다.
- 처음 들어온 값이 "1일치 조회"였다면 소량 데이터에 맞는 계획(예: Nested Loop + Index Seek)이 캐시된다.
- 다음에 "3개월치 조회"가 들어와도 같은 계획을 재사용해서 매우 느려진다.
- 같은 쿼리가 **개발에서는 빠른데 운영에서만 느린** 경우, 데이터 분포와 캐시된 계획이 다른지 의심해 볼 만하다.

### OPTION (RECOMPILE)

```sql
SELECT ...
FROM inspection i
WHERE i.check_date BETWEEN @from AND @to
OPTION (RECOMPILE);
```

- 캐시된 계획을 쓰지 않고, 매번 **실제 파라미터 값 기준**으로 계획을 만든다.
- 실행계획을 "고정"하는 힌트가 아니라 **매번 새로 만드는** 힌트다.

### 아무 데나 붙이면 안 되는 이유

- 실행할 때마다 컴파일하는 **CPU 비용**이 든다.
- 자주 호출되는 가벼운 쿼리에는 손해다.
- 호출 빈도가 낮고 조회가 무거워서, 컴파일 비용보다 잘못된 계획의 비용이 훨씬 큰 쿼리에 쓴다.

### 다른 선택지

- `OPTION (OPTIMIZE FOR (@param = 값))`: 특정 값 기준으로 계획을 만든다.
- `OPTION (OPTIMIZE FOR UNKNOWN)`: 특정 값 대신 통계의 평균 분포로 계획을 만든다.
- 쿼리 자체를 고쳐서(상관 서브쿼리 제거, 미리 집계한 결과와 조인 등) 어떤 값이 와도 무난한 계획이 나오게 하는 게 가장 근본적이다.

## 헷갈렸던 점 / 주의할 점

- "옵티마이저가 한 번도 실행된 적이 없어서 최적화를 못 했다"는 틀린 설명이다. 옵티마이저는 첫 실행 때 계획을 만들고, 문제는 **그 계획이 이후 값에 안 맞는 것**이다.
- 실행계획은 SQL Server에서 "예상 실행계획 / 실제 실행계획"으로 본다. MySQL의 `EXPLAIN`과 용어를 섞어 말하지 않는다.
- 실제 적용 사례: [저장 프로시저 MyBatis 이관 후 타임아웃](../troubleshooting/stored-procedure-migration-timeout.md)

## 참고

- Microsoft Learn - Query Hints (Transact-SQL)
- Microsoft Learn - Parameter Sensitivity / Parameter Sniffing
