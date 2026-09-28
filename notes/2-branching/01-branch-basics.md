# 브랜치 기초

## 브랜치란?

독립적인 작업 공간. 새 기능 개발 시 브랜치를 만들어 작업하고, 완료되면 main에 합침.

```
          feature
            ↓
            ● ─ ●
           /
● ─ ● ─ ●
         ↑
        main
```

---

## git branch

브랜치 관리

```bash
git branch              # 브랜치 목록
git branch 이름          # 브랜치 생성만
git branch -d 이름       # 브랜치 삭제 (merge된 것만)
git branch -D 이름       # 브랜치 강제 삭제
```

---

## git switch

브랜치 이동 (권장)

```bash
git switch 이름          # 브랜치 이동
git switch -c 이름       # 생성 + 이동 동시에
```

### switch vs checkout
- `git checkout`은 구버전 방식
- `git switch`가 브랜치 전용이라 더 명확함

---

## git merge

브랜치 합치기

```bash
git switch main          # main으로 이동
git merge 브랜치명       # 해당 브랜치를 현재 브랜치로 합침
```

### Fast-forward
- main이 그냥 앞으로 이동만 하면 되는 경우
- 새 커밋 없이 포인터만 이동

```
Before:
● ─ ● (main)
     \
      ● ─ ● (feature)

After (fast-forward):
● ─ ● ─ ● ─ ● (main, feature)
```

---

## 브랜치 삭제

- Feature 브랜치는 merge 후 삭제하는 게 일반적
- `main`, `develop` 같은 장기 브랜치는 유지
- 같은 기능 재작업 시 새 브랜치 생성 권장

```bash
git branch -d feature    # merge된 브랜치 삭제
git branch -D feature    # 강제 삭제 (merge 안 됐어도)
```

---

## 브랜치 전략 미리보기

```
main ─────●─────●─────●─────●
           \         /
feature     ●───●───●
```

- `main`: 안정적인 배포 버전
- `feature/*`: 새 기능 개발용
- `hotfix/*`: 긴급 버그 수정용

---

## 주의사항 & 흔한 실수

### 브랜치를 어디서 분기하는지 꼭 확인하라!

브랜치를 만들면 **현재 위치(HEAD)**에서 분기된다.
main에 불필요한 커밋이 있는 상태에서 브랜치를 만들면, 그 커밋들이 새 브랜치에 딸려간다.

```
# 의도한 것
main:  A ─ B
              \
feature:       C ─ D

# 실수한 것 (main에 임시 커밋 X가 있었음)
main:  A ─ B ─ X(불필요)
                 \
feature:          C ─ D    ← X가 딸려옴!
```

**예방법:**
```bash
# 1. 브랜치 만들기 전에 현재 상태 확인
git log --oneline -5       # 최근 커밋 확인

# 2. main이 깨끗한지 확인
git status

# 3. 원하는 커밋에서 브랜치 분기하기
git switch -c feature abc1234    # 특정 커밋에서 분기
```

---

## 실습 가이드 (처음부터 끝까지 따라하기)

아래 명령어를 순서대로 복붙하면 브랜치의 생성, 이동, 합치기, 삭제를 전부 체험할 수 있다.

```bash
# === 준비: 실습용 폴더 만들기 ===
mkdir branch-practice && cd branch-practice
git init

# === 1. main에 기본 커밋 만들기 ===
echo "# 프로젝트" > README.md
git add README.md
git commit -m "init: 프로젝트 시작"

echo "v1 기능" > app.txt
git add app.txt
git commit -m "feat: 기본 기능 추가"

# 현재 상태 확인
git log --oneline
# 출력 예: 2개의 커밋이 보임

# === 2. 브랜치 생성 + 이동 ===
git switch -c feature/login    # 브랜치 생성 + 이동 동시에
git branch                     # 현재 브랜치 목록 (* 표시 확인)

# === 3. feature 브랜치에서 작업 ===
echo "로그인 기능" > login.txt
git add login.txt
git commit -m "feat: 로그인 기능 추가"

echo "로그인 검증" >> login.txt
git add login.txt
git commit -m "feat: 로그인 검증 추가"

# feature에서 커밋 확인
git log --oneline
# 출력 예: 4개의 커밋 (main 2개 + feature 2개)

# === 4. main으로 돌아가서 merge ===
git switch main
git log --oneline              # main에는 아직 2개뿐
git merge feature/login        # fast-forward merge 발생!
git log --oneline              # 이제 4개 커밋

# === 5. 브랜치 삭제 ===
git branch -d feature/login    # merge 완료된 브랜치 삭제
git branch                     # main만 남아있음

# === 6. 정리 ===
cd ..
rm -rf branch-practice
```

### 각 단계에서 확인할 것

| 단계 | 확인 명령어 | 기대 결과 |
|------|------------|----------|
| 브랜치 생성 후 | `git branch` | `* feature/login`과 `main` 표시 |
| feature 커밋 후 | `git log --oneline` | feature 커밋 + main 커밋 모두 보임 |
| main으로 이동 후 | `git log --oneline` | main 커밋만 보임 (feature 커밋 안 보임) |
| merge 후 | `git log --oneline` | 모든 커밋이 일직선으로 보임 (fast-forward) |
| 삭제 후 | `git branch` | `* main`만 남음 |
