# reflog - 실수로 삭제한 커밋 복구

## reflog란?

- **Reference Log**의 줄임말
- HEAD가 가리킨 모든 이동 기록을 저장한 로그
- Git의 "블랙박스" — 내가 한 모든 행동이 기록됨

## git log vs git reflog

| 구분 | git log | git reflog |
|------|---------|------------|
| 보여주는 것 | 커밋 간 부모-자식 관계 | HEAD 이동 기록 (모든 행동) |
| reset으로 날린 커밋 | 안 보임 | 보임 (해시 찾을 수 있음) |
| 용도 | 커밋 히스토리 확인 | 실수 복구 |

## reflog 특징

1. **로컬 전용** — 원격에는 없고 내 컴퓨터에만 있음
2. **90일 보관** — 기본 90일 후 자동 삭제
3. **clone하면 없음** — 새로 clone한 저장소에는 reflog가 비어있음

## 핵심 명령어

```bash
# reflog 보기
git reflog                  # HEAD의 이동 기록
git reflog show <브랜치>     # 특정 브랜치의 이동 기록

# 복구하기
git reset --hard <해시>      # 커밋 복구 (현재 브랜치를 그 커밋으로 이동)
git branch <이름> <해시>     # 삭제된 브랜치 복구 (해시 위치에 새 브랜치 생성)
```

## reflog 출력 읽는 법

```
abc1234 HEAD@{0}: commit: 최신 커밋 메시지
───────  ────────  ─────────────────────────
 해시     몇 번째     무슨 행동을 했는지
(복구용)  전 행동
```

## 복구 상황별 방법

| 상황 | 방법 |
|------|------|
| reset --hard로 커밋 날림 | reflog에서 해시 찾고 `git reset --hard <해시>` |
| 브랜치 삭제 | reflog에서 해시 찾고 `git branch <이름> <해시>` |
| rebase 실수 | reflog에서 rebase 전 해시 찾고 `git reset --hard <해시>` |

## reset vs branch 차이 (복구 시)

- `git reset --hard <해시>` → **현재 브랜치**를 그 커밋으로 옮김
- `git branch <이름> <해시>` → 그 커밋 위치에 **새 브랜치를 생성** (현재 브랜치는 안 움직임)

---

## 실습 가이드: reflog로 실수 복구하기

### 준비

```bash
mkdir ~/git-reflog-practice && cd ~/git-reflog-practice
git init

# 커밋 3개 만들기
echo "첫 번째" > file.txt
git add . && git commit -m "커밋 1: 첫 번째"

echo "두 번째" >> file.txt
git add . && git commit -m "커밋 2: 두 번째"

echo "세 번째" >> file.txt
git add . && git commit -m "커밋 3: 세 번째"

# 현재 상태 확인
git log --oneline
# → abc1234 커밋 3: 세 번째
# → def5678 커밋 2: 두 번째
# → ghi9012 커밋 1: 첫 번째
```

### Step 1: reset --hard로 커밋 날리고 reflog로 복구

```bash
# "실수로" 커밋 2개를 날려버림!
git reset --hard HEAD~2
# → 커밋 1만 남음

git log --oneline
# → ghi9012 커밋 1: 첫 번째  (커밋 2, 3이 사라짐!)

cat file.txt
# → "첫 번째"만 남아있음

# git log에서는 안 보이지만... reflog에는 있다!
git reflog
# → ghi9012 HEAD@{0}: reset: moving to HEAD~2
# → abc1234 HEAD@{1}: commit: 커밋 3: 세 번째  ← 여기!
# → def5678 HEAD@{2}: commit: 커밋 2: 두 번째
# → ghi9012 HEAD@{3}: commit (initial): 커밋 1: 첫 번째

# 커밋 3의 해시로 복구!
git reset --hard abc1234    # reflog에서 찾은 해시 사용

git log --oneline
# → abc1234 커밋 3: 세 번째  ← 돌아왔다!
# → def5678 커밋 2: 두 번째
# → ghi9012 커밋 1: 첫 번째

cat file.txt
# → 세 줄 모두 복구됨!
```

### Step 2: 삭제된 브랜치 복구

```bash
# 브랜치 생성하고 커밋
git switch -c feature/important
echo "중요한 기능" > important.py
git add . && git commit -m "feat: 중요한 기능 추가"

# main으로 돌아가서 브랜치 삭제
git switch main
git branch -D feature/important
# → Deleted branch feature/important (was xyz7890)

# reflog에서 해시 찾기
git reflog
# → xyz7890 HEAD@{1}: commit: feat: 중요한 기능 추가  ← 여기!

# 새 브랜치로 복구 (현재 브랜치는 안 움직임)
git branch feature/recovered xyz7890

git log --oneline feature/recovered
# → 중요한 기능 커밋이 살아있다!
```

### Step 3: HEAD@{n} 문법 사용

```bash
# reflog 항목 번호로도 복구 가능
git reflog
# → HEAD@{0}, HEAD@{1}, HEAD@{2}...

# "3번 전 행동 시점"으로 되돌리기
git reset --hard HEAD@{3}
# → 해시를 외울 필요 없이 번호로 이동!
```

### 정리

```bash
rm -rf ~/git-reflog-practice
```

---

## 주의사항 & 흔한 실수

- reflog는 **로컬 전용**! 다른 컴퓨터에서 clone하면 reflog가 비어있음. 복구 불가.
- 기본 보관 기간은 **90일**. 90일 넘은 기록은 `git gc`가 정리함
- `git reset --hard`로 복구할 때 **현재 작업 중인 변경사항은 날아감** → 먼저 stash하거나 commit
- reflog 해시 대신 `HEAD@{n}` 문법을 쓸 때, **번호는 바뀔 수 있음** (새 행동을 하면 밀림). 확실한 건 해시 사용
- `git reflog expire --expire=now --all && git gc --prune=now` 를 실행하면 reflog가 완전히 삭제됨 — 복구 불가능해지므로 **절대 함부로 실행하지 말 것**
