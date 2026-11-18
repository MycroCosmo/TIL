# @Transactional 내부 호출과 private 메서드

> 작성일: 2026-11-19

## 한 줄 요약

`@Transactional`은 프록시가 호출을 가로채서 동작하기 때문에, **같은 클래스 안에서 직접 호출하거나 private 메서드에 붙이면 트랜잭션이 적용되지 않는다.**

## 내용

Spring은 `@Transactional`이 붙은 빈을 프록시로 감싼다. 외부에서 들어온 호출만 프록시를 거치면서 트랜잭션을 시작·커밋·롤백한다.

```java
@Service
public class OrderService {

    public void placeOrder(Long id) {
        processOrder(id); // this.processOrder() → 프록시를 거치지 않음
    }

    @Transactional
    public void processOrder(Long id) {
        // 트랜잭션 없이 실행된다
    }
}
```

- `placeOrder()`에서 `processOrder()`를 부르는 건 `this`를 통한 호출이라 프록시를 거치지 않는다.
- 그래서 `processOrder()`에 `rollbackFor`를 붙이는 등 무엇을 설정해도 소용이 없다. 트랜잭션 자체가 없기 때문이다.

### private 메서드

- 프록시는 private 메서드를 오버라이드할 수 없어서 가로챌 수 없다.
- Spring 6부터는 클래스 기반(CGLIB) 프록시에서 protected·package-private 메서드에도 적용되지만, **private은 여전히 적용되지 않는다.**

### 해결 방법

1. 트랜잭션이 필요한 메서드를 **별도 빈으로 분리**한다. (가장 깔끔함)
2. 바깥 진입 메서드(`placeOrder`)에 `@Transactional`을 붙인다.
3. 자기 자신을 주입받아 프록시를 통해 호출한다. (가능은 하지만 구조가 꼬여 보여서 비추천)

## 헷갈렸던 점 / 주의할 점

- "어노테이션을 붙였으니 적용된다"가 아니라 **"프록시를 거쳤는가"**로 판단해야 한다.
- 코드 리뷰에서 이 문제가 있으면 롤백 규칙, 락 같은 다른 문제보다 먼저 말해야 한다. 트랜잭션이 아예 없으면 나머지 설정은 의미가 없다.
- 리뷰할 때 말하는 순서: **동작 자체가 안 되는 것 → 데이터가 틀어지는 것 → 성능**

## 참고

- Spring Framework Reference - Declarative Transaction Management (Using @Transactional)
