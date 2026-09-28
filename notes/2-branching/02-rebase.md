# Rebase - 커밋 히스토리 정리

## rebase란?

브랜치의 커밋들을 다른 브랜치 위로 "옮겨 심는" 것.
merge와 달리 히스토리가 일직선으로 깔끔해진다.

## merge vs rebase 비교

```
# merge 결과 (합치는 커밋이 생김)
      A---B---C  feature
     /         \
D---E---F---G---H  main (merge commit)

# rebase 결과 (일직선)
D---E---F---G---A'---B'---C'  main
```

## 기본 사용법

```bash
git switch feature-branch    # feature 브랜치로 이동
git rebase main              # main 위로 rebase
```

## 핵심 특징

1. **커밋 해시가 바뀐다** - rebase는 커밋을 "다시 만드는" 것
2. **히스토리가 깔끔해진다** - 일직선으로 정리됨
3. **충돌 시 각 커밋마다 해결해야 함** - merge는 한 번만

---

## 주의사항 (중요!)

### rebase는 위험할 수 있다

| 상황 | 위험도 | 이유 |
|------|--------|------|
| 아직 push 안 한 로컬 커밋 | 안전 | 나만 알고 있음 |
| 혼자 쓰는 feature 브랜치 | 안전 | 다른 사람에게 영향 없음 |
| **이미 push한 브랜치** | **위험** | 다른 사람 작업과 충돌 |
| **main, develop 등 공용 브랜치** | **매우 위험** | 모든 팀원에게 영향 |

### 왜 위험한가?

```
1. 나: feature 브랜치에서 커밋 A, B 만들고 push
2. 동료: 내 브랜치 pull 받아서 작업 중
3. 나: rebase 함 → 커밋이 A', B'로 해시가 바뀜
4. 나: force push (rebase 후엔 force push 필요)
5. 동료: pull 하려니 히스토리가 달라서 충돌!
```

### 황금률

> **"이미 공유한 커밋은 rebase하지 마라"**

특히 main 브랜치와 관련된 rebase는 조심해야 한다:
- `git rebase main` (feature에서) → OK
- `git rebase feature` (main에서) → 위험!

---

## 언제 rebase를 쓸까?

### rebase가 적합한 경우
- 아직 push 안 한 로컬 커밋 정리
- 혼자 작업하는 feature 브랜치를 최신 main에 맞출 때
- PR 전에 커밋 히스토리 깔끔하게 정리할 때

### merge가 적합한 경우
- 이미 push한 브랜치
- 여러 명이 함께 쓰는 브랜치
- 안전하게 작업하고 싶을 때

---

## 실무 팁

- 팀마다 rebase 정책이 다름 (금지하는 팀도 많음)
- 처음엔 **merge만 쓰는 게 안전**
- rebase는 히스토리 관리가 중요할 때 조심해서 사용
- 확신이 없으면 merge 사용

---

## Interactive Rebase (git rebase -i)

커밋 히스토리를 세밀하게 편집할 수 있는 기능.

```bash
git rebase -i HEAD~3    # 최근 3개 커밋 편집
```

실행하면 에디터가 열리고 이런 화면이 나온다:

```
pick abc1234 첫 번째 커밋
pick def5678 두 번째 커밋
pick ghi9012 세 번째 커밋
```

### 사용 가능한 명령어

| 명령어 | 줄임 | 설명 |
|--------|------|------|
| pick | p | 커밋을 그대로 사용 |
| squash | s | 이전 커밋과 합침 (메시지도 합침) |
| fixup | f | 이전 커밋과 합침 (메시지는 버림) |
| reword | r | 커밋 메시지만 수정 |
| drop | d | 커밋 삭제 |
| edit | e | 커밋 내용 수정 |

---

## 실습 예시

### 예시 1: squash - 여러 커밋 합치기

지저분한 커밋들을 하나로 정리할 때 사용.

```
Before:
  커밋3: 오타 수정하고 비밀번호 확인 넣음
  커밋2: 이메일 검증 추가함
  커밋1: 회원가입 기능 추가

After:
  커밋1: 회원가입 기능 추가 (3개가 1개로!)
```

**방법:**
```bash
git rebase -i HEAD~3
```

에디터에서:
```
pick f5dd6fb 회원가입 기능 추가
s 1e2964a 이메일 검증 추가함       # pick → s 로 변경
s 9d35730 오타 수정하고 비밀번호   # pick → s 로 변경
```

저장하면 커밋 메시지 편집 화면이 나오고, 정리해서 저장하면 완료.

---

### 예시 2: rebase 중 충돌 해결

같은 파일을 다르게 수정한 경우 충돌 발생.

```
main:    "인사말: 환영합니다"
feature: "인사말: 반갑습니다"
         ↓ rebase 시도
       충돌 발생!
```

**충돌 파일 모습:**
```
<<<<<<< HEAD
인사말: 환영합니다        ← main의 내용
=======
인사말: 반갑습니다        ← 내 브랜치 내용
>>>>>>> 7cd44a5
```

**해결 방법:**
1. 파일 열어서 원하는 내용만 남기고 마커(<<<, ===, >>>) 삭제
2. `git add <파일>`
3. `git rebase --continue`

**유용한 명령어:**
| 명령어 | 설명 |
|--------|------|
| `git rebase --continue` | 충돌 해결 후 계속 진행 |
| `git rebase --abort` | 포기하고 rebase 이전으로 돌아가기 |
| `git rebase --skip` | 현재 커밋 건너뛰기 |

---

### 예시 3: reword - 커밋 메시지 수정

오타가 있는 커밋 메시지를 수정할 때 사용.

```
Before:
  40cedc8 결게 기능 추가    ← 오타!

After:
  dd2a25c 결제 기능 추가    ← 수정됨
```

**방법:**
```bash
git rebase -i HEAD~3
```

에디터에서:
```
pick 1487cc3 장바구니 기능 추가
r 40cedc8 결게 기능 추가       # pick → r (reword)
pick 0e3bbaf 주문 내역 기능 추가
```

저장하면 커밋 메시지 편집 화면이 나오고, "결제 기능 추가"로 수정하면 완료.

---

### 예시 4: drop - 커밋 삭제

실수로 만든 커밋을 삭제할 때 사용.

```
Before:
  0e3bbaf 주문 내역 기능 추가
  6ab312e 테스트용 (나중에 지울 것)   ← 삭제할 커밋
  40cedc8 결제 기능 추가
  1487cc3 장바구니 기능 추가

After:
  0d87ff1 주문 내역 기능 추가
  dd2a25c 결제 기능 추가
  1487cc3 장바구니 기능 추가
  (테스트용 커밋 사라짐!)
```

**방법:**
```bash
git rebase -i HEAD~4
```

에디터에서:
```
pick 1487cc3 장바구니 기능 추가
pick 40cedc8 결제 기능 추가
d 6ab312e 테스트용 (나중에 지울 것)  # pick → d (drop)
pick 0e3bbaf 주문 내역 기능 추가
```

저장하면 해당 커밋이 삭제됨. (파일도 함께 사라짐!)

---

## Vim 에디터 기본 단축키

`git rebase -i` 실행 시 vim이 열리면:

| 키 | 설명 |
|----|------|
| `ESC` | Normal 모드로 전환 |
| `i` | Insert 모드 (입력) |
| `cw` | 단어 지우고 입력 모드 |
| `dd` | 줄 삭제 |
| `j` / `k` | 아래 / 위 이동 |
| `:wq` | 저장하고 종료 |
| `:q!` | 저장 안 하고 종료 |

**팁:** VS Code를 기본 에디터로 설정하면 더 편함
```bash
git config --global core.editor "code --wait"
```

---

## 주의사항 & 흔한 실수 (Interactive Rebase 편)

### 1. reword와 squash를 혼동하지 마라!

이 둘은 완전히 다른 동작이다:

| 명령어 | 하는 일 | 커밋 수 변화 |
|--------|---------|-------------|
| `reword` (r) | 커밋 메시지**만** 수정. 코드 변경 없음 | 그대로 유지 |
| `squash` (s) | **이전 커밋과 합침**. 코드 + 메시지 모두 합쳐짐 | 줄어듦 |

```
# reword: 커밋 개수는 그대로, 메시지만 바뀜
Before: A ─ B("오탈") ─ C
After:  A ─ B'("수정됨") ─ C'

# squash: 커밋이 합쳐져서 개수가 줄어듦
Before: A ─ B ─ C
After:  A ─ BC (B와 C가 하나로)
```

**기억법:** reword = re + word (단어를 다시 쓴다) = 메시지만 수정

### 2. reword는 커밋 내용(코드)을 수정하지 않는다

reword를 선택하면 에디터가 열리면서 **커밋 메시지 편집 화면만** 나온다.
코드를 수정하고 싶으면 `edit` (e)을 사용해야 한다.

---

## 실습 가이드 (처음부터 끝까지 따라하기)

### 실습 1: squash로 커밋 합치기

```bash
# === 준비 ===
mkdir rebase-practice && cd rebase-practice
git init

# === 커밋 3개 만들기 ===
echo "기능 A" > feature.txt
git add feature.txt
git commit -m "feat: 기능 A 추가"

echo "기능 A 수정" >> feature.txt
git add feature.txt
git commit -m "fix: 기능 A 오타 수정"

echo "기능 A 완성" >> feature.txt
git add feature.txt
git commit -m "fix: 기능 A 마무리"

# 현재 로그 확인 (3개 커밋)
git log --oneline

# === squash 실행 ===
# 아래 명령어 대신 non-interactive 방식으로 체험
# (실제로는 git rebase -i HEAD~3 을 쓰지만, 여기선 결과만 확인)
git reset --soft HEAD~2           # 최근 2개 커밋을 스테이징으로 되돌림
git commit --amend -m "feat: 기능 A 추가 (오타 수정 및 마무리 포함)"

# 결과 확인 - 3개가 1개로 합쳐짐!
git log --oneline

# === 정리 ===
cd ..
rm -rf rebase-practice
```

### 실습 2: 기본 rebase 체험 (feature를 main 위로)

```bash
# === 준비 ===
mkdir rebase-practice2 && cd rebase-practice2
git init

# === main에 초기 커밋 ===
echo "v1" > main.txt
git add main.txt
git commit -m "init: v1"

# === feature 브랜치 생성 + 작업 ===
git switch -c feature
echo "feature 작업" > feature.txt
git add feature.txt
git commit -m "feat: feature 작업"

# === main에서도 새 커밋 추가 (분기 발생) ===
git switch main
echo "v2" >> main.txt
git add main.txt
git commit -m "update: v2"

# 현재 상태 확인
git log --oneline --all --graph
# 출력 예:
# * (main)  update: v2
# | * (feature)  feat: feature 작업
# |/
# * init: v1

# === feature에서 rebase 실행 ===
git switch feature
git rebase main        # feature를 main 위로 옮겨 심기

# 결과 확인 - 일직선이 됨!
git log --oneline --all --graph
# 출력 예:
# * (feature) feat: feature 작업
# * (main) update: v2
# * init: v1

# === 정리 ===
cd ..
rm -rf rebase-practice2
```

### 각 단계에서 확인할 것

| 단계 | 확인 명령어 | 기대 결과 |
|------|------------|----------|
| 분기 후 (rebase 전) | `git log --oneline --all --graph` | Y자 형태 (갈라짐) |
| rebase 후 | `git log --oneline --all --graph` | 일직선 형태 |
| squash 후 | `git log --oneline` | 커밋 수가 줄어듦 |
