# Thymeleaf 3.1에서 #request, #session이 사라졌다

> 작성일: 2026-11-19

## 한 줄 요약

Thymeleaf 3.1(Spring Boot 3 기본)부터 템플릿에서 `#request`, `#response`, `#session`, `#servletContext` 같은 **서블릿 API 유틸리티 객체를 쓸 수 없다.** 필요한 값은 컨트롤러에서 Model에 담아 넘긴다.

## 내용

### 3.0에서 쓰던 코드

```html
<span th:text="${#request.getParameter('keyword')}"></span>
<span th:text="${#session.getAttribute('loginUser').name}"></span>
<a th:href="${#request.requestURI}">현재 페이지</a>
```

Spring Boot 2.x 기반 프로젝트는 대부분 Thymeleaf 3.0이라 이런 코드가 많다. 3.1로 올리면 템플릿 처리 중 오류가 난다.

### 3.1에서의 대안

```java
@GetMapping("/list")
public String list(@RequestParam(required = false) String keyword,
                   HttpServletRequest request,
                   Model model) {
    model.addAttribute("keyword", keyword);
    model.addAttribute("currentUri", request.getRequestURI());
    return "list";
}
```

```html
<span th:text="${keyword}"></span>
<a th:href="${currentUri}">현재 페이지</a>
```

- 요청 파라미터는 `${param.keyword}`로 계속 읽을 수 있다.
- 세션 값은 `${session.loginUser}`처럼 표현식 객체 대신 속성으로 접근할 수 있다.
- 여러 화면에서 공통으로 쓰는 값은 `@ControllerAdvice`의 `@ModelAttribute`로 한 번에 넣는다.

### 이관 작업에서 같이 볼 문법

| 용도 | JSP | Thymeleaf |
| --- | --- | --- |
| 링크 | `<c:url value="/a"/>` | `th:href="@{/a}"` |
| 폼 액션 | `action="${ctx}/save"` | `th:action="@{/save}"` |
| include | `<jsp:include page="..."/>` | `th:replace="~{fragments/header :: header}"` |
| 인라인 JS 값 | `var id = '${id}';` | `<script th:inline="javascript">` + `/*[[${id}]]*/` |

## 헷갈렸던 점 / 주의할 점

- JSP → Thymeleaf 이관 시 레거시 쪽이 Spring Boot 2.x(Thymeleaf 3.0) 문법으로 쓰여 있으면, 타깃 버전에서 그대로 동작하는지 따로 확인해야 한다.
- 템플릿에서 서블릿 객체를 직접 꺼내 쓰는 습관을 버리고, 화면에 필요한 값은 컨트롤러가 명시적으로 넘기는 구조가 테스트하기도 쉽다.

## 참고

- Thymeleaf 3.1 What's New and Migration Guide
