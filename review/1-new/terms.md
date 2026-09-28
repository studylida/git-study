# 새로 배운 용어/개념

## git worktree

- **worktree**: 하나의 Git 저장소에 연결된 작업 폴더. 같은 객체와 브랜치 정보를 공유하면서 서로 다른 브랜치를 동시에 펼쳐 놓고 작업할 수 있다.
- **브랜치와 worktree의 차이**: 브랜치는 커밋을 가리키는 포인터이고 worktree는 브랜치의 파일을 펼쳐 놓은 폴더다.
- **git worktree add <경로> <브랜치>**: 기존 브랜치를 새 작업 폴더에 연결한다. 새 브랜치도 만들려면 `-b <브랜치>`를 함께 사용한다.
- **git worktree remove <경로>**: Git의 연결 정보를 정리하면서 worktree를 제거한다. 변경사항이 남아 있으면 일반 제거를 거부한다.
- **다른 worktree의 브랜치 표시**: `git branch`에서 `+`가 붙은 브랜치는 다른 worktree에서 체크아웃된 상태다.

## 명령어 옵션

- **짧은 옵션과 긴 옵션**: 짧은 옵션은 보통 `-u`처럼 하이픈 하나와 한 글자를 사용하고, 긴 옵션은 `--set-upstream`처럼 하이픈 두 개와 단어를 사용한다. 지원 여부는 명령마다 다르므로 `--help`에서 확인한다.

## 브랜치 보호 규칙

- **Ruleset**: GitHub의 브랜치 보호 규칙 시스템. Settings → Rules → Rulesets에서 설정.
- **Require a pull request before merging**: main에 직접 push 차단. 반드시 PR을 통해 머지해야 함.
- **Require approvals**: PR 머지 전 리뷰어 승인 필수. 혼자 작업 시 0으로 설정 가능.
- **Block force pushes**: `git push --force`도 차단.
- **Restrict deletions**: 보호된 브랜치 삭제 금지.
- **gh ruleset list**: CLI에서 적용된 ruleset 목록 확인.
- **gh ruleset view <ID>**: 특정 ruleset 상세 내용 확인.
- **git fetch --prune**: 원격에서 삭제된 브랜치의 추적 정보를 로컬에서 정리.

## Issue & Project

- **Issue**: GitHub에서 할 일을 기록하는 티켓. 버그 리포트, 기능 요청, 질문 등에 사용.
- **Label**: 이슈 분류 태그. bug, enhancement, documentation 등 기본 제공.
- **Assignee**: 이슈 담당자 지정. `gh issue edit --add-assignee @me`로 설정.
- **Milestone**: 여러 이슈를 하나의 목표(버전/기한)로 묶는 기능. 진행률 자동 계산.
- **Project (V2)**: 칸반 보드로 이슈를 시각적으로 관리 (Todo → In Progress → Done).
- **gh issue create**: `--title`, `--body`, `--label` 옵션으로 CLI에서 이슈 생성.
- **gh issue list**: 열린 이슈 목록. `--state all`로 닫힌 것도 포함.
- **gh issue close <번호>**: 이슈 닫기. `--reason completed` 또는 `--reason "not planned"`.
- **gh issue edit <번호>**: `--add-assignee`, `--add-label` 등으로 이슈 수정.

## README & Markdown

- **뱃지**: shields.io 같은 서비스가 상태를 실시간 이미지 URL로 제공. GitHub Actions 연결 시 빌드 결과 자동 반영.
- **GitHub 전용 문법**: `@username` 멘션, `#123` 이슈/PR 자동 링크, `- [ ]` / `- [x]` 체크박스.
- **잘 만든 README 구성**: 프로젝트 설명 → 설치 → 사용법 → 기여 → 라이센스.

## Fork

- **git remote add upstream <URL>**: 원본 레포를 upstream으로 수동 등록.
- **gh repo fork <레포> --clone=true**: Fork + Clone + upstream 설정을 한 번에.
- **upstream 동기화**: `git fetch upstream` → `git merge upstream/main` → `git push origin main`. 안 하면 PR 충돌 가능.
- **오픈소스 기여 흐름**: Fork → Clone → 브랜치 → 작업 → push(origin) → 원본에 PR.

## stash

- **git stash list**: 저장된 stash 목록 보기. stash@{0}이 가장 최근.
- **git stash clear**: 모든 stash 삭제.
- **git stash -m "메시지"**: 설명을 달아서 저장. 여러 개 쌓일 때 구분용.

## tag

- **git tag -l "패턴"**: 패턴으로 tag 필터링. -l 없이 패턴 쓰면 tag 생성 시도됨.
- **git show <tag>**: tag 상세 정보. annotated는 Tagger/Date/메시지 나오고, lightweight는 커밋 정보만.
- **git push origin --delete <tag>**: 원격 tag 삭제. 로컬은 `git tag -d`로 따로 삭제.

## 시맨틱 버전 관리

- **pre-release 순서**: alpha(불안정) → beta(테스트) → rc(출시후보) → 정식.
- **rc**: Release Candidate. 정식 출시 직전 최종 테스트 버전.
- **v0.x.x**: 초기 개발 단계. 아직 정식 아님, 언제든 바뀔 수 있음.
- **gh release create**: `gh release create v1.0.0 --title "v1.0.0" --notes "설명"`. tag 기반으로 GitHub Release 생성.

## reflog

- **git reflog**: HEAD의 이동 기록 보기. 모든 행동(commit, reset, checkout, rebase 등) 기록됨.
- **git reflog show <브랜치>**: 특정 브랜치의 이동 기록만 보기.
- **HEAD@{n}**: n번째 전 행동. `git reset --hard HEAD@{2}`처럼 복구에 사용.
- **커밋 복구**: reflog에서 해시 찾고 `git reset --hard <해시>`.
- **브랜치 복구**: reflog에서 해시 찾고 `git branch <이름> <해시>`.

## hooks

- **Hook**: Git 이벤트 시 자동 실행되는 셸 스크립트. `.git/hooks/`에 저장.
- **pre-commit**: 커밋 직전 실행. 코드 검사, 린트 등. exit 1로 커밋 차단 가능.
- **commit-msg**: 커밋 메시지 작성 후 실행. 메시지 형식 강제. `$1`로 메시지 파일 경로 받음.
- **post-commit**: 커밋 완료 후 실행. 알림용. 차단 불가.
- **pre-push**: push 직전 실행. 테스트, 브랜치 체크. exit 1로 push 차단 가능.
- **exit 0 / exit 1**: 0=통과(계속 진행), 1=실패(작업 차단).
- **Hook은 로컬 전용**: `.git/hooks/`는 push 안 됨. 팀원과 자동 공유 불가.
- **.sample 확장자**: 붙어있으면 비활성화. 제거하면 활성화.

## submodule

- **submodule**: 내 저장소 안에 다른 저장소를 포함. 파일이 아닌 커밋 해시(포인터)만 기록.
- **git submodule add <URL> <경로>**: submodule 추가. 경로는 자유롭게 설정 가능.
- **.gitmodules**: submodule 설정 파일. URL과 경로가 기록됨.
- **git clone --recurse-submodules**: submodule 포함 clone. 안 쓰면 submodule 폴더가 비어있음.
- **git submodule init + update**: clone 후 빈 submodule 초기화. init은 등록, update는 파일 다운로드.
- **git submodule update --remote**: 원격 최신 커밋으로 업데이트. (그냥 update는 메인이 기록한 커밋으로 맞추는 것)
- **git submodule status**: 상태 확인. 앞 기호: 공백=정상, `-`=init 안 됨, `+`=커밋이 다름.
- **submodule 업데이트 2단계**: submodule 안에서 pull(파일 가져옴) → 메인에서 커밋(포인터 기록). 둘 다 해야 완성.
- **submodule 폴더는 독립된 Git 저장소**: clone으로 만들어졌으므로 자기만의 remote, log, branch가 별도로 존재.
- **submodule pointer**: 부모 저장소가 submodule의 파일 전체가 아니라 특정 커밋 해시를 기록하는 것.
- **detached HEAD in submodule**: `git submodule update` 후 submodule이 브랜치가 아니라 특정 커밋에 직접 checkout된 상태. 읽기만 할 때는 괜찮지만, 작업하려면 `git switch main`으로 브랜치에 올라타는 것이 안전하다.
- **git submodule sync --recursive**: 부모 저장소의 `.gitmodules`에 적힌 URL/설정을 로컬 Git 설정에 반영한다.
- **git submodule update --init --recursive**: submodule이 없으면 초기화하고, 있으면 부모 저장소가 기록한 커밋으로 맞춘다.
- **git submodule status의 `+`**: 현재 submodule 커밋이 부모가 기록한 커밋과 다르다는 뜻. 앞선 커밋이어도, 뒤처진 커밋이어도 다르면 `+`가 붙는다.
- **git submodule status의 `U`**: merge 중 submodule 포인터가 충돌난 상태. 두 브랜치가 같은 submodule을 서로 다른 커밋으로 기록했을 때 발생할 수 있다.
- **fetch vs pull**: `fetch`는 원격 정보를 가져오지만 현재 브랜치와 작업 파일은 바꾸지 않는다. `pull --ff-only`는 원격을 다시 확인한 뒤 fast-forward 가능한 경우에만 현재 브랜치를 이동한다.
- **SSH 인증**: 로컬 개인키와 GitHub에 등록된 공개키가 짝이 맞는지 확인해서 `git pull`, `git push` 권한을 증명하는 방식.
- **submodule pointer PR**: submodule에서 새 커밋이 merge된 뒤, parent 저장소가 기록하는 submodule 커밋 해시 변경을 별도 PR로 올리는 작업.

## PR 기반 작업 루틴

- **작업 브랜치**: `main`에서 직접 커밋하지 않기 위해 작업 전에 만드는 브랜치. 예: `docs/multi-device-sync-routine`.
- **작업 전 브랜치 생성**: 회사식 표준 루틴. `main` 최신화 후 `git switch -c <branch>`로 시작하면 squash merge 후 로컬 `main` 정리가 쉬워진다.
- **커밋 후 브랜치 생성**: 아직 push하지 않았다면 가능하지만 권장 루틴은 아니다. 로컬 `main`에 커밋이 남아 squash merge 후 원격 `main`과 갈라질 수 있다.
- **squash merge**: PR의 여러 커밋을 하나의 새 커밋으로 합쳐 `main`에 넣는 방식. 원래 브랜치 커밋과 해시가 달라진다.
- **diverged branches**: 로컬과 원격이 서로 다른 커밋을 가진 상태. `ahead 1, behind 1`처럼 표시된다.
- **left-right log**: `git log --left-right main...origin/main`으로 어느 쪽에만 있는 커밋인지 확인하는 방법. `<`는 왼쪽, `>`는 오른쪽 기준 커밋이다.
- **remote branch deletion**: PR merge 후 원격 작업 브랜치를 삭제하는 정리 작업. `git push origin --delete <branch>` 또는 `gh pr merge --delete-branch`로 처리.
- **gh pr merge --squash --delete-branch**: 현재 PR을 squash merge하고 작업 브랜치까지 삭제하는 GitHub CLI 명령.
- **git reset --hard origin/main**: 로컬 브랜치를 원격 브랜치와 정확히 같게 맞추는 위험 명령. working tree와 staging area가 깨끗하고, 보존할 로컬 커밋이 없을 때만 사용.

## alias

- **git config --global alias.<단축키> '<명령어>'**: alias 등록. `git` 뒤 부분만 적음.
- **git config --global --get-regexp alias**: 등록된 alias 전체 목록 확인.
- **git config --global --unset alias.<단축키>**: alias 삭제.
- **~/.gitconfig**: `--global` alias가 저장되는 파일. `[alias]` 섹션에 기록됨.
- **.git/config**: `--local` (기본) alias가 저장되는 파일. 해당 저장소에서만 사용.
- **git config --global --edit**: 설정 파일을 직접 편집기로 열기.

## Git Flow

- **Git Flow**: Vincent Driessen이 제안한 브랜치 전략. main, develop, feature, release, hotfix 5가지 브랜치 사용.
- **main**: 항상 배포 가능한 코드. 직접 커밋 금지. 머지 시 태그 붙임.
- **develop**: 다음 릴리스 준비용 통합 브랜치. feature가 여기로 머지됨.
- **feature/***: develop에서 분기 → develop으로 머지. 새 기능 개발용.
- **release/***: develop에서 분기 → main + develop 둘 다에 머지. 출시 마무리용.
- **hotfix/***: main에서 분기 → main + develop 둘 다에 머지. 긴급 버그 수정용.

## GitHub Flow

- **GitHub Flow**: main + feature 브랜치만 쓰는 간소화된 전략. GitHub 자체가 사용하는 방식.
- **장기 브랜치는 main 1개뿐**: develop, release, hotfix 브랜치 없음. feature는 만들고 머지하면 삭제.
- **6가지 규칙**: main 항상 배포 가능, main에서 바로 브랜치, 설명적 이름, 수시 push, PR 리뷰 후 머지, 머지 즉시 배포.
- **핫픽스 처리**: 별도 규칙 없이 feature 브랜치와 동일하게 처리. 빨리 리뷰하고 머지하면 그게 핫픽스.
- **적합한 환경**: 웹앱, SaaS, 수시 배포, 소규모 팀. 릴리스 주기가 있는 앱은 Git Flow가 적합.

## Conventional Commits

- **Conventional Commits**: 커밋 메시지 표준화 규칙. `<type>[scope]: <description>` 형식.
- **7가지 타입**: feat(새 기능), fix(버그 수정), docs(문서), style(포매팅), refactor(구조 개선), test(테스트), chore(잡일).
- **scope**: 선택사항. 괄호 안에 기능 영역을 소문자로. `feat(auth): 로그인 추가`.
- **description 규칙**: 콜론+공백 후 시작, 명령형, 소문자, 마침표 없음, 50자 이내.
- **body**: 빈 줄 후 작성. "왜" 이 변경을 했는지 설명.
- **footer**: `Closes #123` 등 이슈 참조, 리뷰어 정보.
- **콜론 앞 공백 금지**: `docs : 수정` ❌ → `docs: 수정` ✅. 자동화 도구가 파싱 실패함.

## GitHub Actions

- **Workflow**: 자동화 전체 프로세스. `.github/workflows/` 안의 `.yml` 파일 1개 = 워크플로우 1개.
- **Event (on)**: 워크플로우를 실행시키는 트리거. push, pull_request, workflow_dispatch 등.
- **Step**: Job 안에서 순서대로 실행되는 각 단계.
- **Runner (runs-on)**: 워크플로우가 실행되는 가상 서버. ubuntu-latest, windows-latest, macos-latest.
- **uses**: 마켓플레이스의 Action 가져다 쓰기. `uses: actions/checkout@v4`.
- **run**: 셸 명령어 직접 실행. `run: echo "hello"`. 여러 줄은 `run: |` 사용.
- **actions/checkout@v4**: 저장소 코드를 Runner에 복사(clone)하는 필수 Action. 없으면 파일 관련 작업 불가.
- **workflow_dispatch**: 수동 실행 트리거. Actions 탭에서 "Run workflow" 버튼 생성.

## 자동 테스트

- **on: pull_request**: PR 이벤트에 반응하는 트리거. PR 생성/커밋 추가 시 워크플로우 실행.
- **Status Check**: 워크플로우 실행 결과가 PR 페이지에 표시되는 것. Job 이름이 Check 이름이 됨.
- **Required Status Check**: Ruleset에서 "Require status checks to pass" 설정. 테스트 실패 시 머지 차단.
- **npm ci**: `npm install`보다 빠르고 정확한 의존성 설치. CI 환경에서 사용.
- **actions/setup-node@v4**: Node.js 환경을 Runner에 설치하는 Action. `with: node-version`으로 버전 지정.

## 자동 배포

- **배포 트리거**: `on: push, branches: [main]`. PR 머지 시 main에 push가 발생하므로 자동 배포됨.
- **Secrets**: GitHub에서 민감한 정보를 암호화 저장. 한번 저장하면 다시 볼 수 없음. 로그에 `***`로 마스킹.
- **${{ secrets.NAME }}**: 워크플로우에서 Secrets 값을 참조하는 문법.
- **${{ vars.NAME }}**: Variables 값을 참조하는 문법. Secrets와 달리 평문이고 다시 볼 수 있음.
- **Environment**: 배포 대상 구분 (staging, production). 환경별 secrets, 보호 규칙, 배포 이력 제공.
- **permissions**: 워크플로우에 필요한 권한 명시. `pages: write`, `id-token: write` 등.
- **CI vs CD**: CI(Continuous Integration)=PR마다 테스트, CD(Continuous Deployment)=main 머지 시 자동 배포.
- **actions/upload-pages-artifact@v3**: 배포할 파일을 artifact로 업로드하는 Action.
- **actions/deploy-pages@v4**: GitHub Pages에 실제 배포하는 Action.

## 린트/포매팅

- **Lint (린트)**: 코드 품질 검사. 버그, 안 쓰는 변수, == 대신 === 등. 대표 도구: ESLint.
- **Formatting (포매팅)**: 코드 모양 통일. 들여쓰기, 따옴표, 세미콜론 등. 대표 도구: Prettier.
- **ESLint**: JS/TS 린터. `eslint.config.js`로 규칙 설정. `npx eslint src/`로 실행.
- **Prettier**: 코드 포매터. `.prettierrc`로 스타일 설정. `--check`(검사만), `--write`(수정).
- **eslint-config-prettier**: ESLint와 Prettier 충돌 방지. Prettier와 겹치는 ESLint 규칙 비활성화.
- **globals**: ESLint에 실행 환경 알려주는 패키지. `globals.node`(Node.js), `globals.browser`(브라우저).
- **languageOptions.globals**: eslint.config.js에서 전역 변수(console, process 등) 허용 설정.
- **규칙 레벨**: `"off"`(무시), `"warn"`(경고, CI 통과), `"error"`(에러, CI 실패).
- **npx eslint --fix**: 자동 수정 가능한 린트 문제를 고쳐줌.
- **"type": "module"**: package.json에 추가. eslint.config.js에서 import 문법 쓰려면 필요.

## Git 객체

- **commit 객체 구성**: tree(스냅샷) + parent(이전 커밋) + author/committer + message. 4가지 정보를 담고 있음.
- **git cat-file -p <해시>**: 객체의 내용을 pretty print. HEAD, 해시값 등 사용 가능. 경로(파일/폴더명)는 불가.
- **100644 / 040000 / 100755**: tree에서 쓰는 파일 모드. 각각 일반 파일, 디렉토리, 실행 파일.
- **같은 내용 = 같은 blob**: 파일 이름을 바꿔도 내용이 같으면 blob은 재사용. tree만 변경됨.

## 해시 함수

- **해시 함수**: 입력 데이터를 고정 길이 문자열로 변환하는 함수. 단방향, 결정적, 눈사태 효과.
- **SHA-1**: Git의 기본 해시 알고리즘. 160비트(40자리 16진수) 출력.
- **해시 계산 공식**: `SHA-1("{타입} {크기}\0{내용}")`. blob이면 `blob 5\0hello` 형태.
- **git hash-object --stdin**: 표준 입력의 SHA-1 해시를 계산. `echo -n "내용" |`과 함께 사용.
- **해시 체인**: 커밋 안에 parent 해시가 포함 → 과거 수정 시 후속 커밋 해시가 연쇄 변경 → 조작 불가.
- **눈사태 효과**: 입력이 1글자만 달라도 해시값이 완전히 달라지는 특성.
- **SHA-256**: SHA-1 후속. 256비트(64자리). `git init --object-format=sha256`으로 실험적 지원.
- **규칙 레벨 (ESLint)**: `"off"`(무시), `"warn"`(경고, CI 통과), `"error"`(에러, CI 실패).

## .git 폴더

- **HEAD 파일**: `.git/HEAD`. 현재 체크아웃된 브랜치를 가리킴. `ref: refs/heads/main` 형태. Detached HEAD면 해시값 직접 저장.
- **refs/heads/**: 로컬 브랜치 저장 폴더. 브랜치 1개 = 커밋 해시가 적힌 파일 1개.
- **refs/tags/**: 태그 저장 폴더. 구조는 브랜치와 동일.
- **refs/remotes/origin/**: 원격 추적 브랜치 저장 폴더. fetch/push 시 업데이트.
- **objects/ 폴더 구조**: 해시 앞 2자리 = 폴더명, 나머지 38자리 = 파일명. 256개 폴더에 분산해서 파일시스템 성능 유지.
- **pack 파일**: 객체가 많아지면 loose object를 하나의 packfile로 압축. `.git/objects/pack/`에 저장.
- **index**: `.git/index`. Staging Area의 실체. 바이너리 파일. `git ls-files --stage`로 확인.
- **.git/config**: 로컬 설정 파일. remote URL, branch 추적 정보 등 저장.
- **packed-refs**: refs를 하나의 파일로 합쳐 최적화. refs/ 파일이 있으면 그쪽이 우선.
- **fetch 매핑 규칙**: `+refs/heads/*:refs/remotes/origin/*`. 원격 브랜치를 로컬 원격 추적 브랜치로 복사하는 규칙.
