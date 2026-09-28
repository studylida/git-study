# GitHub Flow - 간소화된 브랜치 전략

## 핵심 개념

### GitHub Flow란?
- GitHub 자체가 사용하는 **간소화된 브랜치 전략**
- **main + feature 브랜치만** 사용 (develop, release, hotfix 없음)
- 수시 배포(CI/CD) 환경에 최적화

### 왜 필요할까?
- Git Flow는 5가지 브랜치 + 복잡한 규칙 → 웹앱에는 과함
- 웹 서비스는 하루에도 여러 번 배포 → 릴리스 브랜치가 불필요
- develop이 불필요한 중간 단계가 됨

## 6가지 규칙

| # | 규칙 | 설명 |
|---|------|------|
| 1 | main은 항상 배포 가능 | main에 있는 코드는 언제든 배포할 수 있는 상태 |
| 2 | main에서 바로 브랜치 생성 | develop 안 거침! |
| 3 | 브랜치 이름은 설명적으로 | `add-login-page`, `fix-header-bug` |
| 4 | 수시로 push | 로컬에만 두지 말고 자주 원격에 push |
| 5 | PR로 리뷰 후 머지 | main에 직접 커밋 금지 |
| 6 | 머지하면 즉시 배포 | main = 프로덕션 |

## 다이어그램

```
GitHub Flow:
main ──●──────●──────●──────●── (항상 배포 가능, 유일한 장기 브랜치)
        \    ↗  \    ↗      \    ↗
         ●──●    ●──●        ●──●
        feature  feature    feature
```

## Git Flow vs GitHub Flow 비교

| | Git Flow | GitHub Flow |
|---|---------|------------|
| 장기 브랜치 | main + develop | main만 |
| 브랜치 종류 | 5가지 | 2가지 (main, feature) |
| 릴리스 | release 브랜치로 준비 | main 머지 = 릴리스 |
| 핫픽스 | hotfix 브랜치 따로 | feature 브랜치와 동일 |
| 복잡도 | 높음 | 낮음 |
| 배포 주기 | 정해진 릴리스 주기 | 수시 배포 (CI/CD) |

## 언제 뭘 써야 할까?

| 상황 | 추천 |
|------|------|
| 웹앱, SaaS, 수시 배포 | **GitHub Flow** |
| 모바일/데스크탑 앱 (버전 릴리스) | Git Flow |
| 오픈소스 라이브러리 (v1.0, v2.0) | Git Flow |
| 1인/소규모 팀 | **GitHub Flow** |
| 대규모 팀 + 엄격한 릴리스 관리 | Git Flow |

## 실습 가이드 (따라하기)

### 전체 흐름 — 이걸 매번 반복

```bash
# 1. main 최신 유지
git switch main
git pull origin main

# 2. 설명적인 이름으로 브랜치 생성
git switch -c docs/github-flow

# 3. 작업 + 커밋 + push
git add <파일>
git commit -m "docs: GitHub Flow 학습 완료"
git push -u origin docs/github-flow

# 4. PR 생성 → 리뷰 → 머지
gh pr create --title "docs: 제목" --body "설명"
gh pr merge --squash

# 5. 정리
git switch main
git pull origin main
git branch -d docs/github-flow
```

### 핫픽스도 똑같은 흐름

```bash
git switch main
git pull origin main
git switch -c fix-critical-bug
# 수정 작업...
git add . && git commit -m "fix: 긴급 버그 수정"
git push -u origin fix-critical-bug
gh pr create --title "fix: 긴급 버그" --body "설명"
# 빠르게 리뷰 → 머지 → 끝!
```

## 주의사항 & 흔한 실수

### main이 깨지면 전체가 멈춤
- GitHub Flow에서 main = 프로덕션이므로, main이 깨지면 배포가 멈춤
- 그래서 **PR 리뷰 + 자동 테스트(CI)**가 필수

### develop 없이 불안하다면?
- CI/CD로 자동 테스트가 돌아가면 develop이 필요 없음
- PR에서 테스트 통과해야 머지 가능하게 설정하면 됨

### GitHub Flow ≠ 규칙이 없다
- 단순한 거지 허술한 게 아님
- 6가지 규칙을 잘 지키면 Git Flow만큼 안정적
