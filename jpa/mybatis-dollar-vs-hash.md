# MyBatis #{} vs ${}

> 작성일: 2026-11-19

## 한 줄 요약

`#{}`는 PreparedStatement의 **바인딩 파라미터(?)**이고, `${}`는 SQL 문자열에 **값을 그대로 이어 붙인다.** 사용자 입력을 `${}`에 넣으면 SQL 인젝션이 된다.

## 내용

```xml
<!-- 위험 -->
<select id="findByWarehouse" resultType="Inbound">
    SELECT * FROM inbound WHERE warehouse_code = '${warehouseCode}'
</select>

<!-- 안전 -->
<select id="findByWarehouse" resultType="Inbound">
    SELECT * FROM inbound WHERE warehouse_code = #{warehouseCode}
</select>
```

- `${}`에 `' OR '1'='1`이 들어오면 `WHERE warehouse_code = '' OR '1'='1'`이 되어 전체 데이터가 조회된다.
- `#{}`는 `WHERE warehouse_code = ?`로 만들고 값을 따로 바인딩해서, 입력이 SQL 문법으로 해석되지 않는다.
- `#{}`는 실행계획 캐시 재사용에도 유리하다. 값만 바뀌고 SQL 문자열은 같기 때문이다.

### ${}가 필요한 경우

바인딩 파라미터는 **값** 자리에만 쓸 수 있다. 컬럼명, 테이블명, 정렬 방향처럼 SQL 구조가 바뀌는 부분은 `#{}`로 안 된다.

```xml
ORDER BY ${sortColumn} ${sortDirection}
```

이때는 반드시 **화이트리스트**로 검증한다.

```java
private static final Set<String> SORTABLE = Set.of("created_at", "invoice_no");

if (!SORTABLE.contains(sortColumn)) {
    throw new IllegalArgumentException("정렬할 수 없는 컬럼");
}
String direction = "DESC".equalsIgnoreCase(sortDirection) ? "DESC" : "ASC";
```

### LIKE 검색

```xml
<!-- 위험: '%${keyword}%' -->
WHERE name LIKE CONCAT('%', #{keyword}, '%')   <!-- MySQL -->
WHERE name LIKE '%' + #{keyword} + '%'         <!-- SQL Server -->
```

## 헷갈렸던 점 / 주의할 점

- 레거시 이관을 하다 보면 `${}`가 섞인 SQL을 그대로 옮기기 쉽다. 이관할 때 `${}`는 하나씩 확인해서 값이면 `#{}`로 바꾼다.
- 코드 리뷰에서 MyBatis XML이 보이면 `${}`부터 찾는다.

## 참고

- MyBatis 3 Reference - Mapper XML Files (String Substitution)
