# SimpleDateFormat은 스레드 안전하지 않다

> 작성일: 2026-11-19

## 한 줄 요약

`SimpleDateFormat`은 내부에 파싱 중간 상태를 가지고 있어서, **여러 스레드가 하나를 공유하면 날짜가 엉뚱하게 바뀌거나 예외가 난다.** `DateTimeFormatter`를 쓰면 된다.

## 내용

```java
@Service
public class DeliveryService {
    private final SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd"); // 위험

    public Date parse(String date) throws ParseException {
        return format.parse(date);
    }
}
```

- 싱글톤 빈의 필드라 모든 요청 스레드가 같은 인스턴스를 쓴다.
- `SimpleDateFormat`은 내부 `Calendar` 객체에 값을 썼다가 읽는 방식이라, 동시에 호출되면 서로의 값을 덮어쓴다.
- 결과: 다른 요청의 날짜가 섞이거나 `NumberFormatException` 같은 이상한 예외가 간헐적으로 난다. 테스트에서는 잘 안 보이고 운영에서만 터지는 종류의 버그다.

### 해결

```java
private static final DateTimeFormatter FORMAT = DateTimeFormatter.ofPattern("yyyy-MM-dd");

public LocalDate parse(String date) {
    return LocalDate.parse(date, FORMAT);
}
```

- `java.time`의 `DateTimeFormatter`는 **불변(immutable)이고 스레드 안전**해서 `static final`로 공유해도 된다.
- 레거시라 `Date`를 꼭 써야 하면 메서드 안에서 매번 새로 만들거나 `ThreadLocal`로 스레드마다 하나씩 둔다.

## 헷갈렸던 점 / 주의할 점

- `final`은 참조를 못 바꾸게 할 뿐, **객체 내부 상태가 바뀌는 건 막지 못한다.** final이라고 스레드 안전한 게 아니다.
- 같은 관점: 싱글톤 빈에 변경 가능한 상태를 두면 안 된다. → [싱글톤 빈의 상태](../spring/singleton-bean-state.md)

## 참고

- `java.text.SimpleDateFormat` Javadoc - Synchronization
- `java.time.format.DateTimeFormatter` Javadoc
