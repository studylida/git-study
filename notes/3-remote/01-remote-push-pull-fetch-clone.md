# remote, push, pull, fetch, clone

## 핵심 개념

### clone - 원격 저장소 복제
- `git clone <URL>`: 원격 저장소를 로컬에 복제
- 자동으로 `origin`이라는 remote 이름이 설정됨
- 모든 브랜치, 커밋 히스토리가 함께 복제됨

### remote - 원격 저장소 관리
- `git remote`: 연결된 원격 저장소 목록 확인
- `git remote -v`: URL까지 상세 확인 (fetch/push 각각)
- `git remote add <이름> <URL>`: 원격 저장소 연결 (보통 `origin` 사용)
- `git remote remove <이름>`: 원격 저장소 연결 해제

### push - 로컬 → 원격 업로드
- `git push origin main`: 로컬 main을 원격 origin의 main에 업로드
- `git push -u origin main`: upstream 설정 + push (이후 `git push`만으로 가능)
- push하면 **원격 추적 브랜치(origin/main)도 함께 업데이트됨**

### fetch - 원격 → 로컬 다운로드 (병합 X)
- `git fetch`: 원격 저장소의 최신 정보를 가져옴
- **원격 추적 브랜치(origin/main)만 업데이트** (로컬 main은 그대로!)
- 스테이징 영역이나 워킹 디렉토리에는 영향 없음
- 안전하게 원격 변경사항을 먼저 확인하고 싶을 때 사용

### pull - fetch + merge
- `git pull`: fetch한 뒤 자동으로 현재 브랜치에 merge
- `git pull` = `git fetch` + `git merge origin/main`
- 편하지만 예상치 못한 충돌이 발생할 수 있음

## fetch vs pull 차이

| 구분 | fetch | pull |
|------|-------|------|
| 동작 | 다운로드만 | 다운로드 + 병합 |
| 안전성 | 안전 (확인 후 병합 가능) | 바로 병합 (충돌 가능) |
| 업데이트 대상 | 원격 추적 브랜치만 | 원격 추적 브랜치 + 로컬 브랜치 |
| 사용 시점 | 변경사항 먼저 확인할 때 | 바로 최신 상태로 맞출 때 |

## 흐름 정리

```
[원격 저장소]  ←── push ──  [로컬 저장소]
     │                          ↑
     │── fetch → [origin/main] ─┤
     │                          │
     └── pull ──────────────────┘
              (fetch + merge)
```

---

## 실습 가이드: 처음부터 따라하기

### 준비: 로컬 bare repo로 원격 저장소 시뮬레이션

```bash
# 1. 실습용 폴더 만들기
mkdir ~/git-remote-practice && cd ~/git-remote-practice

# 2. "원격 저장소" 역할을 할 bare repo 생성
git init --bare remote-repo.git
# → bare repo는 워킹 디렉토리 없이 .git 내용만 있는 저장소

# 3. "로컬 저장소 A" 생성 (개발자 A)
git init local-a
cd local-a
git remote add origin ../remote-repo.git
# → origin이라는 이름으로 원격 저장소 연결
```

### Step 1: push 해보기

```bash
# local-a에서 작업
echo "첫 번째 파일" > hello.txt
git add hello.txt
git commit -m "first commit"

# 원격에 push
git push -u origin main
# → -u로 upstream 설정. 이후 git push만으로 가능
# → 결과: remote-repo.git에 main 브랜치 생성됨
```

### Step 2: clone 해보기

```bash
# 상위 폴더로 이동
cd ~/git-remote-practice

# "로컬 저장소 B" — clone으로 생성 (개발자 B)
git clone remote-repo.git local-b
cd local-b

# remote 확인
git remote -v
# → origin이 자동으로 설정되어 있음
```

### Step 3: fetch vs pull 비교

```bash
# local-a에서 새 커밋 만들기
cd ~/git-remote-practice/local-a
echo "A가 추가한 내용" >> hello.txt
git add . && git commit -m "update from A"
git push

# local-b에서 fetch로 확인
cd ~/git-remote-practice/local-b
git fetch
# → origin/main만 업데이트됨. 내 main은 그대로!

git log --oneline --all
# → origin/main이 앞서 있는 걸 확인

git diff main origin/main
# → 차이점 확인 가능

# 확인 후 merge
git merge origin/main
# → 이제 로컬 main도 최신!
```

### Step 4: pull은 한 번에

```bash
# local-a에서 또 커밋
cd ~/git-remote-practice/local-a
echo "또 추가" >> hello.txt
git add . && git commit -m "another update from A"
git push

# local-b에서 pull (fetch + merge 한 번에)
cd ~/git-remote-practice/local-b
git pull
# → fetch + merge가 동시에 일어남
```

### 정리

```bash
# 실습 끝나면 정리
rm -rf ~/git-remote-practice
```

---

## 주의사항 & 흔한 실수

- `git push`할 때 원격에 내 변경사항과 충돌하는 커밋이 있으면 **rejected** 됨 → 먼저 pull 후 push
- `git pull`로 바로 받으면 편하지만, **예상 못한 merge 충돌**이 발생할 수 있음 → 중요한 작업 전에는 fetch → diff → merge 순서 추천
- remote 이름은 꼭 `origin`일 필요 없음. 관례일 뿐!
- `git push`만 하면 **tag는 올라가지 않음**. 태그는 별도로 push 필요
