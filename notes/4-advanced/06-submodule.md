# Submodule — 저장소 안의 저장소

## 핵심 개념

### submodule이란?
- 내 Git 저장소 안에 **다른 Git 저장소를 포함**시키는 기능
- 파일을 복사하는 게 아니라, **"이 폴더는 저 저장소의 특정 커밋을 가리킨다"**는 포인터를 기록
- 비유: 바로가기 링크. 실제 파일이 아니라 참조

### 메인 저장소가 기록하는 것
- submodule의 **파일 내용이 아니라 커밋 해시(포인터)**만 저장
- 원본에 새 커밋이 생겨도 메인은 자동으로 안 바뀜 → 의도적으로 업데이트해야 함
- 장점: 팀원 모두 같은 버전을 사용하게 됨

### submodule 추가 시 생기는 것
```
my-project/
├── .gitmodules        ← submodule 설정 (URL, 경로)
└── libs/shared-lib/   ← 실제 submodule 폴더 (독립된 Git 저장소)
```

### ⚠️ submodule 폴더는 독립된 Git 저장소
- submodule을 추가하면 내부적으로 `git clone`이 일어남
- 따라서 submodule 폴더 안에는 자기만의 `.git`, `remote`, `branch`, `log`가 있음
- submodule 안에서 `git log`를 치면 → **submodule 저장소의 로그**가 나옴
- 메인 프로젝트에서 `git log`를 치면 → **메인 프로젝트의 로그**가 나옴

#### 예시로 이해하기
```
/tmp/
├── shared-lib/              ← 원본 저장소 (A)
└── my-app/                  ← 메인 프로젝트 (B)
    └── vendor/shared-lib/   ← A를 clone한 것 (C)
```

- `git submodule add`를 하면 A를 clone해서 C가 만들어짐
- C는 A를 clone한 거니까, C의 origin = A
- C 안에서 `git log`를 치면:
  ```
  be7ef5a (HEAD -> main, origin/main, origin/HEAD)
  ```
  - `HEAD -> main` → C 자신의 현재 위치
  - `origin/main` → 원본 A의 main을 마지막으로 fetch한 위치
  - `origin/HEAD` → 원본 A의 기본 브랜치 (보통 origin/main과 같은 곳)
- B(메인 프로젝트)에서 `git log`를 치면 → B 자체의 커밋만 보임 (C의 로그와 완전 별개)

#### 왜 셋 다 같은 커밋에 있어?
```
be7ef5a (HEAD -> main, origin/main, origin/HEAD)
```
- `HEAD -> main` → C의 HEAD가 main을 가리키고, main이 be7ef5a에 있다
- `origin/main` → A(원본)의 main도 be7ef5a에 있다
- `origin/HEAD` → A(원본)의 기본 브랜치도 be7ef5a에 있다
- **방금 clone(또는 pull)해서 최신 상태로 맞춰졌으니까 당연히 같은 위치**

만약 A(원본)에서 새 커밋을 만들면:
```
be7ef5a (HEAD -> main)              ← C는 여기에 멈춰있음
    ↓
fff1234                             ← A는 여기로 이동

C에서 git fetch 하면:
be7ef5a (HEAD -> main)              ← C는 여기
    ↓
fff1234 (origin/main, origin/HEAD)  ← origin이 앞서감
```
이전에 배운 fetch 개념이랑 완전히 같은 원리!

#### 비유
```
내 집(B) 안에 세입자(C)가 살고 있다.
세입자(C)는 자기 고향(A)이 따로 있다. ← 이게 origin
내 집(B)의 주소와 세입자의 고향 주소는 전혀 다른 것.
B에서 git log → 내 집 이력
C에서 git log → 세입자의 이력 (고향 정보 포함)
```

---

## 주요 명령어

### submodule 추가
```bash
git submodule add <URL> <경로>
# 경로는 자유롭게 설정 가능 (libs/, vendor/, third_party/ 등)
```

### submodule 포함 clone
```bash
# 방법 1: 한 번에 (실무에서 이걸 씀)
git clone --recurse-submodules <URL>

# 방법 2: clone 후 초기화
git clone <URL>
git submodule init       # .gitmodules → .git/config에 등록
git submodule update     # 실제 파일 다운로드
```
⚠️ 그냥 `git clone`만 하면 submodule 폴더가 **비어있음!**

### submodule 업데이트 (원본에 새 커밋이 생겼을 때)

#### 상황
```
A(원본):      172d499 → 8331f15 (새 커밋 생김!)
C(submodule): 172d499             ← 여기에 멈춰있음
B(메인):      "C는 172d499를 쓰고 있다"
```

#### 1단계: C 안에서 pull (파일을 실제로 가져옴)
```bash
cd vendor/shared-lib
git pull origin main     # A에서 최신 가져오기
```
```
C: 172d499 → 8331f15 (HEAD -> main)  ← C가 최신으로 이동!
B: "C는 172d499를 쓰고 있다"          ← 아직 옛날 기록 그대로
```
**C의 파일은 업데이트됐지만, B는 아직 모르는 상태!**

#### 2단계: B에서 커밋 (포인터를 업데이트)
```bash
cd ../..
git status
# → "수정함: vendor/shared-lib (새 커밋)" ← B가 눈치챔!

git add vendor/shared-lib
git commit -m "chore: shared-lib 업데이트"
```
```
C: 172d499 → 8331f15 (HEAD -> main)
B: "C는 8331f15를 쓰고 있다"  ← 포인터 업데이트 완료!
```

#### 왜 2번 해야 해?
| 단계 | 뭘 하는 거야 | 안 하면? |
|------|-------------|---------|
| C에서 pull | 실제 파일을 가져옴 | 파일이 안 바뀜 |
| B에서 커밋 | "이 버전 쓴다"를 기록 | 팀원이 clone하면 옛날 버전 받음 |

**pull은 파일을 가져오는 거, 메인 커밋은 "이 버전 쓴다"를 팀에 알리는 거. 둘 다 해야 완성!**

### submodule 상태 확인
```bash
git submodule status
#  abc1234 vendor/shared-lib     → 정상 (앞에 공백)
# -abc1234 vendor/shared-lib     → init 안 됨
# +abc1234 vendor/shared-lib     → 기록된 커밋과 다름
```

### submodule 삭제 (3단계)
```bash
git submodule deinit <경로>           # 1. 등록 해제
rm -rf .git/modules/<경로>            # 2. .git/modules에서 제거
git rm <경로>                         # 3. 파일 제거 + staging
git commit -m "chore: submodule 제거"
```

---

## Git 2.38+ 로컬 파일 프로토콜 보안

### 무슨 일이 있었나?
- 2022년에 Git의 보안 취약점 발견
- Git 2.38부터 **로컬 파일 경로(`file://`)를 통한 clone/fetch가 기본 차단**됨
- submodule add, submodule update 등에서 로컬 경로를 쓰면 이 에러가 남:
  ```
  fatal: transport 'file' not allowed
  ```

### 해결 방법
```bash
# 해당 명령어에만 일시적으로 허용 (안전)
git -c protocol.file.allow=always submodule add /tmp/shared-lib libs/shared-lib
git -c protocol.file.allow=always submodule update --remote
```

### 실무에서는 걱정 없음
실무에서 submodule은 거의 **HTTPS나 SSH URL**을 쓰기 때문에 이 보안 정책에 걸리지 않음:
```bash
# 실무 (정상 작동)
git submodule add https://github.com/someone/lib.git libs/lib
git submodule add git@github.com:someone/lib.git libs/lib

# 로컬 실습 (차단됨)
git submodule add /tmp/shared-lib libs/shared-lib
```
우리는 실습 환경이라 로컬 경로를 써서 걸린 것.

---

## 실습에서 배운 것
- submodule add → `.gitmodules`와 submodule 폴더가 staging에 올라감
- 원본에 새 커밋 생겨도 메인의 submodule에는 자동 반영 안 됨 (포인터 고정)
- `git submodule update`는 "메인이 기록한 커밋으로 맞추는 것"이지, 최신으로 가져오는 게 아님
- `--remote` 옵션을 붙여야 원격 최신으로 업데이트
