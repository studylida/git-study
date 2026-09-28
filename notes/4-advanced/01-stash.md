# stash - 작업 임시 저장

## 핵심 개념

### stash란?
- 커밋하기엔 애매한 작업 중인 변경사항을 **임시 보관**하는 기능
- stash하면 워킹 디렉토리가 깨끗해짐 → 브랜치 전환 가능
- **브랜치에 종속되지 않음** — 어디서든 꺼낼 수 있음

### 기본 흐름

```
작업 중 (변경사항 있음)
    │
    ▼
git stash          ← 서랍에 넣기
    │
    ▼
다른 작업 수행      ← 브랜치 전환, 버그 수정 등
    │
    ▼
git stash pop      ← 서랍에서 꺼내기
```

## 명령어 정리

| 명령어 | 설명 |
|--------|------|
| `git stash` | 변경사항 임시 저장 (tracked 파일만) |
| `git stash -u` | untracked 파일도 포함해서 stash |
| `git stash -m "메시지"` | 설명 달아서 저장 |
| `git stash pop` | 가장 최근 stash 꺼내고 **목록에서 삭제** |
| `git stash apply` | 가장 최근 stash 꺼내고 **목록에 유지** |
| `git stash list` | 저장된 stash 목록 보기 |
| `git stash drop stash@{n}` | 특정 stash 삭제 |
| `git stash clear` | 모든 stash 삭제 |

## pop vs apply

```
pop   = apply + drop  (꺼내고 삭제)
apply = 꺼내기만      (목록에 남아있음)
```

- **pop**: 대부분의 경우 (한 번 쓰고 버릴 때)
- **apply**: 같은 stash를 여러 브랜치에 적용하고 싶을 때

## stash 번호 (stash@{n})

- `stash@{0}`: 가장 최근 stash
- `stash@{1}`: 그 다음
- 숫자가 클수록 오래된 것

```bash
git stash apply stash@{1}   # 두 번째 stash 적용
git stash drop stash@{2}    # 세 번째 stash 삭제
```

## 주의사항

- 기본 `git stash`는 **untracked 파일 무시** → `-u` 옵션 필요
- 같은 파일이 이미 수정된 상태에서 stash apply하면 **충돌** 발생 가능
- 여러 개 쌓이면 구분이 안 되니 **`-m` 메시지 습관** 들이기

---

## 실습 가이드: stash 상황별 연습

### 준비

```bash
mkdir ~/git-stash-practice && cd ~/git-stash-practice
git init

# 초기 파일 생성
echo "메인 코드" > main.py
echo "설정 파일" > config.txt
git add . && git commit -m "init"
```

### Step 1: 기본 stash & pop

```bash
# 작업 중인 변경사항 만들기
echo "새 기능 코드 작업 중..." >> main.py

# 급하게 다른 일을 해야 할 때!
git stash -m "새 기능 작업 중"
# → 워킹 디렉토리가 깨끗해짐

cat main.py
# → "메인 코드"만 있음 (변경사항이 서랍에 들어감)

# 다른 급한 작업 완료 후...
git stash pop
# → 변경사항이 다시 돌아옴!

cat main.py
# → "새 기능 코드 작업 중..."이 다시 보임
```

### Step 2: 여러 개 stash 관리

```bash
# stash 1
echo "기능 A" >> main.py
git stash -m "기능 A 작업 중"

# stash 2
echo "기능 B" >> config.txt
git stash -m "기능 B 작업 중"

# 목록 확인
git stash list
# → stash@{0}: On main: 기능 B 작업 중
# → stash@{1}: On main: 기능 A 작업 중
# (최신이 0번!)

# 특정 stash 꺼내기
git stash apply stash@{1}
# → 기능 A가 돌아옴 (목록에는 남아있음)

# 필요 없는 stash 삭제
git stash drop stash@{0}
```

### Step 3: untracked 파일도 stash

```bash
# 새 파일 생성 (untracked)
echo "새 파일" > new-feature.py

git stash
# → new-feature.py는 stash 안 됨! (untracked라서)

git stash -u
# → 이제 new-feature.py도 포함해서 stash됨

ls
# → new-feature.py가 사라짐

git stash pop
# → 다시 돌아옴
```

### Step 4: stash 충돌 체험

```bash
# 변경사항 stash
echo "버전 A" >> main.py
git stash -m "버전 A"

# 같은 파일을 다르게 수정 후 커밋
echo "버전 B" >> main.py
git add . && git commit -m "버전 B 적용"

# stash pop하면 충돌!
git stash pop
# → CONFLICT! 수동으로 해결 필요
# → 충돌 해결 후 git add & git commit

# 충돌 시 stash는 자동 삭제 안 됨 → 수동 삭제 필요
git stash drop
```

### 정리

```bash
rm -rf ~/git-stash-practice
```

---

## 주의사항 & 흔한 실수 (보충)

- stash pop 중 충돌이 나면 **stash가 자동 삭제되지 않음** → 충돌 해결 후 `git stash drop`으로 수동 삭제
- `git stash`는 **staged(add한 것)와 unstaged 변경사항 모두** stash함. 되돌릴 때는 전부 unstaged 상태로 돌아옴
- staged 상태를 유지하고 싶으면 `git stash pop --index` 사용
- stash는 **브랜치에 종속되지 않음**: A 브랜치에서 stash하고 B 브랜치에서 pop 가능 (단, 충돌 주의)
