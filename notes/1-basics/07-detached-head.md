# Detached HEAD

## 핵심 요약

- **Detached HEAD** = HEAD가 브랜치 없이 커밋을 직접 가리키는 상태
- 과거 코드를 구경하거나 임시 실험할 때 사용
- 커밋하려면 반드시 브랜치를 먼저 만들 것!

## 정상 상태 vs Detached HEAD

| 구분 | 정상 상태 | Detached HEAD |
|------|----------|---------------|
| `.git/HEAD` | `ref: refs/heads/브랜치명` | 커밋 해시 직접 |
| HEAD가 가리키는 것 | 브랜치 → 커밋 | 커밋 직접 |
| 커밋하면 | 브랜치가 자동으로 따라감 | 미아 커밋 위험! |

```
정상:     HEAD → main → C3
Detached: HEAD → C1 (브랜치 없음)
```

## Detached HEAD 진입 방법

```bash
# 방법 1: checkout (간단)
git checkout <커밋해시>

# 방법 2: switch (명시적)
git switch --detach <커밋해시>
```

## 탈출 방법

```bash
# 원래 브랜치로 돌아가기
git switch <브랜치이름>

# 새 브랜치 만들면서 탈출
git switch -c <새브랜치>
```

## 미아 커밋 구출

```bash
# 현재 위치에서 브랜치 만들기
git switch -c <새브랜치>
git branch <새브랜치>          # 생성만 (이동 안 함)

# 떠난 후에도 해시를 알면 구출 가능
git branch <브랜치이름> <커밋해시>
```

## 미아 커밋의 운명

1. 브랜치 없이 떠나면 → 미아 커밋
2. `git log --all`에도 안 보임
3. 일정 시간 후 (30~90일) garbage collection으로 자동 삭제
4. 그 전에는 `reflog`로 복구 가능 (4단계에서 학습)

---

# checkout vs switch vs restore

## 왜 나뉘었나?

예전에는 `git checkout` 하나가 너무 많은 역할을 했음 (브랜치 전환 + 파일 복원).
Git 2.23 (2019년)에서 역할별로 분리됨.

```
                    ┌─ git switch   → 브랜치 관련
git checkout ──────┤
                    └─ git restore  → 파일 복원 관련
```

## 명령어 비교표

| 하고 싶은 것 | 옛날 (checkout) | 지금 (권장) |
|-------------|----------------|------------|
| 브랜치 전환 | `git checkout main` | `git switch main` |
| 브랜치 생성+전환 | `git checkout -b feat` | `git switch -c feat` |
| Detached HEAD | `git checkout <해시>` | `git switch --detach <해시>` |
| 파일 변경 취소 | `git checkout -- file.txt` | `git restore file.txt` |
| 스테이징 취소 | 불편했음 | `git restore --staged file.txt` |
| 특정 커밋에서 파일 복원 | `git checkout <해시> -- file.txt` | `git restore --source <해시> file.txt` |

## restore 개념 정리

### 스테이징 취소 vs 파일 변경 취소

```
Repository  ──restore --staged──→  Staging Area  ──restore──→  Working Directory
 (커밋 저장소)                      (add한 공간)                 (작업 폴더)
```

- `git restore --staged file.txt`: Staging에서만 빼기 (파일 내용은 그대로)
- `git restore file.txt`: 작업 폴더의 수정 내용을 마지막 커밋으로 되돌리기 (복구 불가!)

### 특정 커밋에서 파일 복원

```bash
git restore --source <커밋해시> file.txt
```

- 커밋을 이동하지 않고, 과거 커밋에서 **그 파일만** 꺼내옴
- 다른 파일에는 영향 없음

---

## 실습 가이드: 처음부터 따라하기

```bash
# 1. 실습용 폴더 만들기
mkdir git-detached-practice && cd git-detached-practice
git init

# 2. 커밋 3개 만들기
echo "v1" > file.txt && git add . && git commit -m "커밋 1"
echo "v2" > file.txt && git add . && git commit -m "커밋 2"
echo "v3" > file.txt && git add . && git commit -m "커밋 3"

# 3. 현재 HEAD 상태 확인
cat .git/HEAD
# 결과: ref: refs/heads/main   ← 정상 (브랜치를 가리킴)

# === Detached HEAD 진입 ===
# 4. 과거 커밋으로 이동
git log --oneline           # 커밋 1의 해시 확인
git checkout HEAD~2         # 커밋 1로 이동
# 결과: You are in 'detached HEAD' state. 경고 메시지 출력

# 5. Detached HEAD 상태 확인
cat .git/HEAD
# 결과: ghi9012abc...       ← 해시값 직접 표시 (브랜치가 아님!)
cat file.txt                # v1 (커밋 1 시점의 파일)

# === 탈출 방법 1: 원래 브랜치로 돌아가기 ===
# 6. main으로 돌아가기
git switch main
cat .git/HEAD               # ref: refs/heads/main (정상 복귀!)
cat file.txt                # v3

# === 탈출 방법 2: 새 브랜치 만들면서 탈출 ===
# 7. 다시 Detached HEAD로 진입
git switch --detach HEAD~1  # 커밋 2로 이동 (switch 방식)

# 8. 여기서 커밋하면 미아가 됨! → 브랜치 만들어서 보존
echo "experiment" > exp.txt
git add exp.txt
git commit -m "실험 커밋"

# 9. 브랜치 만들어서 미아 방지
git switch -c experiment    # 새 브랜치 생성 + 전환
git log --oneline
# 결과: 실험 커밋이 experiment 브랜치에 안전하게 보존됨

# 10. main으로 돌아가기
git switch main

# 실습 끝! 정리하려면:
cd .. && rm -rf git-detached-practice
```
