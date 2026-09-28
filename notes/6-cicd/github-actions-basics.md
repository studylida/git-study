# GitHub Actions 기초

## GitHub Actions란?

GitHub 저장소에서 **이벤트가 발생하면 자동으로 작업을 실행**하는 CI/CD 플랫폼.

```
이벤트 발생 → 워크플로우 실행 → 결과 확인
(push, PR 등)   (테스트, 빌드 등)   (✅ 통과 / ❌ 실패)
```

---

## 핵심 용어 5가지

```
┌─ Workflow (.yml 파일) ──────────────────────┐
│                                              │
│  Event: push, pull_request 등                │
│                                              │
│  ┌─ Job 1 ─────────────────────────────┐    │
│  │  runs-on: ubuntu-latest              │    │
│  │                                      │    │
│  │  ┌─ Step 1 ────────────────────┐    │    │
│  │  │  actions/checkout@v4         │    │    │
│  │  └─────────────────────────────┘    │    │
│  │  ┌─ Step 2 ────────────────────┐    │    │
│  │  │  run: npm test               │    │    │
│  │  └─────────────────────────────┘    │    │
│  └──────────────────────────────────────┘    │
└──────────────────────────────────────────────┘
```

| 용어 | 설명 | 비유 |
|------|------|------|
| **Workflow** | 자동화 전체 프로세스. `.yml` 파일 1개 = 워크플로우 1개 | 레시피 전체 |
| **Event** | 워크플로우를 실행시키는 트리거 | "주문 들어옴!" |
| **Job** | 독립된 작업 단위. 기본적으로 **병렬 실행** | 요리사 1명의 담당 |
| **Step** | Job 안에서 **순서대로** 실행되는 각 단계 | 요리 순서 1단계, 2단계... |
| **Runner** | 워크플로우가 실행되는 가상 서버 | 주방 |

---

## 워크플로우 파일 위치

**반드시** 이 경로에 있어야 GitHub가 인식:

```
.github/
└── workflows/
    ├── ci.yml
    ├── deploy.yml
    └── lint.yml
```

---

## YAML 파일 구조

```yaml
name: Hello CI                    # ① 워크플로우 이름

on:                               # ② Event — 언제 실행할지
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:                             # ③ Jobs — 무슨 작업을 할지
  greeting:                       #    Job ID (자유롭게)
    runs-on: ubuntu-latest        #    Runner 선택

    steps:                        # ④ Steps — 순서대로 실행
      - name: 코드 가져오기
        uses: actions/checkout@v4  #    Action 사용

      - name: 인사하기
        run: echo "Hello!"        #    셸 명령어 실행
```

핵심 구조: **on (언제) → jobs (뭘) → steps (어떻게)**

---

## Event 종류

| 이벤트 | 언제 실행 |
|--------|----------|
| `push` | 코드가 push될 때 |
| `pull_request` | PR 생성/업데이트될 때 |
| `schedule` | 크론(cron) 스케줄로 정기 실행 |
| `workflow_dispatch` | 수동으로 버튼 눌러서 실행 |

`branches: [main]`으로 특정 브랜치만 필터링 가능.

---

## uses vs run

| 키워드 | 의미 | 예시 |
|--------|------|------|
| `uses` | 마켓플레이스의 Action 사용 | `actions/checkout@v4` |
| `run` | 셸 명령어 직접 실행 | `echo "Hello"`, `npm test` |

여러 줄 명령어는 `run: |` 사용:

```yaml
- name: 여러 줄 실행
  run: |
    echo "첫 번째 줄"
    echo "두 번째 줄"
```

---

## actions/checkout@v4

Runner는 **빈 서버**이므로 코드가 없음. checkout이 `git clone`으로 코드를 가져옴.

```
네 저장소 (GitHub)          Runner (가상 서버)
┌──────────────┐  checkout  ┌──────────────┐
│ src/         │ ────────→ │ src/         │
│ README.md    │  git clone │ README.md    │
└──────────────┘           └──────────────┘
```

- 코드 필요한 작업 → checkout **필수**
- 단순 셸 명령어 → checkout 불필요

---

## Runner 선택지

| runs-on | OS |
|---------|-----|
| `ubuntu-latest` | Ubuntu Linux |
| `windows-latest` | Windows |
| `macos-latest` | macOS |

---

## Job 실행 순서

- 기본: **병렬 실행**
- 순서 강제: `needs:` 키워드 사용

---

## 실행 확인 명령어

```bash
# 최근 실행 목록
gh run list --limit 5

# 특정 실행 상세
gh run view <실행ID>

# 실행 로그 보기
gh run view <실행ID> --log

# PR의 체크 상태
gh pr checks <PR번호>
```

---

## 주의사항 & 팁

- **들여쓰기 주의**: YAML은 들여쓰기에 민감. `name`, `on`, `jobs`는 줄 맨 앞에서 시작
- **squash merge 후 브랜치 삭제**: `-d` 대신 `-D` (강제 삭제) 필요
- **workflow_dispatch**: `on:` 아래에 추가하면 Actions 탭에 수동 실행 버튼 생성
