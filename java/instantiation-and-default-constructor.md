# 인스턴스화와 기본 생성자

> 작성일: 2026-11-25

## 한 줄 요약

인스턴스화는 객체를 생성하는 과정이고, 초기화는 객체의 초기 상태를 설정하는 과정이다.

## 내용

```java
public class User {
    private String name;
}
```

생성자를 선언하지 않으면 일반 클래스의 컴파일러는 기본 생성자를 제공한다. 위 예시는 다음과 같은 의미다.

```java
public User() {
    super();
}
```

`new User()`는 객체를 할당하고 생성자를 실행한 뒤 참조값을 반환한다. 필드는 먼저 기본값으로 초기화되며, name의 기본값은 null이다.

`javac User.java` 이후 `javap -c -p User`로 생성자의 바이트코드를 확인할 수 있다. 일반적으로 aload_0, invokespecial(Object.<init>), return이 나타난다.

## 헷갈렸던 점 / 주의할 점

- 생성자를 하나라도 명시하면 기본 생성자는 자동 생성되지 않는다.
- 일반 클래스의 기본 생성자 접근 수준은 클래스의 접근 수준을 따른다.
- 부모의 접근 가능한 무인자 생성자가 없으면 암묵적인 super() 호출은 컴파일 오류가 발생한다.
- private 생성자는 외부의 일반적인 생성자 호출을 제한한다. 내부 호출까지 막지는 않는다.
- JVM 최적화에 따라 실제 메모리 할당이 제거될 수 있다.

## 참고

- JLS 8.8.9: https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html#jls-8.8.9
- Effective Java 아이템 4
