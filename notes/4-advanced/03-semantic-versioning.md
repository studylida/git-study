# 시맨틱 버전 관리 (Semantic Versioning)

## 기본 구조

```
v MAJOR . MINOR . PATCH
  주요     부가    패치

예: v1.2.3
    │ │ └─ PATCH: 버그 수정
    │ └─── MINOR: 기능 추가 (하위 호환 O)
    └───── MAJOR: 대규모 변경 (하위 호환 X)
```

## 각 숫자의 의미

| | 언제 올려? | 하위 호환 | 예시 |
|---|---|---|---|
| **MAJOR** | 기존 코드가 깨지는 변경 | ❌ | API 파라미터 변경, 함수 삭제 |
| **MINOR** | 새 기능 추가 | ✅ | 새 함수, 새 옵션 추가 |
| **PATCH** | 버그 수정 | ✅ | 오타, 버그 패치 |

## 핵심 규칙

- **상위 숫자가 올라가면 하위는 0으로 리셋**
- v1.3.7 → 버그 수정 → v1.3.8
- v1.3.8 → 기능 추가 → v1.4.0
- v1.4.0 → 호환 깨짐 → v2.0.0

## pre-release 버전

```
alpha → beta → rc → 정식
(불안정)  (테스트)  (출시후보)  (릴리즈)
```

- **alpha**: 개발 초기, 불안정
- **beta**: 기능 완성, 테스트 중
- **rc** (Release Candidate): 정식 출시 후보, 거의 완성

예: `v1.0.0-alpha.1` → `v1.0.0-beta.1` → `v1.0.0-rc.1` → `v1.0.0`

## v0.x.x

- MAJOR가 0 = 아직 정식 아님, 언제든 바뀔 수 있음
- v1.0.0을 찍는 순간 = "안정판" 선언

## 실무 패턴

```bash
git tag -a v1.0.0 -m "첫 정식 릴리즈"
git push origin v1.0.0
gh release create v1.0.0 --title "v1.0.0" --notes "첫 정식 릴리즈"
```

---

## 실습 가이드: 버전 관리 시뮬레이션

### 준비

```bash
mkdir ~/git-semver-practice && cd ~/git-semver-practice
git init

echo "v1.0.0 - 초기 버전" > CHANGELOG.md
echo "print('hello')" > app.py
git add . && git commit -m "init: 프로젝트 시작"
git tag -a v1.0.0 -m "첫 정식 릴리즈"
```

### Step 1: PATCH 버전 올리기 (버그 수정)

```bash
# 버그 수정
echo "print('hello world')" > app.py   # 오타 수정
echo -e "\n## v1.0.1\n- 오타 수정" >> CHANGELOG.md
git add . && git commit -m "fix: hello 오타 수정"
git tag -a v1.0.1 -m "오타 수정 패치"

# 확인
git tag
# → v1.0.0
# → v1.0.1
```

### Step 2: MINOR 버전 올리기 (기능 추가)

```bash
# 새 기능 추가 (기존 코드와 호환됨)
echo "def greet(name): print(f'hello {name}')" >> app.py
echo -e "\n## v1.1.0\n- greet 함수 추가" >> CHANGELOG.md
git add . && git commit -m "feat: greet 함수 추가"
git tag -a v1.1.0 -m "greet 기능 추가"
# → MINOR가 올라가면 PATCH는 0으로 리셋!
```

### Step 3: MAJOR 버전 올리기 (호환 깨짐)

```bash
# API 변경 (기존 코드가 깨짐)
echo "def greet(name, lang='ko'): print(f'안녕 {name}')" > app.py
echo -e "\n## v2.0.0\n- greet API 변경 (lang 파라미터 필수)" >> CHANGELOG.md
git add . && git commit -m "feat!: greet API 변경 (BREAKING CHANGE)"
git tag -a v2.0.0 -m "API 대규모 변경 - 하위 호환 안 됨"
# → MAJOR가 올라가면 MINOR, PATCH 모두 0으로 리셋!
```

### Step 4: pre-release 태그

```bash
# 다음 버전 개발 중
echo "# 새 기능 개발 중" >> app.py
git add . && git commit -m "feat: 새 기능 개발 시작"
git tag -a v2.1.0-alpha.1 -m "v2.1 알파 테스트"

# 안정화
git add . && git commit --allow-empty -m "fix: 알파 버그 수정"
git tag -a v2.1.0-beta.1 -m "v2.1 베타 테스트"

git add . && git commit --allow-empty -m "chore: 출시 준비"
git tag -a v2.1.0-rc.1 -m "v2.1 릴리즈 후보"

# 정식 출시
git tag -a v2.1.0 -m "v2.1 정식 릴리즈"

# 전체 태그 확인
git tag
# → alpha → beta → rc → 정식 순서 확인
```

### 정리

```bash
rm -rf ~/git-semver-practice
```

---

## 주의사항 & 흔한 실수

- **MAJOR 0.x.x는 "아직 불안정"이라는 의미**. 정식 API가 아니므로 언제든 바뀔 수 있음
- 버전을 건너뛰지 말 것: v1.0.0 → v1.0.2 (v1.0.1은 어디 갔지?) → 혼란 유발
- CHANGELOG.md를 태그와 함께 관리하면 **어떤 버전에서 뭐가 바뀌었는지** 추적 가능
- **하위 호환이 깨지는지 판단하는 기준**: 기존 사용자의 코드가 수정 없이 동작하면 MINOR, 수정이 필요하면 MAJOR
