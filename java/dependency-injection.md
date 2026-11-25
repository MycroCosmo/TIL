# 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

> 작성일: 2026-11-25

## 한 줄 요약

클래스가 사용하는 교체 가능한 협력 객체는 직접 생성하기보다 외부에서 전달받는다.

## 내용

```java
class OrderService {
    private final OrderRepository repository = new JpaOrderRepository();
}
```

이 방식은 구현체를 내부에서 고정한다. 생성자 주입으로 바꾸면 서비스의 생성 책임과 사용 책임을 분리할 수 있다.

```java
interface OrderRepository {
    void save();
}

class OrderService {
    private final OrderRepository repository;

    OrderService(OrderRepository repository) {
        this.repository = java.util.Objects.requireNonNull(repository);
    }

    void order() {
        repository.save();
    }
}
```

```java
OrderService service = new OrderService(new JpaOrderRepository());
// 테스트에서는 가짜 구현체를 전달할 수 있다.
```

Spring에서는 생성자가 하나인 빈에 대해 컨테이너가 의존 빈을 주입한다. Lombok의 @RequiredArgsConstructor는 생성자를 생성할 뿐, 주입 자체는 Spring이 담당한다.

## 헷갈렸던 점 / 주의할 점

- DI는 Spring 없이도 가능하다.
- 인터페이스는 DI의 필수 조건은 아니지만 구현체 교체에 유리하다.
- DTO, 값 객체, 지역 컬렉션까지 무조건 주입할 필요는 없다.
- 설정이 다른 자원이나 외부 시스템에 의존하는 객체는 주입을 고려한다.
- 필요할 때 자원을 생성해야 한다면 Supplier<T> 같은 팩터리를 주입할 수도 있다.

## 참고

- Effective Java 아이템 5
- Spring Framework: https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-collaborators.html
