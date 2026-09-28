# git diff - 변경사항 비교

## 개념

```
Working Directory ←──diff──→ Staging Area ←──diff --staged──→ Repository
                     ↑                                            ↑
                     └────────────diff HEAD────────────────────────┘
```

---

## 기본 명령어

```bash
git diff                  # Working Directory ↔ Staging Area
git diff --staged         # Staging Area ↔ 최신 커밋 (= --cached)
git diff HEAD             # Working Directory ↔ 최신 커밋
```

---

## 비교 대상 지정

```bash
git diff 커밋해시              # 특정 커밋과 비교
git diff 커밋1 커밋2           # 두 커밋 간 비교
git diff 브랜치1 브랜치2       # 두 브랜치 간 비교
git diff HEAD~2              # 2개 전 커밋과 비교
```

---

## 유용한 옵션

```bash
git diff 파일명              # 특정 파일만
git diff --stat             # 변경 통계만 (몇 줄 추가/삭제)
git diff --name-only        # 변경된 파일 이름만
git diff --word-diff        # 단어 단위로 비교
```

---

## 출력 읽는 법

```diff
--- a/hello.txt           ← 변경 전 파일
+++ b/hello.txt           ← 변경 후 파일
@@ -1,3 +1,4 @@           ← 위치 정보
 안녕하세요!               ← 공백: 변경 없음
+새로 추가된 줄            ← +: 추가된 줄
-삭제된 줄                 ← -: 삭제된 줄
```

---

## 핵심 포인트

| 상태 | `git diff` | `git diff --staged` |
|------|------------|---------------------|
| 수정만 함 | 변경사항 보임 | 비어있음 |
| add 후 | 비어있음 | 변경사항 보임 |

---

## 주의사항 & 흔한 실수

### `git add` 후 `git diff`가 비어있다?!

```bash
echo "새 내용" >> file.txt
git diff                    # ✅ 변경사항 보임

git add file.txt
git diff                    # ❌ 아무것도 안 나옴! (당황 포인트)
git diff --staged           # ✅ 이걸 써야 보임!
```

**원리**: `git diff`는 Working Directory와 Staging Area를 비교하는데, `git add` 하면 둘이 같아지니까 차이가 없음. Staging Area와 최신 커밋을 비교하려면 반드시 `--staged` (또는 `--cached`) 옵션을 붙여야 함.

**외우는 팁**: "add 했으면 --staged 붙이기"

---

## 실습 가이드: 처음부터 따라하기

```bash
# 1. 실습용 폴더 만들기
mkdir git-diff-practice && cd git-diff-practice
git init

# 2. 초기 파일 만들고 커밋
echo "line 1" > test.txt
git add test.txt
git commit -m "init: 첫 커밋"

# 3. 파일 수정
echo "line 2" >> test.txt

# 4. git diff 확인 (Working Directory ↔ Staging Area)
git diff
# 결과:
# --- a/test.txt
# +++ b/test.txt
# @@ -1 +1,2 @@
#  line 1
# +line 2              ← 추가된 줄

# 5. add 후 diff가 비어있는 것 확인
git add test.txt
git diff
# 결과: (아무것도 안 나옴!)

# 6. --staged로 확인
git diff --staged
# 결과:
# --- a/test.txt
# +++ b/test.txt
# @@ -1 +1,2 @@
#  line 1
# +line 2              ← Staging에 있는 변경사항

# 7. HEAD와 비교 (Working + Staging 모두 포함)
git diff HEAD
# 결과: 위와 동일 (커밋 이후 모든 변경사항)

# 8. 커밋 후 두 커밋 간 비교
git commit -m "feat: line 2 추가"
echo "line 3" >> test.txt
git add test.txt
git commit -m "feat: line 3 추가"

git diff HEAD~1             # 직전 커밋과 비교
git diff HEAD~2             # 2개 전 커밋과 비교
git diff --stat HEAD~2      # 통계만 보기

# 실습 끝! 정리하려면:
cd .. && rm -rf git-diff-practice
```
