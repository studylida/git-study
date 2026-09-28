# Git 객체 (blob, tree, commit)

## 핵심 개념: Git은 키-값 저장소

Git은 모든 데이터를 **객체(Object)**로 저장하고, 각 객체에 **SHA-1 해시**라는 고유 ID를 붙인다. 3가지 핵심 객체가 있다.

---

## 3가지 객체

| 객체 | 한마디 | 저장하는 것 |
|------|--------|------------|
| **blob** | 파일 내용 | 텍스트 데이터만 (파일 이름은 모름!) |
| **tree** | 폴더 구조 | "이 이름은 저 blob/tree를 가리킨다" |
| **commit** | 스냅샷 봉투 | tree + parent + author/committer + message |

### 비유: 서류 보관함

```
커밋 = 봉투
  겉면: 누가, 언제, 왜 (메시지)
  안에: → 폴더 목록 (tree)

tree = 폴더
  "README.md는 3번 서랍에 있어"
  "notes/는 7번 캐비닛에 있어"

blob = 서랍 속 실제 문서
  파일 내용 그 자체 (이름은 모름)
```

---

## 객체 간 관계 (다이어그램)

```
commit ab8f298
  │
  ▼
tree 7897970  (루트 디렉토리)
  ├── 100644 blob f5cac6a  → .gitignore   (파일 = blob)
  ├── 100644 blob cb82b87  → CLAUDE.md
  ├── 040000 tree 53db45f  → logs/        (폴더 = tree)
  ├── 040000 tree 89076c6  → notes/       (폴더 = tree)
  │         └── 040000 tree ddeb8e0 → 1-basics/   (tree 안의 tree)
  │         └── 040000 tree ddab015 → 2-branching/
  │         └── ...
  └── ...
```

### 파일 모드 숫자

| 숫자 | 의미 | 예시 |
|------|------|------|
| `100644` | 일반 파일 | .gitignore, README.md |
| `040000` | 디렉토리 | .github/, logs/, notes/ |
| `100755` | 실행 가능 파일 | 스크립트 등 |

---

## 핵심 원리: 같은 내용은 한 번만 저장

- blob은 **내용 기반 해시**로 식별됨
- 같은 내용의 파일이 10개 있어도 blob은 **1개만** 생김
- 파일 이름만 바꾸면 (`mv a.txt b.txt`):
  - blob은 **재사용** (내용 안 바뀌었으니까)
  - tree만 "이 blob의 이름이 b.txt로 바뀜"이라고 기록
- 커밋 100번 해도 안 바뀐 파일은 blob 1개로 끝!

---

## 명령어 정리표

| 명령어 | 용도 | 예시 |
|--------|------|------|
| `git cat-file -t <해시>` | 객체의 **타입** 확인 | `git cat-file -t HEAD` → `commit` |
| `git cat-file -p <해시>` | 객체의 **내용** 출력 | `git cat-file -p HEAD` → tree, parent, author... |

### 실습: 커밋 → tree → blob 따라가기

```bash
# 1. 최근 커밋 확인
git log --oneline -1

# 2. 커밋 객체 열기 → tree 해시 확인
git cat-file -p HEAD

# 3. tree 객체 열기 → blob/tree 목록 확인
git cat-file -p <tree 해시>

# 4. blob 객체 열기 → 파일 실제 내용 확인
git cat-file -p <blob 해시>
```

---

## commit 객체에 들어있는 4가지 정보

```
tree 7897970...        ← 이 커밋 시점의 전체 디렉토리 구조
parent a896e68...      ← 이전 커밋 (첫 커밋은 parent 없음)
author/committer       ← 작성자 + 커밋한 사람 + 시간
message                ← 커밋 메시지
```

---

## 주의사항 & 흔한 실수

- `git cat-file`에는 **반드시 해시값이나 참조(HEAD 등)**를 넘겨야 함
  - `git cat-file -t notes` ❌ (경로는 안 됨)
  - `git cat-file -p HEAD` ✅
- blob은 파일 이름을 **모른다** — 이름은 tree가 관리함
- 해시값은 앞 7자리 정도만 써도 됨 (유일하면 OK)
