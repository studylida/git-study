# Issue & Project

## Issue란?

GitHub에서 "해야 할 일"을 기록하는 티켓. 버그 리포트, 기능 요청, 질문 등을 관리.

## Issue 구성 요소

| 요소 | 설명 |
|------|------|
| Title | 이슈 제목 |
| Body | 상세 설명 (마크다운 가능) |
| Assignee | 담당자 |
| Label | 분류 태그 (bug, enhancement, docs 등) |
| Milestone | 목표 기한/버전 |
| Project | 연결된 프로젝트 보드 |

## 기본 라벨

- `bug` - 버그
- `enhancement` - 새 기능 요청
- `documentation` - 문서 관련
- `question` - 질문
- `good first issue` - 초보자용 이슈
- `wontfix` - 안 고칠 것
- `duplicate` - 중복 이슈
- `invalid` - 유효하지 않음

## Issue ↔ PR 자동 연결

PR 본문이나 커밋 메시지에 아래 키워드 + 이슈 번호를 쓰면, PR 머지 시 이슈가 자동 Close됨:

```
closes #12
fixes #12
resolves #12
```

**주의**: `removes`, `deletes` 등은 키워드가 아님!

## Milestone

여러 이슈를 하나의 목표(버전, 기한)로 묶는 기능. 진행률을 한눈에 파악 가능.

- 웹: Issues 탭 → Milestones → New milestone
- CLI: `gh api repos/{owner}/{repo}/milestones` 사용

## Project (V2)

칸반 보드 형태로 이슈를 시각적으로 관리.

- Todo → In Progress → Done 형태로 드래그 이동
- 웹: Projects 탭 → New project → Board 선택

## 주요 CLI 명령어

```bash
# 이슈 생성
gh issue create --title "제목" --body "내용" --label "라벨"

# 이슈 목록
gh issue list
gh issue list --state all        # 닫힌 이슈 포함
gh issue list --label "bug"      # 특정 라벨만

# 이슈 조회/닫기/열기
gh issue view <번호>
gh issue close <번호> --reason completed
gh issue close <번호> --reason "not planned"
gh issue reopen <번호>

# 이슈 편집
gh issue edit <번호> --add-assignee @me
gh issue edit <번호> --add-label "bug"
```

---

## 실습 가이드: Issue & PR 워크플로우 따라하기

> 이 실습은 GitHub 저장소가 필요합니다. 실습용 레포를 하나 만들어서 진행하세요.

### 준비

```bash
# 실습용 레포 생성 (GitHub CLI 사용)
gh repo create issue-practice --public --clone
cd issue-practice

# 첫 커밋
echo "# Issue Practice" > README.md
git add . && git commit -m "init"
git push -u origin main
```

### Step 1: Issue 만들기

```bash
# CLI로 이슈 생성
gh issue create --title "로그인 버그 수정" --body "로그인 시 500 에러 발생" --label "bug"
# → Created issue #1

# 이슈 확인
gh issue list
gh issue view 1
```

### Step 2: Issue와 연결된 PR 만들기

```bash
# 브랜치 만들고 작업
git switch -c fix/login-bug
echo "버그 수정 코드" > fix.txt
git add . && git commit -m "fix: 로그인 버그 수정

closes #1"
# → 커밋 메시지에 "closes #1" 포함!

git push -u origin fix/login-bug

# PR 생성
gh pr create --title "로그인 버그 수정" --body "closes #1"
```

### Step 3: PR 머지 후 이슈 자동 닫힘 확인

```bash
# PR 머지
gh pr merge --merge

# 이슈 상태 확인
gh issue view 1
# → State: CLOSED  ← 자동으로 닫혔다!
```

### Step 4: Milestone 활용

```bash
# 웹에서: Issues → Milestones → New milestone
# 제목: "v1.0 출시", 기한 설정

# 이슈 생성 시 milestone 연결
gh issue create --title "새 기능 추가" --milestone "v1.0 출시"
```

### 정리

```bash
# 실습용 레포 삭제 (선택)
gh repo delete issue-practice --yes
```

---

## 주의사항 & 흔한 실수

- **`closes #1`은 이슈를 닫는 것이지, PR을 닫는 게 아님!** `closes`, `fixes`, `resolves` 키워드는 **이슈 번호**와 함께 사용해서 **이슈를 자동으로 닫는** 기능이다. PR은 merge하면 자동으로 닫힌다.
- 키워드는 대소문자 구분 없음: `Closes`, `closes`, `CLOSES` 모두 동작
- `removes`, `deletes`, `completes` 같은 건 **키워드가 아님** — 이슈가 안 닫힘
- PR 본문에 `closes #1`을 넣으면 **PR이 merge될 때** 이슈가 닫힘. 커밋 메시지에 넣으면 **해당 커밋이 default branch에 merge될 때** 닫힘
- 다른 레포의 이슈를 닫으려면: `closes owner/repo#번호` 형식 사용
