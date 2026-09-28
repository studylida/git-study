# git worktree

## 개념

`git worktree`는 하나의 Git 저장소에 여러 작업 폴더를 연결하는 기능이다. 각 worktree는 같은 객체와 브랜치 정보를 공유하지만 서로 다른 브랜치와 작업 파일을 가질 수 있다.

브랜치와 worktree는 같은 대상이 아니다. 브랜치는 커밋을 가리키는 포인터이고, worktree는 해당 브랜치의 파일을 펼쳐 놓고 작업하는 폴더다. Git은 같은 브랜치를 두 worktree에서 동시에 체크아웃하지 못하게 막는다.

## 실습

현재 연결 상태를 먼저 확인한다.

```bash
git worktree list
```

이미 존재하는 작업 브랜치를 저장소 밖의 폴더에 연결한다.

```bash
git worktree add /tmp/git-study-task docs/example-task
```

브랜치도 함께 만들려면 `-b`를 사용한다.

```bash
git worktree add -b docs/example-task /tmp/git-study-task main
```

원래 폴더를 떠나지 않고 새 worktree의 상태를 확인할 수 있다.

```bash
git -C /tmp/git-study-task status
```

작업이 끝났다면 변경사항이 없는지 확인한 뒤 worktree를 제거한다. worktree가 사용하던 로컬 브랜치는 별도로 삭제한다.

```bash
git worktree remove /tmp/git-study-task
git branch -d docs/example-task
```

## 주의사항

- 별도 worktree를 저장소 내부에 만들면 부모 저장소가 그 폴더를 추적되지 않은 파일로 볼 수 있으므로 저장소 밖의 경로를 사용한다.
- worktree 폴더를 파일 명령으로 직접 삭제하지 않고 `git worktree remove <경로>`를 사용한다.
- `git branch`에서 브랜치 앞에 `+`가 보이면 다른 worktree에서 체크아웃된 브랜치라는 뜻이다.
- 변경사항이 남은 worktree는 일반 제거가 거부된다. 강제 제거보다 해당 worktree의 `git status`를 확인하고 변경을 먼저 보존하는 편이 안전하다.
