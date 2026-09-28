# Git Flow - 브랜치 전략

## 핵심 개념

### Git Flow란?
- Vincent Driessen이 2010년에 제안한 **체계적인 브랜치 관리 전략**
- 5가지 브랜치를 역할별로 나눠서 사용
- 기능 개발, 릴리스 준비, 긴급 수정을 **동시에 체계적으로** 관리 가능

### 왜 필요할까?
- 새 기능 개발 중 **긴급 버그 수정**이 필요할 때
- 릴리스 준비 중 **다른 팀은 계속 개발**해야 할 때
- 여러 버전을 **동시에 관리**해야 할 때

## 5가지 브랜치

```
영구 브랜치 (항상 존재)
├── main       ← 출시된 코드 (= 프로덕션)
└── develop    ← 다음 출시 준비 중인 코드

임시 브랜치 (필요할 때 생성 → 완료 후 삭제)
├── feature/*  ← 새 기능 개발
├── release/*  ← 출시 전 마무리
└── hotfix/*   ← 긴급 버그 수정
```

| 브랜치 | 분기 원본 | 머지 대상 | 용도 |
|--------|-----------|-----------|------|
| `main` | - | - | 배포된 코드 (영구) |
| `develop` | main | - | 다음 릴리스 준비 (영구) |
| `feature/*` | develop | develop | 새 기능 개발 |
| `release/*` | develop | main + develop | 출시 마무리 |
| `hotfix/*` | main | main + develop | 긴급 버그 수정 |

### 각 브랜치 상세

**main**
- 항상 배포 가능한 상태 유지
- 직접 커밋 금지
- 머지할 때마다 태그(v1.0.0 등) 붙임

**develop**
- 다음 출시를 준비하는 통합 브랜치
- 모든 feature가 여기로 머지됨
- "개발 서버"에 배포되는 코드

**feature/***
- 새 기능 하나당 하나의 브랜치
- develop에서 분기 → develop으로 머지
- 예: `feature/login`, `feature/search`

**release/***
- 출시 직전 마무리 (버그 수정, 버전 번호 변경)
- develop에서 분기 → **main + develop 둘 다**에 머지
- release 중에도 develop에서 다음 버전 개발 가능

**hotfix/***
- 프로덕션 긴급 버그 수정
- **main에서 분기** → **main + develop 둘 다**에 머지
- 예: `hotfix/1.0.1`, `hotfix/payment-fix`

## 전체 흐름 다이어그램

```
main     ●─────────────────●──────●
          \               / hotfix/
develop    ●──●──●──●──●──●───●──●
              \  /  \      /
         feature/  release/
```

### feature 흐름

```
develop ──●──●──────●──●──
            \      /
feature/login ●──●
```

### release 흐름

```
main    ──────────────●  (v1.0.0 태그)
                      /
release/1.0.0  ●──●──●
              /        \
develop ──●──●──────────●──
```

### hotfix 흐름

```
main    ──●──────●  (v1.0.1 태그)
           \    /
hotfix/1.0.1 ●
              \
develop ──●────●──
```

## `--no-ff` 옵션

Git Flow에서 머지할 때 `--no-ff` (No Fast-Forward)를 사용하는 이유:

```
# 일반 merge (fast-forward)
main ──●──●──●  ← 브랜치가 있었는지 알 수 없음

# --no-ff merge
main ──●─────────●  ← 머지 커밋으로 분기/합류 기록 남음
        \       /
feature  ●──●──●
```

- fast-forward는 포인터만 옮겨서 **브랜치 히스토리가 사라짐**
- `--no-ff`는 무조건 머지 커밋을 만들어서 **"어디서 갈라졌다가 합쳐졌는지"** 추적 가능

## 실습 가이드 (따라하기)

### 준비

```bash
mkdir ~/git-flow-practice && cd ~/git-flow-practice
git init
echo "v1.0.0" > version.txt
git add version.txt
git commit -m "init: v1.0.0 출시"
git tag v1.0.0
git switch -c develop
```

### feature 브랜치 → develop 머지

```bash
git switch -c feature/login
echo "로그인 기능" > login.txt
git add login.txt
git commit -m "feat: 로그인 기능 추가"

git switch develop
git merge feature/login --no-ff -m "Merge feature/login into develop"
git branch -d feature/login
```

### release 브랜치 → main + develop 머지

```bash
git switch -c release/1.1.0
echo "v1.1.0" > version.txt
git add version.txt
git commit -m "chore: 버전 1.1.0으로 업데이트"

# main에 머지 + 태그
git switch main
git merge release/1.1.0 --no-ff -m "Merge release/1.1.0"
git tag v1.1.0

# develop에도 머지
git switch develop
git merge release/1.1.0 --no-ff -m "Merge release/1.1.0 into develop"
git branch -d release/1.1.0
```

### hotfix 브랜치 → main + develop 머지

```bash
git switch main
git switch -c hotfix/1.1.1
echo "결제 버그 수정" > bugfix.txt
git add bugfix.txt
git commit -m "fix: 결제 오류 긴급 수정"

# main에 머지 + 태그
git switch main
git merge hotfix/1.1.1 --no-ff -m "Merge hotfix/1.1.1"
git tag v1.1.1

# develop에도 머지 (중요! 안 하면 다음 릴리스에 수정이 빠짐)
git switch develop
git merge hotfix/1.1.1 --no-ff -m "Merge hotfix/1.1.1 into develop"
git branch -d hotfix/1.1.1
```

### 결과 확인

```bash
git log --oneline --graph --all
```

## 주의사항 & 흔한 실수

### 커밋 안 하고 브랜치 전환하면?
- add/commit 안 한 변경사항은 **어떤 브랜치에도 속하지 않음**
- Working Directory에 떠돌아다니면서 **어느 브랜치로 전환해도 따라다님**
- 브랜치 전환 시 `M filename` 메시지가 나오면 커밋 안 된 변경사항이 있다는 뜻
- **반드시 add + commit 하고 브랜치 전환할 것!**

### hotfix/release는 반드시 두 곳에 머지
- main에만 머지하고 develop을 깜빡하면 → 다음 릴리스에 수정 내용이 빠짐
- **main + develop 둘 다 머지**하는 걸 잊지 말 것

### 핵심 규칙 요약
1. **feature** → develop에서 나와서 develop으로
2. **release** → develop에서 나와서 main + develop으로
3. **hotfix** → main에서 나와서 main + develop으로
4. main에는 직접 커밋 안 함
5. main에 머지할 때마다 태그 붙임
6. 머지 시 `--no-ff` 사용
