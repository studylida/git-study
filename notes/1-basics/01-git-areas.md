# Git의 3가지 영역

## 핵심 개념

```
Working Directory → Staging Area → Repository
    (작업 공간)        (대기 공간)      (.git 폴더)
```

### Working Directory
- 실제 파일을 편집하는 공간
- 일반 폴더처럼 보이는 곳

### Staging Area
- 다음 커밋에 포함시킬 파일들을 모아두는 곳
- `git add`로 파일을 올림

### Repository
- `.git` 폴더 안에 저장된 커밋 히스토리
- `git commit`으로 스냅샷 저장

---

## Staging이 필요한 이유

원하는 파일만 골라서 커밋 가능!

```bash
# 전체가 아닌 특정 파일만 커밋
git add file1.txt
git commit -m "file1만 커밋"
```

---

## 상태 흐름

```
[Untracked] --add--> [Staged] --commit--> [Committed]
     ↑                                         |
     └─────────────── 수정 ────────────────────┘
```

---

## 실습 가이드: 3가지 영역 직접 체험하기

```bash
# 1. 실습용 폴더 만들기
mkdir git-areas-practice && cd git-areas-practice
git init

# 2. 파일 생성 → Untracked 상태
echo "Hello" > hello.txt
git status
# 결과: Untracked files: hello.txt
# (Working Directory에만 존재, Git이 아직 모름)

# 3. git add → Staging Area로 이동
git add hello.txt
git status
# 결과: Changes to be committed: new file: hello.txt
# (Staging Area에 올라감, 커밋 대기 중)

# 4. git commit → Repository로 저장
git commit -m "init: hello.txt 추가"
git status
# 결과: nothing to commit, working tree clean
# (3개 영역이 모두 동일한 상태)

# 5. 파일 수정 → Working Directory에서 변경 발생
echo "World" >> hello.txt
git status
# 결과: Changes not staged for commit: modified: hello.txt
# (Working Directory만 바뀜, Staging/Repository는 아직 이전 버전)

# 6. 선택적 커밋 체험 - 파일 2개 만들고 1개만 커밋
echo "file A" > a.txt
echo "file B" > b.txt
git add a.txt              # a.txt만 Staging에 올림
git commit -m "feat: a.txt만 커밋"
git status
# 결과: b.txt는 여전히 Untracked (선택적 커밋 성공!)

# 실습 끝! 정리하려면:
cd .. && rm -rf git-areas-practice
```
