# Redis 캐시에서 SliceImpl 역직렬화 실패

> 작성일: 2026-11-19

## 한 줄 요약

`GenericJackson2JsonRedisSerializer`로 Spring Data의 `SliceImpl`을 캐시하면, 기본 생성자가 없고 타입 정보(`@class`)가 빠져서 **꺼낼 때 역직렬화가 실패**한다.

## 상황

- 개인 포트폴리오 프로젝트(Spring Boot + Vue 3, Cloud Run 배포).
- 무한 스크롤 목록 API의 결과(`Slice<ProjectListDto>`)를 `@Cacheable`로 Redis에 캐시했다.
- 전체 조회는 되는데 카테고리별 조회에서 500 에러가 났다.

```
SerializationException: Could not read JSON: Could not resolve subtype of
[simple type, class java.lang.Object]: missing type id property '@class'
```

## 원인

1. `GenericJackson2JsonRedisSerializer`는 JSON에 `@class`를 넣어 두었다가, 꺼낼 때 그 타입으로 복원한다.
2. `SliceImpl`은 Jackson이 쓸 수 있는 **기본 생성자나 `@JsonCreator`가 없어서** 그대로는 복원할 수 없다.
3. 내부 DTO 목록에도 타입 정보가 일관되게 들어가지 않아 하위 타입을 찾지 못했다.

## 해결 과정

- `RedisCacheManager`에 쓸 `ObjectMapper`를 별도로 구성했다.
  - `activateDefaultTyping`으로 타입 정보를 넣고, `PolymorphicTypeValidator`에서 허용 패키지를 지정했다.
  - `JavaTimeModule`을 등록해 날짜 타입도 처리했다.
- `SliceImpl`용 **커스텀 Serializer/Deserializer 모듈**을 만들어 `content`, `number`, `size`, `hasNext`를 직접 쓰고 읽게 했다. 필드가 없을 때를 대비한 null 체크도 넣었다.
- DTO에 `@JsonCreator`, `@JsonProperty`를 붙여 생성자로 복원되게 했다.

## 배운 점

- 캐시에 **프레임워크 내부 구현 클래스를 그대로 넣으면** 직렬화 형식이 그 클래스 구조에 묶인다. 버전이 바뀌면 또 깨질 수 있다.
- 더 단순한 방법: `Slice` 대신 캐시 전용 DTO(`List<Dto> content`, `boolean hasNext`)를 만들어 저장하면 커스텀 모듈이 필요 없다.
- `activateDefaultTyping`은 허용 범위를 너무 넓히면 역직렬화 취약점이 될 수 있어서 `PolymorphicTypeValidator`로 범위를 좁혀야 한다.

## 참고

- Spring Data Redis Reference - Serializers
- Jackson `ObjectMapper.activateDefaultTyping` Javadoc
