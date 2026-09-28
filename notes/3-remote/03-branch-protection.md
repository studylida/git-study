# 브랜치 보호 규칙 (Branch Protection Rules)

## 개념

특정 브랜치(보통 main)에 규칙을 걸어서 실수를 시스템 차원에서 방지하는 기능.

## 설정 위치

GitHub → Settings → Rules → Rulesets → New branch ruleset

## 주요 규칙

| 규칙 | 효과 |
|------|------|
| **Require a pull request before merging** | main 직접 push 차단, PR 필수 |
| **Require approvals** | 리뷰어 승인 필수 (0명도 가능) |
| **Require status checks** | CI 테스트 통과해야 머지 가능 |
| **Block force pushes** | `--force` push 차단 |
| **Restrict deletions** | 브랜치 삭제 금지 |
| **Require linear history** | 머지 커밋 금지 (rebase/squash만 허용) |
| **Include administrators** | 관리자도 규칙 적용 |

## 보호 규칙 있을 때 워크플로우

```
브랜치 생성 → 작업 → add → commit → push → PR 생성 → (리뷰) → 머지 → main pull → 브랜치 삭제
```

main에 직접 push하면:
```
! [remote rejected] main -> main (push declined due to repository rule violations)
```

## CLI 명령어

```bash
gh ruleset list              # 적용된 ruleset 목록
gh ruleset view <ID>         # 상세 내용 확인
git fetch --prune            # 원격에서 삭제된 브랜치 추적 정보 정리
```

## 실전 교훈

- 커밋 없이 PR 만들면 실패 (`No commits between main and <branch>`)
- 브랜치는 main이 깨끗한 상태에서 만들기 (안 그러면 쓰레기 커밋이 딸려감)
