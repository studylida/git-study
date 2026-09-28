# 기본 명령어

## git init

저장소 초기화 (`.git` 폴더 생성)

```bash
git init              # 현재 폴더를 저장소로
git init 폴더명        # 새 폴더에 저장소 생성
git init --bare       # 서버용 저장소 (작업 파일 없음)
```

---

## git status

현재 상태 확인

```bash
git status            # 전체 상태
git status -s         # 짧은 형식
git status -b         # 브랜치 정보 포함
```

### 짧은 형식 읽는 법
```
M  file.txt    # 왼쪽 M = staged
 M file.txt    # 오른쪽 M = unstaged (modified)
?? file.txt    # untracked
!! file.txt    # ignored
```

---

## git add

Staging Area로 파일 올리기

```bash
git add 파일명         # 특정 파일
git add .             # 현재 폴더 전체
git add -A            # 저장소 전체
git add *.txt         # 패턴 매칭
git add -p            # 변경사항 하나씩 확인하며 선택
```

---

## git commit

커밋 (스냅샷 저장)

```bash
git commit -m "메시지"      # 메시지와 함께 커밋
git commit                  # 편집기로 긴 메시지 작성
git commit -a -m "메시지"   # add + commit 동시에 (추적 중인 파일만)
git commit --amend          # 직전 커밋 수정
```

---

## git log

커밋 히스토리 확인

```bash
git log               # 전체 로그
git log --oneline     # 한 줄로 간략히
git log -3            # 최근 3개만
git log --graph       # 브랜치 그래프
git log -p            # 변경 내용까지 표시
```

---

## git commit --amend

직전 커밋을 수정 (새 커밋으로 교체됨)

```bash
git commit --amend -m "새 메시지"     # 메시지 수정
git commit --amend --no-edit          # 파일만 추가, 메시지 유지
```

### 주의사항
- 이미 push한 커밋은 amend 금지! (충돌 발생)
- amend하면 커밋 해시가 변경됨

---

## 주의사항 & 흔한 실수

### `git status -s` 왼쪽/오른쪽 헷갈림

```
M  file.txt     ← 왼쪽 M = Staging Area (add된 상태)
 M file.txt     ← 오른쪽 M = Working Directory (수정만 한 상태)
MM file.txt     ← 둘 다 = add 후에 또 수정함
```

**외우는 팁**: 왼쪽이 **S**taging (왼쪽 = Stage), 오른쪽이 **W**orking Directory (오른쪽 = Work)
- 파일 흐름도 `Working → Staging → Repository`이므로, 오른쪽(작업)에서 왼쪽(스테이지)으로 간다고 기억

### 커밋 명령어 오타

```bash
# ❌ 흔한 오타들
git commmit -m "메시지"    # m이 3개
git commti -m "메시지"     # 철자 틀림
git comit -m "메시지"      # m이 1개

# ✅ 올바른 명령어
git commit -m "메시지"
```

**팁**: 오타가 나면 Git이 `git: 'commmit' is not a git command.`라고 알려주고, 비슷한 명령어를 추천해줌. 당황하지 말고 다시 입력하면 됨!

---

## 실습 가이드: 처음부터 따라하기

```bash
# 1. 실습용 폴더 만들기
mkdir git-basic-practice && cd git-basic-practice

# 2. Git 저장소 초기화
git init
# 결과: Initialized empty Git repository in .../git-basic-practice/.git/

# 3. 파일 만들기
echo "Hello Git" > hello.txt
echo "Second file" > second.txt

# 4. 상태 확인 (두 파일 모두 Untracked)
git status -s
# 결과: ?? hello.txt
#        ?? second.txt

# 5. 하나만 스테이징
git add hello.txt

# 6. 상태 다시 확인 (hello는 staged, second는 untracked)
git status -s
# 결과: A  hello.txt       ← 왼쪽 A = Staging에 추가됨
#        ?? second.txt

# 7. 첫 커밋
git commit -m "feat: hello.txt 추가"
# 결과: [main (root-commit) abc1234] feat: hello.txt 추가

# 8. 로그 확인
git log --oneline
# 결과: abc1234 feat: hello.txt 추가

# 9. 파일 수정 후 상태 확인
echo "Modified" >> hello.txt
git status -s
# 결과:  M hello.txt       ← 오른쪽 M = Working Directory에서 수정됨
#        ?? second.txt

# 10. add 후 상태 변화 비교
git add hello.txt
git status -s
# 결과: M  hello.txt       ← 왼쪽 M = Staging에 올라감!
#        ?? second.txt

# 11. 커밋
git commit -m "feat: hello.txt 수정"

# 12. amend 실습 - 빠뜨린 파일 추가
git add second.txt
git commit --amend --no-edit
# 결과: 직전 커밋에 second.txt가 포함됨 (커밋 해시 변경됨)

# 13. 최종 로그 확인
git log --oneline
# 결과: def5678 feat: hello.txt 수정   ← second.txt도 포함됨
#        abc1234 feat: hello.txt 추가

# 실습 끝! 정리하려면:
cd .. && rm -rf git-basic-practice
```
