# 유틸리티 클래스 제대로 만들기 (JsonUtils 복기)

> 작성일: 2026-11-19

## 한 줄 요약

유틸리티 클래스는 **private 생성자 + static 메서드 + static final 필드**로 만든다. 공유하는 객체는 스레드 안전한지 확인한다.

## 내용

면접에서 설명을 요청받았던 코드와 비슷한 형태다.

```java
public class JsonUtils {
    static ObjectMapper mapper = new ObjectMapper();   // final 없음, package-private

    public String toJson(Object obj) throws Exception { // 인스턴스 메서드
        return mapper.writeValueAsString(obj);
    }
}
```

문제점

1. **`final`이 없다**: 다른 코드가 `JsonUtils.mapper = ...`로 바꿔치기할 수 있다.
2. **접근 제어자가 없다(package-private)**: 같은 패키지에서 필드에 직접 접근할 수 있다.
3. **인스턴스 메서드다**: 상태가 없는 유틸인데 쓰려면 `new JsonUtils()`를 해야 한다.
4. **생성자를 막지 않았다**: 의미 없는 인스턴스가 만들어질 수 있다.

### 개선

```java
public final class JsonUtils {

    private static final ObjectMapper MAPPER = new ObjectMapper();

    private JsonUtils() {
        // 인스턴스화 방지
    }

    public static String toJson(Object obj) {
        try {
            return MAPPER.writeValueAsString(obj);
        } catch (JsonProcessingException e) {
            throw new IllegalArgumentException("JSON 변환 실패", e);
        }
    }
}
```

- `ObjectMapper`는 **설정을 끝낸 뒤에는 스레드 안전**해서 하나를 만들어 공유하는 게 맞다. 생성 비용이 커서 매번 새로 만들면 오히려 손해다.
- 단, 공유한 뒤에 `configure()`로 설정을 바꾸면 안 된다.
- checked 예외를 그대로 던지면 호출하는 쪽마다 try-catch가 필요해서, 의미 있는 unchecked 예외로 감싸는 경우가 많다.

## 헷갈렸던 점 / 주의할 점

- `static`과 `final`은 다른 개념이다. `static`은 "클래스에 하나", `final`은 "다시 할당 못 함"이다.
- Spring을 쓰면 `ObjectMapper`를 빈으로 주입받는 게 더 자연스럽다. Spring Boot가 설정한 `ObjectMapper`(날짜 형식 등)와 일관성도 맞출 수 있다.

## 참고

- Effective Java 아이템 4: 인스턴스화를 막으려거든 private 생성자를 사용하라
- Jackson `ObjectMapper` Javadoc (thread-safety)
