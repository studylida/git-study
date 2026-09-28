# 자동 테스트 — PR마다 테스트 자동 실행

## 핵심 개념

### 자동 테스트란?

PR을 올리면 GitHub Actions가 자동으로 테스트를 실행하고, 결과에 따라 머지 가능 여부를 결정하는 시스템.

```
PR 생성/업데이트
       │
       ▼
GitHub Actions 트리거 (on: pull_request)
       │
       ▼
Runner에서 테스트 실행
       │
       ├── 통과 ✅ → "All checks passed" → 머지 가능
       │
       └── 실패 ❌ → "Some checks failed" → 머지 차단 (설정 시)
```

### PR 이벤트 트리거

```yaml
on:
  pull_request:
    branches: [main]
```

- main으로 향하는 PR이 **생성**될 때 실행
- PR에 **새 커밋이 추가**될 때도 실행

### Status Check

워크플로우 실행 결과가 PR 페이지에 표시됨:
- ✅ All checks passed → 머지 가능
- ❌ Some checks failed → 머지 차단 (Required 설정 시)

Status Check 이름은 워크플로우의 **Job 이름**에서 나옴.

### Required Status Check (필수 체크)

Status Check가 표시되는 것만으론 머지 차단이 안 됨. **진짜로 차단**하려면:

```
Settings → Rules → Rulesets
  → "Require status checks to pass" 체크
  → 워크플로우 Job 이름 추가
  → Save
```

### exit code가 성패를 결정

- **exit 0** → 성공 ✅ → 다음 step 진행
- **exit 1** → 실패 ❌ → 워크플로우 중단

테스트 프레임워크(pytest, jest, go test 등)는 실패 시 자동으로 exit 1 반환.

---

## 실전 워크플로우 예시

### 셸 스크립트 테스트 (우리가 만든 것)

```yaml
name: CI

on:
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run tests
        run: bash tests/check_notes.sh
```

### Node.js 프로젝트 테스트

```yaml
name: CI

on:
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test
```

### push + pull_request 동시 사용

```yaml
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
```

- `pull_request` → PR에서 머지 전 검증
- `push` → main에 직접 커밋이 들어왔을 때도 검증

---

## 명령어/설정 정리표

| 항목 | 설명 |
|------|------|
| `on: pull_request` | PR 이벤트에 반응하는 트리거 |
| `branches: [main]` | main으로 향하는 PR만 대상 |
| `actions/setup-node@v4` | Node.js 환경 설치 Action |
| `npm ci` | 의존성 설치 (install보다 빠르고 정확) |
| Required Status Check | Ruleset에서 설정, 테스트 실패 시 머지 차단 |

---

## 주의사항 & 흔한 실수

### 1. `action/checkout` vs `actions/checkout`
- **s** 빠뜨리면 `repository not found` 에러! 반드시 `actions/checkout`

### 2. Required Status Check를 안 켜면?
- Status Check는 보이지만 실패해도 **머지 가능** — 의미 없음
- 반드시 Ruleset에서 "Require status checks to pass" 설정

### 3. `-d` vs `-D` (squash merge 후 브랜치 삭제)
- `git branch -d`는 커밋 해시로 HEAD 포함 여부를 판단
- squash merge로 해시가 달라지면 "머지 안 됨"으로 판단 → `-d` 거부
- 이때 `-D`로 강제 삭제

### 4. upstream 설정 안 하면 push 실패
- `git push`만 치면 `no upstream branch` 에러
- 첫 push 시 `git push -u origin 브랜치명` 필요
