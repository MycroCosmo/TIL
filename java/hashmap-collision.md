# HashMap 해시 충돌과 equals/hashCode

> 작성일: 2026-11-19

## 한 줄 요약

HashMap은 `hashCode()`로 버킷을 찾고 `equals()`로 같은 키인지 확인한다. 충돌하면 같은 버킷에 연결 리스트로 쌓고, 많아지면 트리로 바꾼다. **equals를 재정의했으면 hashCode도 반드시 재정의**해야 한다.

## 내용

### 동작 순서

1. `key.hashCode()`를 보정한 값으로 버킷 인덱스를 구한다.
2. 그 버킷에 이미 다른 키가 있으면(해시 충돌) **같은 버킷에 이어서 저장**한다(Separate Chaining).
3. 조회할 때는 버킷을 찾은 뒤 `equals()`로 실제 같은 키를 찾는다.

### Java 8 이후 트리화

- 한 버킷에 노드가 **8개 이상** 쌓이면 연결 리스트를 **레드-블랙 트리**로 바꾼다. 최악의 조회가 O(n) → O(log n)이 된다.
- 단, 전체 테이블 크기가 64 미만이면 트리로 바꾸지 않고 테이블을 키운다.
- 노드가 6개 이하로 줄면 다시 리스트로 돌아간다.

### 리사이즈

- 저장된 개수가 `용량 × 0.75(load factor)`를 넘으면 용량을 2배로 늘리고 다시 배치한다.
- 넣을 개수를 미리 알면 초기 용량을 지정해 리사이즈 비용을 줄일 수 있다.

### equals만 재정의하면 생기는 문제

```java
public class InboundItem {
    private String invoiceNo;

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof InboundItem)) return false;
        return invoiceNo.equals(((InboundItem) o).invoiceNo);
    }
    // hashCode() 없음
}

Set<InboundItem> unique = new HashSet<>(items); // 중복 제거가 안 된다
```

- `hashCode()`가 `Object` 기본 구현이라 송장 번호가 같아도 해시가 달라서 **다른 버킷**에 들어간다.
- `equals()`를 비교할 기회조차 없어서 중복이 그대로 남는다.
- 규칙: `equals()`가 true면 `hashCode()`도 같아야 한다.

## 헷갈렸던 점 / 주의할 점

- 해시 충돌은 "같은 hashCode"뿐 아니라 "다른 hashCode인데 같은 버킷 인덱스"도 포함한다.
- 이 버그는 코드 리뷰에서 `HashSet`으로 중복 제거하는 코드를 보면 바로 의심해야 한다.

## 참고

- `java.util.HashMap` 소스 (`TREEIFY_THRESHOLD`, `UNTREEIFY_THRESHOLD`, `MIN_TREEIFY_CAPACITY`)
- Effective Java 아이템 11: equals를 재정의하려거든 hashCode도 재정의하라
