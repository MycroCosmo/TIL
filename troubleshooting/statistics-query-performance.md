# 10년 범위 통계 조회 5,000ms → 300ms 개선

> 작성일: 2026-11-19

## 한 줄 요약

장기간 통계 화면이 느렸던 원인은 쿼리와 애플리케이션 양쪽에 있었다. **쿼리를 UNION ALL로 재작성하고, Java의 이중 반복을 HashMap 조회로 바꾸고, 인덱스를 정리**해서 5초 → 0.3초로 줄였다.

## 상황

- 공공기관 웹 서비스의 통계 화면. 최대 10년 범위를 조회할 수 있었다.
- 기간을 길게 잡으면 응답이 약 5초 걸렸다.

## 원인과 해결

### 1. 쿼리 재작성 (UNION ALL)

- 집계 쿼리를 `UNION ALL`로 합치는 구조로 재작성했다.
- `UNION`이 아니라 `UNION ALL`을 쓴 이유: `UNION`은 중복 제거를 위해 정렬·비교 비용이 추가된다. 각 결과가 겹치지 않으면 `UNION ALL`이 맞다.

### 2. 이중 반복 → HashMap

```java
// Before: 기간 목록 × 집계 결과를 이중 for문으로 매칭 → O(n × m)
for (Period p : periods) {
    for (Stat s : stats) {
        if (s.getKey().equals(p.getKey())) { ... }
    }
}

// After: 집계 결과를 Map으로 한 번 만들고 조회 → O(n + m)
Map<String, Stat> statMap = stats.stream()
        .collect(Collectors.toMap(Stat::getKey, Function.identity()));
for (Period p : periods) {
    Stat s = statMap.get(p.getKey());
}
```

- 10년 범위면 기간과 결과 행이 모두 많아서 이중 반복의 비용이 컸다.

### 3. 인덱스

- 조회 조건에 맞게 인덱스를 추가했다.

## 배운 점

- 느린 원인이 DB에만 있다고 단정하지 않는다. **DB 실행 시간과 애플리케이션 처리 시간을 나눠서 측정**해야 어디를 고칠지 보인다.
- 성능 수치를 말할 때는 **무엇을 측정했는지**(API 응답 시간인지, 쿼리 실행 시간인지)를 같이 말한다.
- 시간복잡도 개념이 실무에서 그대로 쓰인 사례다. O(n × m) → O(n + m).

## 참고

- [HashMap 해시 충돌](../java/hashmap-collision.md)
