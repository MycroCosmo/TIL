# GitHub 잔디(기여 그래프)가 찍히는 조건

> 작성일: 2026-11-19

## 한 줄 요약

커밋을 했는데 잔디가 안 찍히면 대부분 **커밋 이메일, 브랜치, fork 여부, private 설정** 중 하나가 원인이다.

## 내용

### 커밋이 기여로 집계되려면

1. 커밋 작성자 이메일이 **GitHub 계정에 연결된 이메일**이어야 한다.
2. 저장소의 **기본 브랜치(main)** 또는 `gh-pages` 브랜치에 있어야 한다. 작업 브랜치에만 있으면 머지 전까지 안 보인다.
3. **fork한 저장소**에 한 커밋은 집계되지 않는다. 원본 저장소에 PR이 머지되어야 집계된다.

### 확인할 것

```bash
git config user.email          # 지금 저장소에서 쓰는 이메일
git config --global user.email # 전역 설정
git log --format='%an <%ae>' -1
```

- 회사 PC라 회사 이메일이 전역으로 잡혀 있으면, 개인 저장소에서는 저장소별로 따로 설정한다.

```bash
git config user.email "아이디@users.noreply.github.com"
```

- GitHub의 noreply 이메일(`숫자ID+아이디@users.noreply.github.com`)을 쓰면 개인 이메일을 공개하지 않고도 집계된다.

### private 저장소

- 프로필 설정에서 **Private contributions**를 켜야 잔디에 표시된다. 내용은 공개되지 않고 칸 색만 보인다.

## 헷갈렸던 점 / 주의할 점

- 회사 GitLab·사내 저장소 커밋은 당연히 GitHub 잔디에 반영되지 않는다.
- 잔디 자체를 목표로 의미 없는 커밋을 만들면 커밋 내용을 열어보는 순간 들통난다. 매일 하는 일(알고리즘, TIL, 사이드 프로젝트)의 결과로 쌓이게 하는 게 오래간다.

## 참고

- GitHub Docs - Why are my contributions not showing up on my profile?
