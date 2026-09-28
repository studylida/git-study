# .git 폴더 탐험 — refs, objects, HEAD 파일 구조

## 핵심 개념: .git/ 폴더가 Git의 전부

Git 명령어들은 결국 `.git/` 폴더 안의 파일을 읽고 쓰는 것. 이 폴더의 구조를 이해하면 Git의 내부 동작이 보인다.

---

## .git/ 전체 구조

```
.git/
├── HEAD              ← 현재 체크아웃된 브랜치 포인터
├── config            ← 로컬 설정 (remote, branch 등)
├── index             ← Staging Area (바이너리)
├── description       ← GitWeb용 (거의 안 씀)
│
├── objects/          ← 모든 Git 객체 (blob, tree, commit)
│   ├── 00/           ← 해시 앞 2자리 = 폴더
│   ├── 0e/           ← 나머지 38자리 = 파일명
│   ├── info/
│   └── pack/         ← 압축된 객체 (packfile)
│
├── refs/             ← 참조 (브랜치, 태그 등)
│   ├── heads/        ← 로컬 브랜치
│   ├── tags/         ← 태그
│   └── remotes/      ← 원격 추적 브랜치
│
├── hooks/            ← Git 이벤트 훅 스크립트
├── logs/             ← reflog 기록
└── packed-refs       ← refs 최적화 파일
```

---

## HEAD — "지금 어디에 있는가?"

`.git/HEAD`는 현재 위치를 가리키는 텍스트 파일.

```bash
# 일반 상태 (브랜치에 있을 때)
$ cat .git/HEAD
ref: refs/heads/main     ← "main 브랜치를 보고 있어"

# Detached HEAD 상태
$ cat .git/HEAD
0ed835c0267fce84...      ← 해시값이 직접 적혀있음
```

### HEAD가 커밋을 찾는 과정

```
.git/HEAD
  → "ref: refs/heads/main"
    → .git/refs/heads/main
      → "0010450..." (커밋 해시)
        → .git/objects/00/10450... (커밋 객체)
```

---

## refs/ — 브랜치와 태그의 정체

### 브랜치 = 커밋 해시가 적힌 파일 1개

```bash
$ cat .git/refs/heads/main
00104509955bda931de297594c3d9c2502236da1

$ git log -1 --format="%H"
00104509955bda931de297594c3d9c2502236da1
# ↑ 완벽히 일치!
```

| 동작 | 내부에서 벌어지는 일 |
|------|---------------------|
| 브랜치 생성 | `refs/heads/`에 파일 1개 생성 |
| 브랜치 삭제 | 해당 파일 삭제 |
| 커밋 | 파일 내용을 새 커밋 해시로 갱신 |

### 직접 증명: 파일로 브랜치 만들기

```bash
# 파일 생성 → 브랜치 등장!
git log -1 --format="%H" > .git/refs/heads/test-branch
git branch   # test-branch가 보임!

# 파일 삭제 → 브랜치 사라짐!
rm .git/refs/heads/test-branch
git branch   # test-branch가 없음!
```

### refs 구조

```
refs/
├── heads/              ← 로컬 브랜치
│   └── main            ← main 브랜치
├── tags/               ← 태그 (구조 동일)
│   └── v1.0.0
└── remotes/origin/     ← 원격 추적 브랜치
    ├── main
    └── HEAD
```

---

## objects/ — Git의 데이터베이스

모든 blob, tree, commit이 여기 저장된다.

```
objects/
├── 00/10450...   ← 해시 앞 2자리 = 폴더, 38자리 = 파일
├── 0e/d835c...
├── info/
└── pack/         ← packfile (압축)
    ├── pack-xxx.idx   ← 인덱스 (검색용)
    └── pack-xxx.pack  ← 실제 데이터
```

### 왜 앞 2자리로 폴더를 나누나?

한 폴더에 파일이 수만 개 이상 쌓이면 파일시스템 성능이 떨어진다. 앞 2자리로 나누면 최대 **256개(00~ff) 폴더에 분산**되어 검색이 빨라진다.

### pack 파일

객체가 많아지면 Git이 자동으로 loose object(개별 파일)를 하나의 pack 파일로 압축한다.
- `git gc` 실행 시
- `git push` / `git fetch` 시

---

## index — Staging Area의 실체

```bash
# 바이너리 파일이라 직접 못 읽음
$ file .git/index
.git/index: Git index, ...

# 내용 확인 명령어
$ git ls-files --stage
100644 93bcbf20c072... 0    .github/workflows/ci.yml
100644 f5cac6ae9d1d... 0    .gitignore
100644 cb82b87e7c97... 0    CLAUDE.md
```

각 줄: `{파일모드} {blob 해시} {스테이지 번호} {파일 경로}`

---

## config — 로컬 설정

```ini
[remote "origin"]
    url = https://github.com/rometemp0613/git-study.git
    fetch = +refs/heads/*:refs/remotes/origin/*

[branch "main"]
    remote = origin
    merge = refs/heads/main
```

### fetch 매핑 규칙

```
fetch = +refs/heads/*:refs/remotes/origin/*
         ^^^^^^^^^^^^  ^^^^^^^^^^^^^^^^^^^^^^^
         원격의 브랜치들    로컬에 저장할 위치
```

- `git fetch` 시 원격의 `refs/heads/*`를 로컬 `refs/remotes/origin/*`에 복사
- `+` = 강제 업데이트 (fast-forward 아니어도 덮어쓰기)

---

## packed-refs — refs 최적화

refs 파일이 많아지면 Git이 하나의 파일로 합친다.

```bash
$ cat .git/packed-refs
# pack-refs with: peeled fully-peeled sorted
507c0e79... refs/heads/main
507c0e79... refs/remotes/origin/main
```

**우선순위**: `refs/` 폴더의 파일이 packed-refs보다 우선. 둘 다 있으면 파일이 이김.

---

## 명령어 정리표

| 명령어 | 설명 |
|--------|------|
| `cat .git/HEAD` | 현재 HEAD가 가리키는 브랜치 확인 |
| `cat .git/refs/heads/<브랜치>` | 브랜치가 가리키는 커밋 해시 확인 |
| `cat .git/packed-refs` | 압축된 refs 보기 |
| `cat .git/config` | 로컬 설정 보기 |
| `git ls-files --stage` | Staging Area 내용 보기 (파일모드, blob 해시, 경로) |
| `ls .git/objects/` | 저장된 객체 폴더 목록 |
| `ls .git/refs/heads/` | 로컬 브랜치 목록 (파일로) |

---

## 전체 그림: Git의 내부 동작

```
git add hello.txt
  1. hello.txt 내용으로 blob 객체 생성 → objects/에 저장
  2. index 파일 업데이트 (Staging Area)

git commit -m "message"
  1. index로부터 tree 객체 생성 → objects/에 저장
  2. tree + parent + author + message로 commit 객체 생성 → objects/에 저장
  3. refs/heads/main 파일을 새 커밋 해시로 갱신

git switch -c new-branch
  1. refs/heads/new-branch 파일 생성 (현재 커밋 해시 기록)
  2. HEAD 파일을 "ref: refs/heads/new-branch"로 변경
```

---

## 주의사항 & 흔한 실수

- **`.git/` 폴더를 직접 수정하는 건 학습용으로만**: 실무에서는 Git 명령어를 사용할 것
- **packed-refs vs refs/ 파일**: 둘 다 있으면 refs/ 파일이 우선. packed-refs의 값은 무시됨
- **objects 폴더 분산 이유**: "약속"이 아니라 파일시스템 성능 때문 (한 폴더에 파일 과다 방지)
