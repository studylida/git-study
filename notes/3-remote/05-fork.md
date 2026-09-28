# Fork

## Fork란?

GitHub에서 다른 사람의 레포를 **내 계정으로 복사**하는 것. 서버 간 복사(GitHub → GitHub).
원본에 push 권한이 없을 때, Fork해서 작업 후 PR로 기여하는 방식.

## Fork vs Clone

| | Fork | Clone |
|------|------|-------|
| 어디서 | GitHub → GitHub (서버 간) | GitHub → 내 PC (로컬) |
| 결과 | 내 계정에 레포 생성 | 내 PC에 폴더 생성 |
| push | 내 Fork에 가능 | 원본에는 권한 필요 |
| 용도 | 오픈소스 기여 | 일반적인 작업 |

## Remote 구조

Fork 후에는 remote가 2개:

```
origin   → 내 Fork (push 가능)
upstream → 원본 레포 (읽기용)
```

## 오픈소스 기여 흐름

```
1. Fork (GitHub에서 내 계정으로 복사)
2. Clone (내 PC로 다운로드)
3. git remote add upstream <원본URL> (upstream 설정)
4. 브랜치 만들고 작업
5. git push origin <브랜치> (내 Fork에 push)
6. 원본 레포에 PR 생성
7. 원본 관리자가 리뷰 후 머지
```

## Upstream 동기화

내 Fork를 원본 최신 상태로 맞추기:

```bash
git fetch upstream          # 원본 최신 가져오기
git switch main             # main으로 이동
git merge upstream/main     # 원본과 합치기
git push origin main        # 내 Fork에도 반영
```

동기화 안 하면 PR 만들 때 충돌 발생 가능!

## 주요 명령어

```bash
# Fork + Clone + upstream 한 번에
gh repo fork <원본레포> --clone=true

# upstream 수동 설정
git remote add upstream <원본URL>

# upstream 동기화
git fetch upstream
git merge upstream/main

# 원본에 PR 보내기
gh pr create --repo <원본레포> --title "제목" --body "설명"
```

---

## 실습 가이드: Fork & PR 기여 흐름 따라하기

> 실제 오픈소스에 기여하는 전체 과정을 연습합니다.
> 연습용으로 자신의 다른 레포를 Fork하거나, GitHub의 공개 연습 레포를 사용하세요.

### Step 1: Fork & Clone

```bash
# GitHub CLI로 Fork + Clone 한 번에
gh repo fork <원본레포-URL> --clone=true
# 예: gh repo fork https://github.com/octocat/Spoon-Knife --clone=true

cd Spoon-Knife    # Fork된 레포 폴더로 이동

# remote 확인
git remote -v
# → origin    https://github.com/내계정/Spoon-Knife.git (내 Fork)
# → upstream  https://github.com/octocat/Spoon-Knife.git (원본)
```

### Step 2: 브랜치 만들고 작업

```bash
# 작업 브랜치 생성
git switch -c feature/my-contribution

# 파일 수정
echo "내가 기여한 내용" >> README.md
git add . && git commit -m "docs: README에 설명 추가"
```

### Step 3: 내 Fork에 push & PR 생성

```bash
# 내 Fork에 push (origin)
git push -u origin feature/my-contribution

# 원본 레포에 PR 보내기
gh pr create --repo octocat/Spoon-Knife \
  --title "docs: README 설명 추가" \
  --body "README에 사용법 설명을 추가했습니다."
```

### Step 4: upstream 동기화 (시간이 지난 후)

```bash
# 원본 최신 가져오기
git fetch upstream

# main 브랜치로 이동해서 동기화
git switch main
git merge upstream/main

# 내 Fork에도 반영
git push origin main
```

### 만약 upstream이 설정 안 되어 있다면

```bash
# 수동으로 upstream 추가
git remote add upstream <원본레포-URL>
git fetch upstream
```

---

## 주의사항 & 흔한 실수

- **절대 main에서 직접 작업하지 말 것!** Fork한 main은 원본과 동기화용으로 깨끗하게 유지해야 함
- upstream 동기화를 안 하면 PR 만들 때 **conflict** 발생 가능 → PR 전에 항상 동기화
- `origin`과 `upstream`을 헷갈리지 말 것: **origin = 내 Fork**, **upstream = 원본**
- Fork한 레포에서 이슈를 만들면 **내 Fork의 이슈**가 됨. 원본에 이슈를 남기려면 원본 레포에서 만들어야 함
