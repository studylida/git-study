# Git Hooks - 커밋 전후 자동 실행 스크립트

## Hook이란?

Git에서 **특정 이벤트가 발생할 때 자동으로 실행되는 스크립트**. 셸 스크립트(bash)로 작성.

```
커밋 실행 → Hook 발동 → 스크립트 실행 → 통과(exit 0) → 커밋 완료
                                       → 실패(exit 1) → 커밋 차단!
```

## Hook 파일 위치

```
.git/hooks/
├── pre-commit.sample    ← .sample 붙어있으면 비활성화
├── commit-msg.sample
├── pre-push.sample
└── ...
```

- `.sample` 확장자를 제거하면 활성화
- `chmod +x` 실행 권한 필수

## 주요 Hook 종류

### 커밋 과정

| Hook | 시점 | 용도 | 차단 가능 |
|------|------|------|----------|
| **pre-commit** | 커밋 직전 | 코드 검사, 린트, 포맷 체크 | ✅ |
| **commit-msg** | 메시지 작성 후 | 커밋 메시지 형식 검사 | ✅ |
| **post-commit** | 커밋 완료 후 | 알림 보내기 | ❌ |

### Push 과정

| Hook | 시점 | 용도 | 차단 가능 |
|------|------|------|----------|
| **pre-push** | push 직전 | 테스트 실행, 브랜치 체크 | ✅ |

## 실행 순서

```
git commit → pre-commit → commit-msg → (커밋 생성) → post-commit
git push   → pre-push → (push 실행)
```

## Hook의 규칙

1. **실행 권한 필요**: `chmod +x .git/hooks/<hook-name>`
2. **exit 0** = 통과 (계속 진행)
3. **exit 1** = 실패 (작업 차단)
4. **로컬 전용**: `.git/hooks/`는 push 안 됨 → 팀원과 자동 공유 안 됨

## 실습 예제

### pre-commit: TODO/console.log 검사

```bash
#!/bin/bash
echo "커밋 전 검사 중..."

if git diff --cached --name-only | xargs grep -lE "(TODO|console.log)" 2>/dev/null; then
    echo "TODO 또는 console.log가 남아있는 파일 존재!"
    exit 1
fi

echo "통과! 커밋 진행"
exit 0
```

### commit-msg: 커밋 메시지 형식 강제

```bash
#!/bin/bash
MSG=$(cat "$1")

if ! echo "$MSG" | grep -qE "^(feat|fix|docs|style|refactor|test|chore):"; then
    echo "커밋 메시지는 prefix로 시작해야 함"
    echo "feat, fix, docs, style, refactor, test, chore"
    exit 1
fi

echo "커밋 메시지 형식 OK!"
exit 0
```

### pre-push: main 브랜치 push 차단

```bash
#!/bin/bash
BRANCH=$(git branch --show-current)

if [ "$BRANCH" = "main" ]; then
    echo "main 브랜치에서 직접 push 금지!"
    exit 1
fi

exit 0
```

### post-commit: 커밋 완료 알림

```bash
#!/bin/bash
HASH=$(git rev-parse --short HEAD)
MSG=$(git log -1 --pretty=%s)
echo "커밋 완료! [$HASH] $MSG"
```

## bash 스크립트 기본 문법

| 문법 | 의미 |
|------|------|
| `#!/bin/bash` | bash로 실행하겠다는 선언 (shebang) |
| `$(명령어)` | 명령어 실행 결과를 변수에 저장 |
| `if [ 조건 ]; then ... fi` | 조건문 |
| `"$변수" = "값"` | 문자열 비교 |
| `exit 0` | 성공 (통과) |
| `exit 1` | 실패 (차단) |
