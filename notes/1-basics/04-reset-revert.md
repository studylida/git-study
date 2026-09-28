# 되돌리기: reset vs revert

## reset vs revert 비교

```
reset:  A - B - C  →  A - B      (C 삭제, 히스토리 변경)
revert: A - B - C  →  A - B - C - C'  (C를 취소하는 새 커밋 추가)
```

| | reset | revert |
|--|-------|--------|
| 방식 | 커밋 삭제 | 취소 커밋 생성 |
| 히스토리 | 변경됨 | 유지됨 |
| push한 커밋 | ❌ 위험 | ✅ 안전 |

---

## git reset

### 3가지 모드

```
           --soft     --mixed(기본)    --hard
              ↓            ↓              ↓
HEAD        되돌림       되돌림         되돌림
Staging     유지         되돌림         되돌림
Working     유지         유지           되돌림
```

### 사용법

```bash
git reset HEAD 파일       # unstage (staging → working)
git reset --soft HEAD~1   # 커밋 취소, 변경사항 staged 유지
git reset HEAD~1          # 커밋 취소, 변경사항 unstaged (기본)
git reset --hard HEAD~1   # 모두 삭제 ⚠️
git reset --hard HEAD     # 모든 로컬 변경사항 삭제 ⚠️
```

### HEAD~ 표기법
- `HEAD~1` = 1개 전 커밋
- `HEAD~2` = 2개 전 커밋
- `HEAD~` = `HEAD~1`과 동일

---

## git revert

안전하게 되돌리기 (새 커밋 생성)

```bash
git revert HEAD              # 최신 커밋 취소
git revert 커밋해시           # 특정 커밋 취소
git revert HEAD~2            # 2개 전 커밋 취소
git revert --no-edit HEAD    # 편집기 없이 기본 메시지 사용
```

### 여러 커밋 revert

```bash
git revert HEAD~2..HEAD      # 최근 2개 커밋 순차적으로 revert
git revert -n HEAD~2..HEAD   # 커밋 없이 변경만 (-n = --no-commit)
```

---

## 언제 무엇을 사용?

| 상황 | 사용할 명령어 |
|------|-------------|
| 아직 push 안 한 커밋 | `reset` 또는 `revert` |
| 이미 push한 커밋 | `revert` ✅ |
| 로컬 변경사항 버리기 | `reset --hard` |
| 협업 중 | `revert` ✅ |

### 핵심 원칙
> **push한 커밋은 revert, 안 한 커밋은 reset**

---

## 주의사항 & 흔한 실수

### `reset --mixed`가 뭘 되돌리는지 혼동

```
           --soft     --mixed(기본)    --hard
HEAD        되돌림       되돌림         되돌림
Staging     유지         되돌림         되돌림
Working     유지         유지           되돌림
```

**핵심 정리**:
- `--soft`: 커밋만 취소. 변경사항은 **staged 상태**로 남음 → 바로 다시 커밋 가능
- `--mixed` (기본): 커밋 + staging 취소. 변경사항은 **unstaged 상태**로 남음 → 다시 add 필요
- `--hard`: 전부 삭제. 변경사항 **완전히 사라짐** → 복구 어려움!

**외우는 팁**: soft(부드럽게) → mixed(중간) → hard(강하게) 순서로 점점 더 많이 되돌린다. mixed는 "Staging까지" 되돌리고 Working Directory는 건드리지 않는다.

```bash
# 실수 예시: mixed를 썼는데 staged 상태가 유지될 거라고 착각
git reset HEAD~1            # --mixed가 기본! staging도 풀림
git status                  # 변경사항이 unstaged 상태
git add .                   # 다시 add 해야 함
git commit -m "메시지"

# 원래 의도가 "커밋만 취소하고 바로 다시 커밋"이었다면:
git reset --soft HEAD~1     # 이걸 써야 함!
git commit -m "새 메시지"   # 바로 커밋 가능
```

---

## 실습 가이드: 처음부터 따라하기

```bash
# 1. 실습용 폴더 만들기
mkdir git-reset-practice && cd git-reset-practice
git init

# 2. 커밋 3개 만들기
echo "v1" > file.txt && git add . && git commit -m "커밋 1: v1"
echo "v2" > file.txt && git add . && git commit -m "커밋 2: v2"
echo "v3" > file.txt && git add . && git commit -m "커밋 3: v3"

# 3. 현재 상태 확인
git log --oneline
# 결과: abc1234 커밋 3: v3
#        def5678 커밋 2: v2
#        ghi9012 커밋 1: v1

# === reset --soft 실습 ===
# 4. soft reset (커밋만 취소, staging 유지)
git reset --soft HEAD~1
git log --oneline           # 커밋 3이 사라짐
git status -s               # M  file.txt (staged 상태!)
cat file.txt                # v3 (파일 내용 그대로)

# 5. 다시 커밋해서 원복
git commit -m "커밋 3: v3"

# === reset --mixed 실습 ===
# 6. mixed reset (커밋 + staging 취소)
git reset HEAD~1
git log --oneline           # 커밋 3이 사라짐
git status -s               #  M file.txt (unstaged 상태!)
cat file.txt                # v3 (파일 내용 그대로)

# 7. 다시 add + 커밋
git add . && git commit -m "커밋 3: v3"

# === reset --hard 실습 ===
# 8. hard reset (전부 삭제! ⚠️)
git reset --hard HEAD~1
git log --oneline           # 커밋 3이 사라짐
git status -s               # (깨끗함 - 변경사항 없음)
cat file.txt                # v2 (파일 내용도 되돌아감!)

# === revert 실습 ===
# 9. revert로 안전하게 되돌리기
git revert --no-edit HEAD
git log --oneline
# 결과: xyz7890 Revert "커밋 2: v2"   ← 취소 커밋 추가됨
#        def5678 커밋 2: v2             ← 원본 유지
#        ghi9012 커밋 1: v1
cat file.txt                # v1 (커밋 2의 변경이 취소됨)

# 실습 끝! 정리하려면:
cd .. && rm -rf git-reset-practice
```
