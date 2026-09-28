# Tag - 버전 표시

## 핵심 개념

- **tag**: 특정 커밋에 붙이는 영구 이름표. 브랜치와 달리 움직이지 않는 포인터
- 브랜치 = 움직이는 포인터, 태그 = 고정 포인터

## 두 종류의 tag

### Lightweight tag
- 이름만 붙이는 태그. 메타데이터 없음
- `git tag v0.1`

### Annotated tag ⭐
- 만든 사람, 날짜, 메시지 포함. Git 객체로 저장됨
- `git tag -a v1.0 -m "정식 릴리즈"`
- 실무에서는 거의 항상 annotated tag 사용

## 주요 명령어

| 명령어 | 설명 |
|--------|------|
| `git tag` | tag 목록 보기 |
| `git tag -l "v1.*"` | 패턴으로 필터링 (-l 필수!) |
| `git show v1.0` | tag 상세 정보 |
| `git tag -a v1.0 -m "메시지"` | annotated tag 생성 |
| `git tag -a v0.5 -m "msg" <해시>` | 과거 커밋에 tag 붙이기 |
| `git tag -d v1.0` | 로컬 tag 삭제 |
| `git push origin v1.0` | 특정 tag push |
| `git push origin --tags` | 모든 tag push |
| `git push origin --delete v1.0` | 원격 tag 삭제 |

## 중요 포인트

- **git push만 하면 tag는 안 올라감!** 반드시 별도로 push 필요
- 이유: 테스트용 임시 tag까지 자동으로 올라가는 걸 방지
- 로컬 + 원격 둘 다 삭제해야 완전히 제거됨

---

## 실습 가이드: 태그 생성부터 삭제까지

### 준비

```bash
mkdir ~/git-tag-practice && cd ~/git-tag-practice
git init

# 커밋 3개 만들기 (버전 히스토리 시뮬레이션)
echo "v1 코드" > app.py
git add . && git commit -m "feat: 첫 번째 기능"

echo "v1.1 코드" >> app.py
git add . && git commit -m "fix: 버그 수정"

echo "v2 코드" >> app.py
git add . && git commit -m "feat: 대규모 업데이트"
```

### Step 1: Lightweight vs Annotated 태그

```bash
# Lightweight tag (메타데이터 없음)
git tag v0.1

# Annotated tag (메타데이터 포함 — 실무에서는 이걸 쓴다)
git tag -a v1.0 -m "첫 정식 릴리즈"

# 태그 목록 확인
git tag
# → v0.1
# → v1.0

# 차이 비교
git show v0.1
# → 커밋 정보만 보임

git show v1.0
# → 태그 만든 사람, 날짜, 메시지 + 커밋 정보
```

### Step 2: 과거 커밋에 태그 붙이기

```bash
# 커밋 히스토리 확인
git log --oneline
# → abc1234 feat: 대규모 업데이트
# → def5678 fix: 버그 수정
# → ghi9012 feat: 첫 번째 기능

# 두 번째 커밋에 태그 붙이기
git tag -a v0.5 -m "버그 수정 버전" def5678
# → 해시값은 git log에서 확인한 값 사용

# 태그별 커밋 확인
git log --oneline --decorate
# → 각 커밋에 태그가 붙어있는 걸 확인
```

### Step 3: 태그 필터링

```bash
# 패턴으로 검색 (-l 필수!)
git tag -l "v1.*"
# → v1.0

git tag -l "v0.*"
# → v0.1
# → v0.5
```

### Step 4: 원격에 태그 push & 삭제

```bash
# bare repo로 원격 시뮬레이션
cd ~/git-tag-practice
git clone --bare . remote-repo.git
git remote add origin remote-repo.git

# 특정 태그만 push
git push origin v1.0

# 모든 태그 push
git push origin --tags

# 로컬 태그 삭제
git tag -d v0.1

# 원격 태그 삭제
git push origin --delete v0.5
```

### 정리

```bash
rm -rf ~/git-tag-practice
```

---

## 주의사항 & 흔한 실수

- **`git push`만 하면 태그는 안 올라감!** 반드시 `git push origin --tags` 또는 `git push origin v1.0`으로 별도 push
- 태그 이름은 한 번 붙이면 **같은 이름으로 재사용 불가** (삭제 후 다시 붙여야 함)
- 이미 push한 태그를 수정하려면: 로컬 삭제 → 원격 삭제 → 다시 생성 → 다시 push
- `git tag -l` 없이 `git tag "v1.*"` 하면 **v1.*이라는 태그를 생성**해버림! 필터링은 반드시 `-l` 옵션과 함께
