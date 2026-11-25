# 정적 팩터리 메서드

> 작성일: 2026-11-25

## 한 줄 요약

정적 팩터리 메서드는 생성자 대신 이름이 있는 static 메서드로 객체를 획득하는 방식이다.

## 내용

```java
public class User {
    private final String name;

    private User(String name) {
        this.name = name;
    }

    public static User of(String name) {
        return new User(name);
    }
}
```

`User.of("Kim")`으로 객체를 생성할 수 있다. private 생성자로 외부의 직접 호출을 제한하고 생성 정책을 팩터리에 둘 수 있다.

- 이름으로 생성 목적을 표현할 때 (of, from, valueOf 등).
- 기존 객체 재사용이나 반환 구현체 선택이 필요할 때.
- 생성 전에 검증하거나 생성 경로를 제한할 때.

## 헷갈렸던 점 / 주의할 점

- 팩터리 호출마다 새 인스턴스가 만들어지는 것은 아니다.
- private 생성자만 있으면 외부에서 일반적인 상속도 제한된다.
- 단순 생성만 필요하면 public 생성자가 더 명확할 수 있다.
- GoF의 Factory Method 패턴과 반드시 같은 개념은 아니다.

## 참고

- Effective Java 아이템 1
