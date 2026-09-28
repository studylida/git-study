# Conventional Commits — 커밋 메시지 표준화

## 핵심 개념

Conventional Commits는 커밋 메시지를 일관된 형식으로 작성하는 규칙이다.
팀원 누구나 커밋 히스토리만 보고 **어떤 종류의 변경인지** 바로 파악할 수 있게 해준다.

### 왜 필요한가?

```
# 규칙 없을 때
fix bug
update
asdf
작업 완료

# Conventional Commits 적용 후
feat: 사용자 로그인 기능 추가
fix: 비밀번호 검증 오류 수정
docs: API 문서 업데이트
```

자동화 도구(CHANGELOG 생성, 시맨틱 버전 관리)와도 연결된다.

---

## 커밋 메시지 형식

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### 실제 예시

```
feat(auth): 소셜 로그인 기능 추가

Google, Kakao OAuth2 로그인을 지원합니다.
기존 이메일 로그인과 병행 가능합니다.

Closes #42
```

---

## 7가지 타입

| 타입 | 의미 | 예시 |
|------|------|------|
| `feat` | **새 기능** 추가 | `feat: 다크모드 추가` |
| `fix` | **버그** 수정 | `fix: 로그인 실패 시 크래시 수정` |
| `docs` | **문서**만 변경 | `docs: README 설치 방법 추가` |
| `style` | **코드 스타일** (포매팅, 세미콜론. 로직 변화 없음) | `style: 들여쓰기 통일` |
| `refactor` | **리팩토링** (기능 변화 없이 코드 구조 개선) | `refactor: 유저 서비스 클래스 분리` |
| `test` | **테스트** 추가/수정 | `test: 로그인 유닛테스트 추가` |
| `chore` | **기타** 잡일 (빌드, 설정, 의존성) | `chore: eslint 설정 업데이트` |

### 헷갈리기 쉬운 구분

```
feat     → 사용자가 새로운 걸 할 수 있게 됨
fix      → 잘못된 걸 고침
refactor → 내부 구조만 바꿈 (사용자 모름)
style    → 로직 안 건드림 (공백, 포매팅만)
docs     → 문서만 수정
chore    → 그 외 잡일 (빌드, 설정)
```

---

## 각 구성 요소 상세

### Scope (범위) — 선택사항

괄호 안에 어떤 부분을 건드렸는지 적는다:

```
feat(auth): 로그인 기능 추가
fix(database): 커넥션 풀 누수 수정
docs(api): 엔드포인트 문서 업데이트
```

- 팀에서 정한 규칙에 따라 사용
- 소문자로 작성
- **기능 영역**을 적는 게 일반적 (파일명보다는)

### Description (설명)

- 콜론 뒤에 **한 칸 공백** 후 시작
- **명령형**으로 작성 (영어: "add" / 한글: "추가")
- 첫 글자 소문자 (영어 기준)
- 마침표 안 붙임
- **50자 이내** 권장

```
# 좋은 예
feat: add user authentication
fix: 로그인 실패 시 에러 메시지 표시

# 나쁜 예
feat: Added user authentication.   ← 과거형, 마침표
fix: 로그인 실패 시 에러 메시지를 표시합니다.  ← 서술형
```

### Body (본문) — 선택사항

- 빈 줄 하나 띄우고 작성
- **왜** 이 변경을 했는지 설명

```
fix: 비밀번호 검증 오류 수정

기존에는 특수문자를 포함한 비밀번호가 정규식 매칭에서
실패하여 유효한 비밀번호도 거부되는 문제가 있었음.
정규식 패턴을 수정하여 해결.
```

### Footer (꼬리말) — 선택사항

```
feat: 결제 API 연동

Closes #123
Reviewed-by: Kim
```

---

## Breaking Change

기존 사용자의 코드가 깨질 수 있는 변경일 때 표시한다.

### 방법 1: 타입 뒤에 `!` 붙이기

```
feat!: API 응답 형식을 JSON에서 XML로 변경
```

### 방법 2: Footer에 명시

```
feat: API 응답 형식 변경

BREAKING CHANGE: 응답이 JSON에서 XML로 변경됩니다.
기존 클라이언트는 파서 업데이트가 필요합니다.
```

---

## 시맨틱 버전과의 연결

```
fix              → PATCH 올림  (v1.0.0 → v1.0.1)
feat             → MINOR 올림  (v1.0.0 → v1.1.0)
BREAKING CHANGE  → MAJOR 올림  (v1.0.0 → v2.0.0)
```

자동화 도구(semantic-release 등)가 이 규칙에 따라 버전을 자동으로 올려준다.

---

## 주의사항 & 흔한 실수

| 실수 | 올바른 형식 |
|------|------------|
| `Feat: 기능 추가` | `feat: 기능 추가` (타입은 소문자) |
| `docs : 문서 수정` | `docs: 문서 수정` (콜론 앞에 공백 없음) |
| `feat: Added login.` | `feat: add login` (명령형, 마침표 없음) |
| 설명이 너무 길게 한 줄 | title은 50자 이내, 나머지는 body에 |

---

## 실습 가이드

### 커밋 메시지 작성 연습

상황별로 어떤 타입을 쓸지 판단하는 연습:

```bash
# 새 기능 추가
git commit -m "feat(auth): 비밀번호 찾기 기능 추가"

# 버그 수정
git commit -m "fix: 로그인 실패 시 크래시 수정"

# 문서 수정
git commit -m "docs: README 설치 방법 업데이트"

# 리팩토링
git commit -m "refactor: 인증 로직 함수 분리"

# Breaking Change (body 포함)
git commit -m "feat!(api): API 응답 형식 변경

BREAKING CHANGE: result 필드가 data로 변경됩니다."
```

### 기존 프로젝트에 도입하기

1. 팀에서 사용할 타입 목록 합의
2. scope 규칙 정하기 (선택)
3. commit-msg hook으로 형식 강제 가능 (hooks에서 배운 것 활용!)
4. commitlint 같은 도구로 자동 검증
