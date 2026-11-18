# 싱글톤 빈에 상태(필드)를 두면 안 되는 이유

> 작성일: 2026-11-19

## 한 줄 요약

Spring 빈은 기본이 싱글톤이라 **모든 요청 스레드가 같은 인스턴스를 공유**한다. 변경 가능한 필드를 두면 동시성 문제와 메모리 누수가 생긴다.

## 내용

```java
@Service
public class InboundService {

    private List<String> processedInvoices = new ArrayList<>(); // 위험

    public void receive(List<InboundItem> items) {
        for (InboundItem item : items) {
            if (processedInvoices.contains(item.getInvoiceNo())) continue;
            inboundMapper.insert(item);
            processedInvoices.add(item.getInvoiceNo());
        }
    }
}
```

문제점

1. **스레드 안전하지 않음**: 여러 요청이 동시에 `ArrayList`를 읽고 쓴다. `contains` 확인과 `add` 사이에 다른 스레드가 끼어들 수 있다(Check-then-Act).
2. **서버마다 따로 존재**: 서버가 여러 대면 각 서버의 리스트가 다르다. 중복 판단이 서버 하나 안에서만 된다.
3. **재시작하면 사라짐**: 메모리에만 있어서 배포 후에는 중복 방지가 초기화된다.
4. **계속 쌓임**: 지우는 코드가 없으면 메모리가 계속 늘어난다.

### 해결

- 중복 방지는 **DB 유니크 키**로 한다. (예: 송장 번호 UNIQUE → 중복 INSERT는 예외로 걸러짐)
- 빈에는 상태를 두지 말고, 필요한 값은 메서드 지역 변수나 파라미터로 넘긴다.
- 정말 공유 상태가 필요하면 `ConcurrentHashMap` 같은 스레드 안전한 자료구조를 쓰되, 다중 서버라면 Redis·DB 같은 외부 저장소를 쓴다.

## 헷갈렸던 점 / 주의할 점

- 같은 이유로 `SimpleDateFormat`을 필드로 두는 것도 위험하다. → [SimpleDateFormat 스레드 안전성](../java/simpledateformat-thread-safety.md)
- `final`로 선언된 의존성(Mapper, Repository)은 상태가 아니라 괜찮다. 문제는 **요청마다 바뀌는 값**을 필드에 두는 것이다.

## 참고

- Spring Framework Reference - Bean Scopes (Singleton)
