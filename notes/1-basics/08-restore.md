# git restore - 변경사항 취소

## 핵심 명령어

| 명령어 | 용도 | 안전성 |
|--------|------|--------|
| `git restore <파일>` | 워킹 디렉토리 변경 취소 | 위험 (수정 내용 삭제) |
| `git restore --staged <파일>` | 스테이징 취소 (add 취소) | 안전 (내용 유지) |
| `git restore --source <커밋> <파일>` | 특정 커밋 시점으로 되돌리기 | - |
| `git restore .` | 모든 변경 파일 한 번에 되돌리기 | 위험 |

## 3가지 영역에서의 동작

```
[Repository]
     │
     │  git restore --staged <파일>
     ▼
[Staging Area]
     │
     │  git restore <파일>
     ▼
[Working Directory]
```

## restore vs reset vs revert

| 명령어 | 대상 | 용도 |
|--------|------|------|
| `restore` | 파일 | 파일 변경사항/스테이징 취소 |
| `reset` | 커밋 | 커밋 히스토리를 되돌림 |
| `revert` | 커밋 | 취소 커밋을 새로 만듦 |

**파일 단위 = restore, 커밋 단위 = reset/revert**

## 복합 상황: staged + 파일 내용 모두 되돌리기

```bash
git restore --staged <파일>   # 1) 스테이징 취소 (먼저!)
git restore <파일>            # 2) 워킹 디렉토리 되돌리기
```

순서 중요: staged를 먼저 풀고, 그다음 파일을 되돌려야 함.

## checkout과의 관계

Git 2.23에서 checkout의 역할이 분리됨:
- `git checkout -- <파일>` → `git restore <파일>`
- `git checkout <브랜치>` → `git switch <브랜치>`

---

## 주의사항 & 흔한 실수

### `--staged` 옵션을 자주 빠뜨림

```bash
# ❌ 스테이징 취소하려는데 --staged를 빠뜨리면?
git restore file.txt            # Working Directory 변경을 되돌림! (수정 내용 삭제됨!)

# ✅ 스테이징만 취소하려면 반드시 --staged
git restore --staged file.txt   # Staging에서만 빼기 (파일 내용 유지)
```

**외우는 팁**: "staging 취소 = `--staged` 필수!" — 옵션 없이 쓰면 **파일 내용이 날아간다!**

### restore vs reset vs revert 구분법

자주 헷갈리는 세 명령어 한눈에 정리:

```
┌──────────┬──────────────┬──────────────────────────────────────┐
│ 명령어    │ 대상         │ 하는 일                               │
├──────────┼──────────────┼──────────────────────────────────────┤
│ restore  │ 파일 단위     │ 파일 변경 취소 / 스테이징 취소         │
│ reset    │ 커밋 단위     │ 커밋 히스토리를 과거로 되돌림 (삭제)    │
│ revert   │ 커밋 단위     │ 취소하는 새 커밋을 만듦 (안전)         │
└──────────┴──────────────┴──────────────────────────────────────┘
```

**외우는 팁**:
- **restore** = "파일 복원" (re-store = 다시 저장)
- **reset** = "리셋, 초기화" (히스토리 자체를 되돌림)
- **revert** = "되돌리기 커밋" (새 커밋으로 취소 기록 남김)
- 파일 하나만 건드리고 싶으면 → `restore`
- 커밋을 없애고 싶으면 → `reset` (push 전) 또는 `revert` (push 후)

---

## 실습 가이드: 처음부터 따라하기

```bash
# 1. 실습용 폴더 만들기
mkdir git-restore-practice && cd git-restore-practice
git init

# 2. 초기 파일 만들고 커밋
echo "original content" > file.txt
git add file.txt
git commit -m "init: 초기 파일"

# === restore: 워킹 디렉토리 변경 취소 ===
# 3. 파일 수정
echo "modified content" > file.txt
cat file.txt                # modified content

# 4. 수정 취소 (워킹 디렉토리 되돌리기)
git restore file.txt
cat file.txt                # original content (원래대로!)

# === restore --staged: 스테이징 취소 ===
# 5. 파일 수정 후 add
echo "new content" > file.txt
git add file.txt
git status -s               # M  file.txt (staged)

# 6. 스테이징만 취소 (파일 내용은 유지)
git restore --staged file.txt
git status -s               #  M file.txt (unstaged로 바뀜)
cat file.txt                # new content (내용은 그대로!)

# === 복합 상황: staged + 파일 내용 모두 되돌리기 ===
# 7. 다시 add
git add file.txt

# 8. 스테이징 취소 → 파일 되돌리기 (순서 중요!)
git restore --staged file.txt   # 먼저 staging 취소
git restore file.txt            # 그다음 파일 되돌리기
cat file.txt                    # original content (완전 원복!)

# === --source: 특정 커밋에서 파일 복원 ===
# 9. 커밋 2개 더 만들기
echo "version 2" > file.txt && git add . && git commit -m "v2"
echo "version 3" > file.txt && git add . && git commit -m "v3"

# 10. 첫 번째 커밋의 파일 내용 복원
git log --oneline           # 커밋 해시 확인
git restore --source HEAD~2 file.txt
cat file.txt                # original content (첫 커밋 시점의 내용!)
git status -s               #  M file.txt (워킹 디렉토리에 변경으로 표시)

# 11. 되돌리기 (현재 커밋 내용으로)
git restore file.txt
cat file.txt                # version 3

# 실습 끝! 정리하려면:
cd .. && rm -rf git-restore-practice
```
