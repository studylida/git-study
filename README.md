# Git & GitHub 학습 기록

**학습 방식**: Codex의 코칭을 받으며 명령어를 직접 실행하는 실습 중심 학습
**학습 시작일**: 2026-01-28

**학습 목표**:
- Git 기초부터 고급 기능까지
- GitHub 활용
- CI/CD 자동화

---

## 진도 체크리스트

### 1단계: 기초

- [x] **3가지 영역 개념**
  - 핵심: Working Directory, Staging Area, Repository
- [x] **init, add, commit, status, log**
  - 핵심: 저장소 생성, 스테이징, 커밋, 상태 확인, 히스토리
- [x] **commit --amend**
  - 핵심: 직전 커밋 수정 (메시지, 파일 추가)
- [x] **diff - 변경사항 비교**
  - 핵심: diff, diff --staged, diff HEAD
- [x] **reset, revert - 되돌리기**
  - 핵심: reset --soft/--mixed/--hard, revert (커밋 취소)
- [x] **.gitignore - 추적 제외 파일 설정**
  - 핵심: 패턴 문법, 전역 gitignore
- [x] **HEAD 개념**
  - 핵심: 현재 위치 포인터, HEAD~1, HEAD^
- [x] **Detached HEAD**
  - 핵심: 브랜치 없이 커밋 직접 체크아웃, 임시 작업
- [x] **restore - 변경사항 취소**
  - 핵심: restore, restore --staged (reset/revert보다 간단)

### 2단계: 브랜치

- [x] **branch, switch/checkout, merge**
  - 핵심: 브랜치 생성/전환/병합, fast-forward vs 3-way merge
- [x] **rebase - 커밋 히스토리 정리**
  - 핵심: rebase, interactive rebase (squash, reword, drop)
- [x] **conflict 해결 - 충돌 상황 대처**
  - 핵심: 충돌 마커, 수동 해결, merge --abort
- [x] **cherry-pick - 특정 커밋만 가져오기**
  - 핵심: cherry-pick <commit>, 다른 브랜치에서 커밋 복사

### 3단계: 원격 저장소 & GitHub

- [x] **remote, push, pull, fetch, clone**
  - 핵심: 원격 저장소 연결, 업로드/다운로드, fetch vs pull
- [x] **원격 추적 브랜치**
  - 핵심: origin/main의 정체, upstream 설정
- [x] **Pull Request**
  - 핵심: PR 생성, 리뷰, 머지, 협업 워크플로우
- [x] **브랜치 보호 규칙**
  - 핵심: main 직접 push 막기, 리뷰 필수 설정
- [x] **Issue & Project**
  - 핵심: 이슈 생성, 라벨, 마일스톤, 프로젝트 보드
- [x] **Fork**
  - 핵심: 오픈소스 기여 방식, upstream 동기화
- [x] **README & Markdown**
  - 핵심: 프로젝트 문서 작성, 마크다운 문법

### 4단계: 고급 기능

- [x] **stash - 작업 임시 저장**
  - 핵심: stash, stash pop, stash list, stash apply
- [x] **tag - 버전 표시**
  - 핵심: tag, annotated tag, tag push
- [x] **시맨틱 버전 관리**
  - 핵심: v1.0.0 규칙 (MAJOR.MINOR.PATCH)
- [x] **reflog - 실수로 삭제한 커밋 복구**
  - 핵심: reflog, 삭제된 브랜치/커밋 복구
- [x] **hooks - 커밋 전후 자동 실행 스크립트**
  - 핵심: pre-commit, post-commit, .git/hooks/
- [x] **submodule - 저장소 안의 저장소**
  - 핵심: submodule add, update, 의존성 관리
- [x] **Git Alias - 자주 쓰는 명령어 단축키**
  - 핵심: git config --global alias.co checkout

### 5단계: 협업 전략

- [x] **Git Flow**
  - 핵심: main, develop, feature, release, hotfix 브랜치
- [x] **GitHub Flow**
  - 핵심: 간소화된 전략, main + feature 브랜치
- [x] **Conventional Commits**
  - 핵심: feat, fix, docs, style, refactor, test, chore

### 6단계: CI/CD 자동화

- [x] **GitHub Actions 기초**
  - 핵심: 워크플로우 YAML, on/jobs/steps
- [x] **자동 테스트**
  - 핵심: PR마다 테스트 실행, status check
- [x] **자동 배포**
  - 핵심: main 머지 시 배포, secrets 관리
- [x] **린트/포매팅**
  - 핵심: 코드 품질 자동 검사, prettier, eslint

### 7단계: Git 내부 구조 (심화)

- [x] **Git 객체**
  - 핵심: blob, tree, commit 객체의 관계
- [x] **해시 함수**
  - 핵심: SHA-1, 커밋 ID의 원리
- [x] **.git 폴더 탐험**
  - 핵심: refs, objects, HEAD 파일 구조

### 기본 과정 완주! (37/37)

---

### 8단계: 실무 필수 도구

- [ ] **git bisect**
  - 핵심: 버그 원인 커밋을 이진탐색으로 찾기
- [ ] **git blame**
  - 핵심: 누가, 언제, 왜 이 줄을 바꿨는지 추적
- [ ] **git log 심화**
  - 핵심: --graph, --oneline, 필터링, 파일별 히스토리

### 9단계: 대규모 프로젝트

- [ ] **Git LFS**
  - 핵심: 대용량 파일(이미지, 모델, 바이너리) 관리
- [ ] **Shallow clone & Sparse checkout**
  - 핵심: --depth로 히스토리 제한, 필요한 폴더만 체크아웃
- [ ] **Signed commits (GPG)**
  - 핵심: 커밋 서명, 신뢰성 검증, 보안 중시 회사 요구사항
- [ ] **.gitattributes**
  - 핵심: 파일별 diff/merge 전략, line ending 설정
- [x] **다중 기기 GitHub 동기화 루틴**
  - 핵심: 노트북/본체 작업 시작 전 pull, 종료 전 status/commit/push, 여러 저장소 반복 점검

### 10단계: 고급 기법

- [x] **git worktree**
  - 핵심: 하나의 repo에서 여러 브랜치 동시 작업
- [ ] **merge 전략**
  - 핵심: ours, theirs, recursive, octopus
- [ ] **git rerere**
  - 핵심: 반복되는 충돌 자동 해결 (Reuse Recorded Resolution)
- [ ] **git filter-repo**
  - 핵심: 히스토리 재작성, 민감정보 제거, 대형 파일 정리

### 11단계: Git 너머

- [ ] **GitLab / Bitbucket / Gerrit**
  - 핵심: GitHub 외 플랫폼 비교, Gerrit 코드 리뷰, 회사별 선택 기준

---

## 학습 일지

| 날짜 | 주제 | 주요 내용 | 비고 |
|------|------|----------|------|
| 2026-01-28 | 1단계 기초 | 3가지 영역, init/add/commit/status/log, amend, 브랜치 기초 | 완료 |
| 2026-01-30 | 1단계 기초 | diff, reset, revert, .gitignore | 완료 |
| 2026-01-31 | 2단계 브랜치 | rebase, interactive rebase (squash, reword, drop), 충돌 해결 | 완료 |
| 2026-02-07 | 1단계 기초 | HEAD 개념, HEAD~n, HEAD^, .git/HEAD 파일 | 완료 |
| 2026-02-07 | 1단계 기초 | Detached HEAD, checkout vs switch/restore 정리 | 완료 |
| 2026-02-12 | 1단계 기초 | restore, restore --staged, --source, restore vs reset vs revert | 1단계 완료! |
| 2026-02-13 | 2단계 브랜치 | cherry-pick, --no-commit, --continue/--abort, 충돌 해결 | 2단계 완료! |
| 2026-02-16 | 3단계 원격 | remote, push, pull, fetch, clone, 원격 추적 브랜치 | |
| 2026-02-17 | 3단계 원격 | Pull Request 생성, 리뷰, 머지 (merge/squash/rebase) | |
| 2026-02-18 | 3단계 원격 | 브랜치 보호 규칙, Ruleset, fetch --prune | |
| 2026-02-22 | 3단계 원격 | Issue & Project, 라벨, 마일스톤, closes 키워드 | |
| 2026-02-22 | 3단계 원격 | Fork, upstream 동기화, 오픈소스 기여 흐름 | |
| 2026-02-23 | 3단계 원격 | README & Markdown 문법, 뱃지, 잘 만든 README 구성 | |
| 2026-02-24 | 4단계 고급 | stash 임시 저장, pop/apply, -u, stash 관리 | |
| 2026-02-25 | 4단계 고급 | tag (lightweight/annotated, push), 시맨틱 버전 (MAJOR.MINOR.PATCH) | |
| 2026-02-26 | 4단계 고급 | reflog, 커밋/브랜치 복구, rebase 실수 복구 | |
| 2026-02-26 | 4단계 고급 | hooks, pre-commit/commit-msg/post-commit/pre-push | 2회차 |
| 2026-02-27 | 4단계 고급 | submodule, 추가/업데이트/삭제, 독립 저장소 개념 | |
| 2026-02-28 | 4단계 고급 | Git Alias, 단축키 등록/확인/삭제, ~/.gitconfig | 4단계 완료! |
| 2026-03-01 | 5단계 협업 | Git Flow, 5가지 브랜치 전략, --no-ff | |
| 2026-03-02 | 5단계 협업 | GitHub Flow, 6가지 규칙, Git Flow vs GitHub Flow | |
| 2026-03-03 | 5단계 협업 | Conventional Commits, 7가지 타입, Breaking Change | 5단계 완료! |
| 2026-03-04 | 6단계 CI/CD | GitHub Actions 기초, 워크플로우 YAML, on/jobs/steps | |
| 2026-03-07 | 6단계 CI/CD | 자동 테스트, on: pull_request, Status Check, Required Check | |
| 2026-03-10 | 6단계 CI/CD | 자동 배포, GitHub Pages, Secrets, Environment, CI vs CD | |
| 2026-03-13 | 6단계 CI/CD | 린트/포매팅, ESLint, Prettier, GitHub Actions 자동 검사 | 6단계 완료! |
| 2026-03-14 | 7단계 내부구조 | Git 객체 (blob, tree, commit), cat-file, 키-값 저장소 | |
| 2026-03-16 | 7단계 내부구조 | 해시 함수, SHA-1, 해시 계산 공식, 해시 체인, SHA-256 | |
| 2026-03-16 | 7단계 내부구조 | .git 폴더 탐험, HEAD, refs, objects, index, config | 7단계 완료! 전체 완주! |
| 2026-07-02 | 실전 운용 | 다중 기기 동기화 루틴, studylida 원격 정리, submodule/LFS 활용 | 실습 기록 |
| 2026-08-25 | 수동 작업 능력 회복 | worktree 분리, staged diff, upstream, PR, fetch와 fast-forward, 브랜치 정리 | 5회차 완료 |

---

## 진행 현황

### 기본 과정
- **총 항목**: 37개
- **완료**: 37개
- **진행률**: 100%

### 심화 과정
- **총 항목**: 13개
- **완료**: 2개
- **진행률**: 15%

### 수동 작업 능력 회복

- **과정**: [2주 동안 주 5회, 총 10회, 회당 30분 실습](practice/README.md)
- **진행 상황**: `practice/README.md`의 체크리스트에서 관리

---

## 디렉토리 구조

```text
git-study/
├── AGENTS.md              # Codex 학습 및 코칭 지침
├── README.md              # 전체 진도와 저장소 안내
├── practice/              # 수동 작업 능력 회복 실습
├── notes/                 # 주제별 학습 노트
├── logs/                  # 날짜별 학습 일지
├── review/                # 새 용어, 복습 중, 암기 완료 항목
├── tests/                 # 학습 저장소 구조 검사
├── src/                   # CI/CD 학습용 예제 코드
├── docs/                  # GitHub Pages 배포 파일
└── .github/workflows/     # CI, 린트, 배포 워크플로우
```

---

## GitHub 저장소

**URL**: https://github.com/studylida/git-study

### 학습 후 반드시 실행할 것 (PR 필수!)

> ⚠️ main 브랜치에 보호 규칙이 적용되어 있어 직접 push 불가

```bash
git switch -c docs/학습주제
git add -A
git commit -m "docs: 학습 내용 추가"
git push -u origin docs/학습주제
gh pr create --title "docs: 학습 내용 추가" --body "설명"
gh pr merge --squash
git switch main
git pull
```

### 다른 컴퓨터에서 받아오기

```bash
git clone https://github.com/studylida/git-study.git
```

### 최신 내용 동기화

```bash
git pull
```
