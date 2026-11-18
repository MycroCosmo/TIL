# 첫 오픈소스 PR: Spring Boot 문서 수정과 DCO

> 작성일: 2026-11-19

## 한 줄 요약

Spring Boot 레퍼런스 문서의 gRPC Gradle 예제에서 **짝이 맞지 않는 중괄호**를 찾아 PR을 보냈고 머지됐다. 작은 수정이지만 오픈소스 기여 절차(fork, DCO 서명, 리뷰)를 처음부터 끝까지 겪어봤다.

## 내용

### 무엇을 고쳤나

- PR: [spring-projects/spring-boot#51956](https://github.com/spring-projects/spring-boot/pull/51956) "Fix Gradle example for servlet-based gRPC servers"
- 서블릿 컨테이너에서 gRPC 서버를 띄우는 방법이 궁금해서 문서를 읽다가, Gradle 예제에 여는 중괄호만 있고 닫는 중괄호가 없어서 그대로 복사하면 빌드 스크립트로 해석되지 않는 걸 발견했다.
- 단순 오타라고 넘기기 전에 실제로 문제가 되는지 직접 확인한 뒤 PR을 보냈다.
- 메인테이너가 리뷰 후 main과 4.1.x 브랜치에 머지했다.

### 기여 절차

1. 저장소를 fork하고 브랜치를 만든다.
2. `CONTRIBUTING.md`를 먼저 읽는다. 커밋 규칙, 코드 포맷, 브랜치 기준이 프로젝트마다 다르다.
3. Spring 프로젝트는 **DCO(Developer Certificate of Origin) 서명**이 필요하다.

```bash
git commit -s -m "Fix Gradle example for servlet-based gRPC servers"
# 커밋 메시지 끝에 Signed-off-by: 이름 <이메일> 이 붙는다
```

4. PR 본문에 무엇이 문제였고 어떻게 고쳤는지 짧게 적는다.
5. 리뷰를 기다린다. 메인테이너가 직접 다듬어서 머지하는 경우도 흔하다.

## 헷갈렸던 점 / 주의할 점

- DCO는 CLA처럼 별도 서명 페이지가 아니라 **커밋마다** `Signed-off-by`를 붙이는 방식이다. 빠뜨리면 체크가 실패한다. 이미 올린 커밋은 `git commit --amend -s` 후 force push로 고친다.
- `status: ideal-for-contribution` 같은 라벨이 붙은 이슈는 경쟁이 심하다. 문서를 직접 읽거나 실무에서 겪은 문제가 더 좋은 기여 재료다.
- 작은 기여라도 **과정에서 배운 점**을 기준으로 이야기하면 된다.

## 참고

- Spring Boot `CONTRIBUTING.md`
- https://developercertificate.org/
