# 다중 기기 GitHub 동기화 루틴

## 언제 필요한가?

노트북과 본체를 번갈아 쓰면 같은 GitHub 저장소를 두 로컬 환경에서 수정하게 된다. 이때 습관은 단순하다.

```text
시작: 원격 변경을 먼저 가져온다
작업: 파일을 수정한다
종료: 로컬 변경을 커밋하고 원격에 올린다
```

이 순서를 지키면 "본체에서 수정한 것을 노트북에서 모른 채 또 수정"하는 상황을 줄일 수 있다.

## 현재 study 구조 기준 대상 저장소

```text
2026-01
database-study
git-study
linux-study
regex-study
```

각 폴더는 독립된 Git 저장소다. 부모 `study` 저장소는 submodule 포인터만 기록한다.

## 시작 루틴

작업 시작 전에는 먼저 원격 상태를 가져온다.

```bash
cd /mnt/c/git/study

for d in 2026-01 database-study git-study linux-study regex-study; do
  echo "===== $d ====="
  git -C "$d" pull --ff-only
done
```

`--ff-only`는 자동 merge commit을 만들지 않는다. 원격과 로컬이 단순히 앞뒤 관계일 때만 이동하고, 서로 다른 커밋이 있으면 멈춘다.

멈추면 바로 덮어쓰거나 force push하지 말고 상태를 본다.

```bash
git -C 저장소이름 status -sb
git -C 저장소이름 log --oneline --left-right --graph HEAD...origin/main
```

## 종료 루틴

작업을 끝낼 때는 먼저 어떤 저장소가 바뀌었는지 확인한다.

```bash
cd /mnt/c/git/study

for d in 2026-01 database-study git-study linux-study regex-study; do
  echo "===== $d ====="
  git -C "$d" status -sb
done
```

변경된 저장소에 들어가서 커밋하고 push한다.

```bash
cd /mnt/c/git/study/git-study
git add -A
git commit -m "docs: update git notes"
git push
```

여러 저장소에 이미 커밋이 만들어져 있고 push만 남았다면:

```bash
cd /mnt/c/git/study

for d in 2026-01 database-study git-study linux-study regex-study; do
  echo "===== $d ====="
  git -C "$d" push
done
```

## `status -sb` 읽는 법

```text
## main...origin/main
```

로컬 main과 원격 main이 같은 상태다.

```text
## main...origin/main [ahead 2]
```

로컬 커밋 2개가 아직 GitHub에 올라가지 않았다. `git push`가 필요하다.

```text
## main...origin/main [behind 1]
```

GitHub에 로컬에 없는 커밋 1개가 있다. 작업 전 `git pull --ff-only`가 필요하다.

```text
## main...origin/main [ahead 1, behind 1]
```

로컬과 원격이 갈라졌다. 바로 push/pull하지 말고 로그를 확인해야 한다.

## submodule일 때 주의할 점

하위 저장소에서 커밋하고 push하는 것과, 부모 `study` 저장소가 기록하는 submodule 포인터를 업데이트하는 것은 다른 일이다.

예를 들어 `git-study`에서 새 커밋을 만들고 push하면:

```bash
cd /mnt/c/git/study/git-study
git add -A
git commit -m "docs: update notes"
git push
```

부모 `study`에서는 `git-study` 폴더가 수정된 것처럼 보일 수 있다.

```bash
cd /mnt/c/git/study
git status -sb
```

이 포인터 변경까지 공유하려면 부모 저장소에서도 커밋해야 한다.

```bash
git add git-study
git commit -m "chore: update git-study submodule"
```

단, 부모 `study` 저장소를 어떤 원격에 올릴지 정하지 않았다면 부모 커밋은 로컬에만 둘 수 있다.

## LFS 파일이 있을 때

큰 zip, pdf, 이미지, 결과 파일은 Git LFS 대상일 수 있다.

```bash
git lfs ls-files
git lfs status
```

push할 때 이런 출력이 보이면 LFS 업로드가 같이 진행된 것이다.

```text
Uploading LFS objects: 100% (...), done.
```

GitHub가 50MB 이상 파일 경고를 낼 수 있다. 100MB를 넘으면 일반 Git push는 거부되므로 LFS로 옮겨야 한다.

## 핵심 습관

- 시작 전: `pull --ff-only`
- 작업 중: 자주 `status -sb`
- 종료 전: 변경 저장소별 `add → commit → push`
- 여러 저장소가 있으면 `for` 루프로 상태를 한 번에 확인
- submodule은 하위 저장소 push와 부모 포인터 커밋을 구분
- LFS 파일은 `git lfs ls-files`로 관리 여부 확인

## 2026-07-02 실습 보강

### `fetch`와 `pull --ff-only` 차이

`git fetch origin`은 GitHub의 최신 정보를 로컬의 원격 추적 브랜치(`origin/main`)에 가져온다. 하지만 현재 작업 파일과 로컬 `main` 브랜치는 바꾸지 않는다.

`git pull --ff-only origin main`은 실행 시점에 다시 원격을 확인한 뒤, fast-forward 가능한 경우에만 현재 브랜치를 최신 커밋으로 이동한다. 그래서 `fetch`와 `pull` 사이에 누군가 `main`을 바꿔도 `pull` 시점에 다시 확인된다.

### submodule 시작 루틴

부모 저장소에서 먼저 상태를 확인하고 최신화한다.

```bash
cd ~/code
git status --short --branch
git fetch origin
git pull --ff-only origin main
```

그 다음 submodule 설정과 체크아웃 상태를 맞춘다.

```bash
git submodule sync --recursive
git submodule update --init --recursive
git submodule status
```

`git submodule update`는 submodule을 부모 저장소가 기록한 정확한 커밋으로 맞춘다. 이 과정에서 submodule이 `HEAD (no branch)` 상태가 될 수 있다.

submodule 안에서 작업할 예정이라면 브랜치로 이동한 뒤 시작한다.

```bash
git submodule foreach 'git switch main'
git submodule foreach 'git status --short --branch'
```

### `git submodule status` 앞 기호

- 공백: 부모 저장소가 기록한 커밋과 현재 submodule 커밋이 일치한다.
- `+`: 현재 submodule 커밋이 부모가 기록한 커밋과 다르다. 앞선 커밋이어도, 뒤처진 커밋이어도 다르면 `+`다.
- `-`: submodule이 아직 초기화되지 않았거나 체크아웃되지 않았다.
- `U`: merge 중 submodule 포인터가 충돌난 상태다. 두 브랜치가 같은 submodule을 서로 다른 커밋으로 기록했을 때 발생할 수 있다.

### 작업 시작 가능 상태

작업 시작 전에는 다음 조건을 확인한다.

- 부모 저장소가 `main` 위에 있고 `origin/main`과 같다.
- 각 submodule도 작업할 예정이면 `main` 위에 있고 `origin/main`과 같다.
- `git status`에 `M`, `A`, `??` 같은 변경사항이 없다.
- LFS 상태에 commit/push 대기 항목이 없다.

## 회사식 PR 기반 작업 루틴

회사식 흐름에서는 `main`에 직접 커밋하거나 push하지 않는다. 먼저 최신 `main`에서 작업 브랜치를 만들고, 그 브랜치를 GitHub에 올린 뒤 PR로 병합한다.

```bash
git switch main
git pull --ff-only origin main
git switch -c docs/some-work

# 작업 후
git add -A
git commit -m "docs: update notes"
git push -u origin docs/some-work
```

PR을 만들고 squash merge까지 끝났다면 로컬을 다시 `main` 기준으로 맞춘다.

```bash
gh pr create --base main --head docs/some-work
gh pr merge --squash --delete-branch

git switch main
git pull --ff-only origin main
git branch -d docs/some-work
```

작업 브랜치를 만들기 전에 `main`에서 먼저 커밋해도, 아직 push하지 않았다면 나중에 브랜치를 만들 수는 있다.

```bash
git switch -c docs/some-work
```

하지만 권장 루틴은 아니다. PR을 squash merge하면 GitHub의 `main`에는 새 squash 커밋이 생기고, 로컬 `main`에는 원래 커밋이 남아 커밋 해시 기준으로 갈라질 수 있다.

```text
local main:   A -- B -- C
origin/main:  A -- B -- S
```

`C`와 `S`의 파일 내용이 같아도 Git은 다른 커밋으로 본다. 그래서 회사식 루틴에서는 작업 전에 브랜치를 먼저 만든다.

## squash merge 후 갈라진 `main` 정리

작업 전에 브랜치를 만들지 않아 로컬 `main`에 커밋이 남아 있으면, squash merge 후 이런 상태가 될 수 있다.

```text
## main...origin/main [ahead 1, behind 1]
```

이때 바로 pull, merge, rebase, force push를 하지 않는다. 먼저 양쪽 커밋과 파일 차이를 확인한다.

```bash
git log --oneline --graph --decorate --left-right main...origin/main
git diff --stat main origin/main
git diff --stat
git diff --cached --stat
```

판단 기준은 다음과 같다.

- `git diff --stat main origin/main`이 비어 있으면 두 브랜치의 최종 파일 내용은 같다.
- `git diff --stat`이 비어 있으면 unstaged 변경이 없다.
- `git diff --cached --stat`이 비어 있으면 staged 변경이 없다.
- 로컬 `main`에 보존해야 할 별도 커밋이 없으면 원격 `main`에 맞춰 정리할 수 있다.

이 조건이 모두 맞을 때만 로컬 `main`을 원격 `main`과 같게 맞춘다.

```bash
git reset --hard origin/main
```

`reset --hard`는 작업 파일과 staging area를 버릴 수 있는 위험한 명령이다. 상태와 diff가 깨끗하다는 것을 확인한 뒤에만 사용한다.

## submodule 작업 후 parent 포인터 PR

submodule에서 PR을 merge하면 parent 저장소는 submodule 폴더가 수정된 것으로 볼 수 있다. 이것은 submodule 내부 파일 변경이 아니라 parent가 기록하는 submodule 커밋 해시가 바뀐 것이다.

```bash
cd ~/code
git status --short --branch
git diff --submodule
```

예시는 다음과 같다.

```text
Submodule git-study 325725c..1e22cf2:
  > docs: expand multi-device sync routine (#1)
```

다른 컴퓨터가 parent 저장소를 받았을 때 같은 submodule 커밋을 보게 하려면, parent에서도 포인터 변경을 PR로 올린다.

```bash
git switch main
git pull --ff-only origin main
git switch -c chore/update-git-study-submodule

git add git-study
git commit -m "chore: update git-study submodule"
git push -u origin chore/update-git-study-submodule

gh pr create --base main --head chore/update-git-study-submodule
gh pr merge --squash --delete-branch
```

parent PR도 squash merge하면 로컬 `main`이 갈라질 수 있다. 이때도 위의 `log`, `diff`, `status` 확인 후에만 정리한다.

## 다른 컴퓨터에서 받는 루틴

다른 컴퓨터에서는 parent 저장소를 먼저 최신화하고, parent가 기록한 submodule 커밋으로 맞춘다.

```bash
cd ~/code
git switch main
git pull --ff-only origin main
git submodule sync --recursive
git submodule update --init --recursive
git submodule status
```

submodule 내용을 읽거나 빌드만 할 때는 여기까지로 충분하다. submodule 안에서 작업할 예정이면 해당 submodule을 브랜치에 올리고 최신화한다.

```bash
cd ~/code/git-study
git switch main
git pull --ff-only origin main
```

모든 submodule에서 작업할 가능성이 있으면 한 번에 확인할 수 있다.

```bash
cd ~/code
git submodule foreach 'git switch main'
git submodule foreach 'git pull --ff-only origin main'
git submodule foreach 'git status --short --branch'
```
