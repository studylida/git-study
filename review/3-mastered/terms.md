# 완벽히 암기한 용어/개념

## HEAD 참조

- **HEAD~n vs HEAD^n**: ~n은 "몇 세대 위"(HEAD~2 = 2세대 위), ^n은 "몇 번째 부모"(HEAD^2 = 두 번째 부모, 머지 커밋에서만 의미). 일반 커밋에서는 HEAD~1 = HEAD^.

## restore

- **git restore --staged <파일>**: 스테이징 영역에서만 제거. 파일 내용은 워킹 디렉토리에 그대로 남아있으므로 안전.

## cherry-pick

- **git cherry-pick <해시>**: 다른 브랜치의 특정 커밋을 현재 브랜치에 복사. 해시가 달라짐(부모 커밋이 다르니까).
- **git cherry-pick --continue**: 충돌 해결 후 계속 진행. 해시를 붙이면 안 됨!

## reset

- **reset --mixed**: reset의 기본값. HEAD + Staging을 되돌리고 Working Directory는 유지.

## fetch

- **fetch vs pull**: fetch는 안전하게 확인만, pull은 바로 병합까지. fetch 후 diff로 확인하고 merge하는 게 안전.

## 원격 추적 브랜치

- **origin/main**: 마지막으로 확인한 원격 저장소의 상태를 기억하는 읽기 전용 포인터. fetch, push, pull 시 업데이트됨. (merge만으로는 업데이트 안 됨!)

## interactive rebase 키워드

- **squash**: 이전 커밋과 합치기. **reword**: 커밋 메시지만 수정. **edit**: 커밋 내용(파일) 수정.

## PR 머지 방식

- **Merge commit** ← 이름 주의!: 모든 커밋 보존 + 머지 커밋 1개 생성. (5커밋 PR → 6개)
- **Squash and merge**: 모든 커밋을 1개로 합쳐서 머지. (5커밋 PR → 1개)
- **Rebase and merge**: 머지 커밋 없이 일직선으로 붙임. (5커밋 PR → 5개)

## fetch

- **git fetch**: 원격 변경사항을 원격 추적 브랜치(origin/main)에만 다운로드. 로컬 브랜치는 안 바뀜.
- **fetch는 스테이징이 아니라 원격 추적 브랜치를 업데이트**: fetch는 워킹 디렉토리나 스테이징에 아무 영향 없음. origin/main만 최신으로 갱신.

## diff

- **git diff가 비어있는 이유**: git add 후에는 Working Directory와 Staging이 같아져서 git diff가 비어있음. 이때는 git diff --staged로 Staging ↔ Repository 비교해야 변경사항이 보임.

## stash

- **git stash pop**: 가장 최근 stash를 꺼내고 목록에서 삭제. pop = apply + drop.
- **git stash**: 작업 중인 변경사항을 임시 저장. 워킹 디렉토리가 깨끗해져서 브랜치 전환 가능.

## git status

- **git status -s 읽는 법**: 왼쪽 칸=Staging 상태, 오른쪽 칸=Working Directory 상태. `M ` = staged, ` M` = unstaged, `??` = untracked.

## tag

- **tag는 별도로 push**: `git push`만으론 tag 안 올라감. `git push origin <tag>` 또는 `--tags` 필요.

## cherry-pick

- **git cherry-pick --no-commit (-n)**: 커밋하지 않고 스테이징에만 올림. 여러 커밋을 하나로 합칠 때 유용.

## restore (추가)

- **git restore <파일>**: 워킹 디렉토리의 변경사항을 마지막 커밋 상태로 되돌림. 수정 내용이 영구 삭제되므로 위험.

## 시맨틱 버전 관리 (추가)

- **상위 올리면 하위 리셋**: v1.3.7 → v1.4.0 (MINOR +1이면 PATCH=0).

## Fork (origin/upstream)

- **origin**: Fork 후 내 계정의 레포를 가리키는 remote. push 가능.
- **upstream**: 원본 레포를 가리키는 remote. 읽기 전용.

## tag (추가)

- **tag**: 특정 커밋에 붙이는 영구 이름표. 브랜치와 달리 움직이지 않는 고정 포인터.
- **lightweight tag**: 이름만 붙이는 태그. 메타데이터 없음. `git tag v0.1`
- **annotated tag**: 만든 사람, 날짜, 메시지 포함. `git tag -a v1.0 -m "메시지"`. 실무에서 사용.
- **git tag -a <tag> -m "msg" <해시>**: 과거 커밋에 뒤늦게 tag 붙이기.

## 시맨틱 버전 관리 (추가)

- **MAJOR.MINOR.PATCH**: v1.2.3 형식. MAJOR=호환 깨짐, MINOR=기능 추가, PATCH=버그 수정.

## reflog (추가)

- **reflog**: Reference Log. HEAD 이동 기록을 저장하는 로컬 전용 로그. 90일 보관, clone하면 없음.
- **git log vs git reflog**: log는 커밋 부모-자식 관계, reflog는 HEAD 이동 행동 기록. reset으로 날린 커밋도 reflog에는 남아있음.

## stash (추가)

- **git stash -u**: untracked(새로 만든) 파일도 포함해서 stash. 기본 stash는 tracked만 저장.
- **git stash drop stash@{n}**: 특정 stash 삭제.

## 원격 추적 브랜치 (추가)

- **git branch -vv**: 로컬 브랜치의 upstream 연결 상태와 ahead/behind 확인.
- **git branch -r**: 원격 추적 브랜치만 보기.

## Issue & Project (추가)

- **closes/fixes/resolves #번호**: PR 본문이나 커밋에 쓰면 머지 시 해당 이슈 자동 Close.

## stash (추가2)

- **git stash apply**: stash를 꺼내되 목록에 유지. 여러 브랜치에 같은 stash 적용할 때 사용.

## Fork (추가2)

- **Fork vs Clone**: Fork는 서버 간 복사(GitHub→GitHub), Clone은 서버→로컬 복사(GitHub→내 PC).

## Git Flow (추가)

- **--no-ff**: No Fast-Forward. 머지 커밋을 강제 생성해서 브랜치 분기/합류 히스토리를 보존.

## stash (추가3)

- **git stash -m "메시지"**: 설명을 달아서 저장. 여러 개 쌓일 때 구분용.

## Git Flow (추가2)

- **hotfix/***: main에서 분기 → main + develop 둘 다에 머지. 긴급 버그 수정용.

## GitHub Flow (추가)

- **장기 브랜치는 main 1개뿐**: develop, release, hotfix 브랜치 없음.

## 원격 추적 브랜치 (추가2)

- **upstream (-u)**: `git push -u origin main`으로 설정. 이후 `git push`만으로 push 가능.

## PR CLI (추가)

- **gh pr create --title "제목" --body "설명"**: CLI에서 PR 생성.

## git branch -D (추가)

- **git branch -D**: 강제 브랜치 삭제. `-d`는 커밋 해시로 HEAD에 포함 여부를 판단하므로, squash merge 등으로 해시가 달라지면 거부됨. 이때 `-D`로 강제 삭제.

## chmod +x (추가)

- **chmod +x .git/hooks/<hook이름>**: Hook 파일에 실행 권한 부여. 없으면 Hook 실행 안 됨.

## 시맨틱 버전 연결 (추가)

- **시맨틱 버전 연결**: fix→PATCH, feat→MINOR, BREAKING CHANGE→MAJOR.

## restore (추가2)

- **git restore .**: 현재 디렉토리의 모든 변경 파일을 한 번에 되돌림.

## pull (추가)

- **git pull**: fetch + merge. 원격 변경사항을 가져와서 바로 병합.

## submodule (추가)

- **submodule 삭제 3단계**: `deinit` → `rm -rf .git/modules/...` → `git rm`. 번거롭지만 이 순서대로.

## alias (추가)

- **! 접두사**: 셸 명령어를 alias로 만들 때 사용. `'!gitk'`처럼 앞에 붙임.

## restore (추가3)

- **git restore --source <커밋> <파일>**: 특정 커밋 시점의 파일 상태로 되돌림. 기본값은 HEAD.

## README & Markdown (추가)

- **마크다운 이미지 vs 링크**: `[텍스트](URL)` = 링크, `![대체텍스트](URL)` = 이미지. `!` 유무 차이.

## 원격 추적 브랜치 (추가3)

- **git branch -a**: 로컬 + 원격 추적 브랜치 모두 보기. `-a`는 `--all`의 줄임.

## PR CLI (추가2)

- **gh pr merge <번호>**: CLI에서 PR 머지. 현재 브랜치의 PR이면 번호 생략 가능.

## GitHub Actions (추가)

- **Job**: 독립된 작업 단위. 기본적으로 병렬 실행. 순서 강제는 `needs:` 사용.
