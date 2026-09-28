# 원격 추적 브랜치 (Remote Tracking Branch)

## 핵심 개념

### 원격 추적 브랜치란?
- `origin/main` 같은 브랜치
- "마지막으로 확인한 원격 저장소의 상태"를 기억하는 포인터
- 로컬에 존재하지만 직접 수정 불가 (읽기 전용)
- fetch나 push할 때 자동으로 업데이트됨

### 왜 필요한가?
- 원격과 로컬의 차이를 비교할 수 있음
- 네트워크 없이도 "마지막으로 확인한 원격 상태"를 알 수 있음
- `git log origin/main..main`으로 아직 push 안 한 커밋 확인 가능

## 주요 명령어

### 브랜치 확인
- `git branch -r`: 원격 추적 브랜치만 보기 (origin/main 등)
- `git branch -a`: 로컬 + 원격 추적 브랜치 모두 보기
- `git branch -vv`: 로컬 브랜치의 upstream 연결 상태 확인

### upstream 설정
- `git push -u origin main`: push하면서 upstream 설정
- `git branch -u origin/main`: 현재 브랜치의 upstream을 origin/main으로 설정
- upstream이 설정되면 `git push`, `git pull`만으로 동작

### 원격 추적 브랜치가 업데이트되는 시점
1. **fetch할 때**: 원격의 최신 상태를 origin/main에 반영
2. **push할 때**: push 성공 후 origin/main도 업데이트
3. **pull할 때**: 내부적으로 fetch하므로 origin/main 업데이트

## 구조 다이어그램

```
[원격 저장소]
  └── main (커밋 C)

[로컬 저장소]
  ├── main          (커밋 B)  ← 내가 작업하는 브랜치
  └── origin/main   (커밋 A)  ← 마지막 fetch/push 시점의 원격 상태

fetch 후:
  ├── main          (커밋 B)  ← 그대로
  └── origin/main   (커밋 C)  ← 원격과 동기화됨!
```

## 자주 하는 실수
- fetch가 스테이징 영역에 가져온다고 착각 → **원격 추적 브랜치를 업데이트하는 것!**
- push하면 origin/main은 안 바뀐다고 착각 → **push도 origin/main을 업데이트함!**
- **merge만 하면 origin/main은 업데이트 안 됨!** → origin/main을 업데이트하는 건 오직 **fetch, push, pull** 3개뿐. 로컬에서 merge는 로컬 브랜치만 변경한다.

---

## 실습 가이드: 원격 추적 브랜치 직접 확인하기

### 준비

```bash
# 실습 폴더 생성
mkdir ~/git-tracking-practice && cd ~/git-tracking-practice

# bare repo (원격 역할)
git init --bare remote-repo.git

# 로컬 저장소 생성
git init local
cd local
git remote add origin ../remote-repo.git

# 첫 커밋 & push
echo "hello" > hello.txt
git add . && git commit -m "initial commit"
git push -u origin main
```

### Step 1: 원격 추적 브랜치 확인

```bash
# 모든 브랜치 보기 (로컬 + 원격 추적)
git branch -a
# → * main
# → remotes/origin/main

# upstream 연결 상태 확인
git branch -vv
# → * main  abc1234 [origin/main] initial commit
#            ↑ 해시    ↑ upstream 연결 상태
```

### Step 2: fetch 후 origin/main 변화 관찰

```bash
# 원격에 직접 커밋을 만들기 위해 다른 clone 사용
cd ~/git-tracking-practice
git clone remote-repo.git other-clone
cd other-clone
echo "원격에서 변경" >> hello.txt
git add . && git commit -m "remote change"
git push

# 다시 local로 돌아와서 fetch
cd ~/git-tracking-practice/local
git fetch

# origin/main은 업데이트됐지만, 내 main은 그대로!
git log --oneline main
# → abc1234 initial commit (1개)

git log --oneline origin/main
# → def5678 remote change (2개!)

# 아직 push 안 한 커밋 확인 (로컬에만 있는 것)
git log origin/main..main --oneline
# → (비어있음 — 로컬이 뒤처진 상태)

# 원격에만 있는 커밋 확인
git log main..origin/main --oneline
# → def5678 remote change
```

### Step 3: merge해도 origin/main은 안 변함

```bash
# 로컬에서 새 커밋 만들기
echo "로컬 작업" >> local.txt
git add . && git commit -m "local work"

# merge로 origin/main 내용 합치기
git merge origin/main

# origin/main은 여전히 merge 전 시점을 가리킴
git log --oneline origin/main
# → 변화 없음! merge는 origin/main을 업데이트하지 않음

# push하면 그때 origin/main이 업데이트됨
git push
git log --oneline origin/main
# → 이제 최신!
```

### 정리

```bash
rm -rf ~/git-tracking-practice
```

---

## 주의사항 & 흔한 실수 (보충)

- **origin/main이 업데이트되는 건 3가지 경우뿐**: `fetch`, `push`, `pull`. 로컬에서 아무리 merge, rebase, commit을 해도 origin/main은 그대로다.
- `git branch -vv`에서 `[ahead 2]`는 push 안 한 커밋이 2개, `[behind 3]`은 fetch/pull 안 한 커밋이 3개라는 뜻
- 원격 저장소에서 브랜치가 삭제돼도 로컬의 원격 추적 브랜치는 남아있음 → `git fetch --prune`으로 정리
