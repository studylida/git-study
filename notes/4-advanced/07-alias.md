# Git Alias - 자주 쓰는 명령어 단축키

## 핵심 개념

Git Alias는 자주 쓰는 긴 명령어를 짧은 단축키로 등록하는 기능.

## 기본 문법

```bash
git config --global alias.<단축키> '<원래 명령어>'
```

`git` 뒤에 오는 부분만 적음. 앞의 `git`은 빼고!

## 자주 쓰는 alias 예시

```bash
git config --global alias.st 'status'
git config --global alias.co 'checkout'
git config --global alias.br 'branch'
git config --global alias.ci 'commit'
git config --global alias.cm 'commit -m'
git config --global alias.lo 'log --oneline'
git config --global alias.lg 'log --oneline --graph --all --decorate'
```

## alias 관리

```bash
# 등록된 alias 전체 보기
git config --global --get-regexp alias

# 특정 alias 확인
git config --global alias.st

# alias 삭제
git config --global --unset alias.st

# 설정 파일 직접 편집
git config --global --edit
```

## 저장 위치

| 옵션 | 저장 파일 | 범위 |
|------|----------|------|
| `--global` | `~/.gitconfig` | 모든 저장소 |
| `--local` (기본) | `.git/config` | 해당 저장소만 |

## 설정 파일 형식

`~/.gitconfig` 파일을 열면:

```ini
[alias]
    st = status
    co = checkout
    br = branch
    lg = log --oneline --graph --all --decorate
```

## 셸 명령어 alias (! 접두사)

Git 명령어가 아닌 셸 명령어는 `!`를 앞에 붙여서 등록:

```bash
git config --global alias.visual '!gitk'
```

## 핵심 정리

- alias는 `git config`로 등록하거나 `~/.gitconfig` 파일을 직접 편집
- `--global`은 전체, `--local`은 해당 레포만
- 셸 명령어는 `!` 접두사 필수
- `--get-regexp alias`로 목록 확인, `--unset`으로 삭제

---

## 실습 가이드: alias 등록부터 활용까지

### Step 1: 기본 alias 등록

```bash
# 자주 쓰는 단축키 등록
git config --global alias.st 'status'
git config --global alias.co 'checkout'
git config --global alias.sw 'switch'
git config --global alias.br 'branch'
git config --global alias.ci 'commit'
git config --global alias.cm 'commit -m'
git config --global alias.lo 'log --oneline'

# 사용해보기
git st          # = git status
git lo          # = git log --oneline
git br          # = git branch
```

### Step 2: 복잡한 명령어 단축

```bash
# 예쁜 로그 보기
git config --global alias.lg 'log --oneline --graph --all --decorate'
git lg
# → 그래프로 브랜치 히스토리가 보임!

# 최근 커밋 수정
git config --global alias.amend 'commit --amend --no-edit'

# unstage (add 취소)
git config --global alias.unstage 'restore --staged'
```

### Step 3: 셸 명령어 alias (! 접두사)

```bash
# Git 명령어가 아닌 셸 명령어 등록
# ⚠️ 반드시 작은따옴표(') 사용! 큰따옴표(")면 bash가 !를 특수문자로 해석함

git config --global alias.visual '!gitk'
git config --global alias.root '!pwd'
# → git root 하면 현재 경로 출력
```

### Step 4: 등록된 alias 관리

```bash
# 전체 alias 목록 보기
git config --global --get-regexp alias
# → alias.st status
# → alias.co checkout
# → ...

# 특정 alias 내용 확인
git config --global alias.lg
# → log --oneline --graph --all --decorate

# alias 삭제
git config --global --unset alias.visual

# 설정 파일 직접 편집 (한꺼번에 수정할 때 편함)
git config --global --edit
# → [alias] 섹션에서 직접 추가/삭제 가능
```

### Step 5: ~/.gitconfig 직접 편집

```bash
# 파일을 직접 열어서 한 번에 설정
cat ~/.gitconfig
# → [alias] 섹션 확인

# 직접 편집으로 추가하는 게 여러 개 등록할 때 더 빠름
# [alias]
#     st = status
#     lg = log --oneline --graph --all --decorate
#     last = log -1 HEAD
```

---

## 주의사항 & 흔한 실수

- **bash에서 `!`는 특수문자!** `git config --global alias.xxx "!command"` 처럼 큰따옴표를 쓰면 bash가 `!`를 히스토리 확장으로 해석해서 에러남. **반드시 작은따옴표(`'`)를 사용**할 것:
  ```bash
  # 틀린 예 (에러 발생)
  git config --global alias.visual "!gitk"
  # → bash: !gitk: event not found

  # 맞는 예
  git config --global alias.visual '!gitk'
  ```
- alias 이름이 기존 Git 명령어와 겹치면 **alias가 무시됨** (예: `alias.status`는 안 먹힘)
- `--local` alias는 해당 레포의 `.git/config`에 저장됨 → 다른 레포에서 안 먹힘
- alias가 너무 많으면 오히려 헷갈림 → 진짜 자주 쓰는 것만 등록하는 게 좋음
- 팀 프로젝트에서는 alias에 의존하지 말 것 — 다른 팀원의 환경에는 없을 수 있음
