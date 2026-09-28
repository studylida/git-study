# cherry-pick - 특정 커밋만 가져오기

## 핵심 개념

- cherry-pick = 다른 브랜치의 **특정 커밋**을 현재 브랜치에 **복사**
- 복사지 이동이 아님 → 원본 브랜치에 커밋 그대로 남아있음
- 복사된 커밋은 **새로운 해시**를 가짐 (부모 커밋이 다르니까)

## 기본 사용법

```bash
# 특정 커밋 하나 가져오기
git cherry-pick <커밋해시>

# 여러 커밋 나열
git cherry-pick <해시1> <해시2> <해시3>

# 범위 지정 (D는 미포함, E~F 포함)
git cherry-pick D..F
```

## 유용한 옵션

```bash
# 커밋하지 않고 스테이징에만 올리기 (여러 커밋을 하나로 합칠 때)
git cherry-pick --no-commit <해시>  # (-n)

# 충돌 발생 시 취소
git cherry-pick --abort

# 충돌 해결 후 계속 진행 (해시 붙이지 말 것!)
git cherry-pick --continue
```

## 충돌 해결 흐름

1. cherry-pick 시도 → 충돌 발생
2. 충돌 파일 열어서 수동으로 해결
3. `git add <파일>`
4. `git cherry-pick --continue` (해시 없이!)

취소하고 싶으면: `git cherry-pick --abort`

## cherry-pick vs merge vs rebase

| 명령어 | 언제 쓸까 |
|--------|----------|
| cherry-pick | 특정 커밋 1~2개만 골라서 가져올 때 |
| merge | 브랜치 작업이 끝나서 전체를 합칠 때 |
| rebase | 푸시 전 커밋 히스토리를 깔끔하게 정리할 때 |

## 실무 활용 상황

1. feature 브랜치의 버그 수정 커밋만 급하게 main에 반영
2. 잘못된 브랜치에서 한 커밋을 올바른 브랜치로 옮기기
3. 다른 팀원 브랜치에서 유용한 커밋 하나만 가져오기

---

## 주의사항 & 흔한 실수

### 1. `--continue` 뒤에 해시를 붙이지 마라!

충돌 해결 후 진행할 때 실수하기 쉬운 부분:

```bash
# 틀린 예 (에러 발생!)
git cherry-pick --continue abc1234

# 맞는 예
git cherry-pick --continue
```

`--continue`는 "이미 진행 중인 cherry-pick을 이어서 해라"는 뜻이다.
어떤 커밋을 cherry-pick 중인지 Git이 이미 알고 있기 때문에 해시를 다시 줄 필요가 없다.

> `--abort`도 마찬가지. 뒤에 해시 붙이지 않는다.

### 2. `--no-commit` 용도를 정확히 이해하라

`--no-commit` (-n)은 **커밋을 만들지 않고 변경사항을 스테이징 영역에만 올린다**.

```bash
# 일반 cherry-pick: 가져오면서 바로 커밋됨
git cherry-pick abc1234
# → 새 커밋이 자동 생성됨

# --no-commit: 스테이징에만 올림, 커밋은 내가 직접
git cherry-pick --no-commit abc1234
# → 변경사항이 staged 상태로 대기 중
# → git commit 으로 직접 커밋해야 함
```

**언제 쓸까?**
- 여러 cherry-pick을 **하나의 커밋으로 합치고** 싶을 때
- cherry-pick한 내용을 **커밋 전에 수정**하고 싶을 때

```bash
# 예: 3개 커밋을 가져와서 1개로 합치기
git cherry-pick --no-commit aaa1111
git cherry-pick --no-commit bbb2222
git cherry-pick --no-commit ccc3333
git commit -m "feat: 3개 기능 한번에 반영"
```

---

## 실습 가이드 (처음부터 끝까지 따라하기)

### 실습 1: 기본 cherry-pick

```bash
# === 준비 ===
mkdir cherry-pick-practice && cd cherry-pick-practice
git init

# === main에 초기 커밋 ===
echo "v1" > main.txt
git add main.txt
git commit -m "init: v1"

# === feature 브랜치에서 여러 커밋 만들기 ===
git switch -c feature
echo "기능 A" > a.txt
git add a.txt
git commit -m "feat: 기능 A"

echo "버그 수정" > bugfix.txt
git add bugfix.txt
git commit -m "fix: 긴급 버그 수정"    # ← 이것만 main에 가져올 것

echo "기능 B" > b.txt
git add b.txt
git commit -m "feat: 기능 B"

# feature 커밋 해시 확인
git log --oneline
# 출력 예:
# ccc3333 feat: 기능 B
# bbb2222 fix: 긴급 버그 수정    ← 이 해시를 복사!
# aaa1111 feat: 기능 A
# 0001111 init: v1

# === main으로 이동해서 cherry-pick ===
git switch main
git cherry-pick <위에서-복사한-bugfix-해시>    # bbb2222 자리에 실제 해시

# 결과 확인
git log --oneline
# 출력 예:
# ddd4444 fix: 긴급 버그 수정    ← 새 해시로 복사됨!
# 0001111 init: v1

ls    # bugfix.txt가 main에 생겼다!

# === 정리 ===
cd ..
rm -rf cherry-pick-practice
```

### 실습 2: --no-commit으로 여러 커밋 합치기

```bash
# === 준비 ===
mkdir cherry-pick-practice2 && cd cherry-pick-practice2
git init

# === 초기 커밋 ===
echo "base" > base.txt
git add base.txt
git commit -m "init: base"

# === feature에서 커밋 3개 만들기 ===
git switch -c feature
echo "수정1" > fix1.txt && git add fix1.txt && git commit -m "fix: 수정 1"
echo "수정2" > fix2.txt && git add fix2.txt && git commit -m "fix: 수정 2"
echo "수정3" > fix3.txt && git add fix3.txt && git commit -m "fix: 수정 3"

# 해시 확인
git log --oneline
# 3개 해시를 메모해둔다

# === main에서 --no-commit으로 합치기 ===
git switch main

# 3개를 가져오되 커밋하지 않음 (해시를 실제 값으로 교체)
git cherry-pick --no-commit <수정1-해시>
git cherry-pick --no-commit <수정2-해시>
git cherry-pick --no-commit <수정3-해시>

# 스테이징 상태 확인 - 3개 파일이 staged 상태
git status

# 하나의 커밋으로 합침
git commit -m "fix: 수정 1~3 일괄 반영"

# 결과 확인 - 커밋 1개에 파일 3개
git log --oneline
ls    # fix1.txt, fix2.txt, fix3.txt 모두 있음

# === 정리 ===
cd ..
rm -rf cherry-pick-practice2
```

### 각 단계에서 확인할 것

| 단계 | 확인 명령어 | 기대 결과 |
|------|------------|----------|
| cherry-pick 후 | `git log --oneline` | 새 해시로 커밋이 복사됨 |
| cherry-pick 후 | `ls` | 해당 파일이 현재 브랜치에 생김 |
| --no-commit 후 | `git status` | 파일이 staged 상태 (커밋 안 됨) |
| feature 확인 | `git switch feature && git log --oneline` | 원본 커밋 그대로 남아있음 |
