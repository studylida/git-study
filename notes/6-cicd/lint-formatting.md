# 린트/포매팅 — 코드 품질 자동 검사

## 린트(Lint) vs 포매팅(Formatting)

```
┌─────────────────────────────────────────────────────┐
│                    코드 품질 검사                      │
├──────────────────────┬──────────────────────────────┤
│     Lint (린트)       │     Formatting (포매팅)       │
├──────────────────────┼──────────────────────────────┤
│ "이 코드 문제있어!"    │  "이 코드 못생겼어!"           │
│                      │                              │
│ • 사용 안 하는 변수    │  • 들여쓰기 (탭 vs 스페이스)   │
│ • == 대신 === 써라    │  • 따옴표 (작은 vs 큰)         │
│ • 정의 안 된 변수 사용  │  • 세미콜론 유무              │
│ • console.log 남아있음 │  • 줄 길이, 줄바꿈            │
│                      │                              │
│ 대표 도구: ESLint     │  대표 도구: Prettier           │
├──────────────────────┼──────────────────────────────┤
│ 코드의 "품질"         │  코드의 "모양"                 │
└──────────────────────┴──────────────────────────────┘
```

- **린트** = 맞춤법 검사기 (틀린 거 잡아줌)
- **포매팅** = 글 정렬 도구 (예쁘게 다듬어줌)

---

## ESLint — JavaScript/TypeScript 린터

### 설치 및 실행

```bash
# 설치
npm install --save-dev eslint @eslint/js

# 실행
npx eslint .           # 현재 폴더 전체 검사
npx eslint src/        # src 폴더만 검사
npx eslint --fix .     # 자동 수정 가능한 건 고쳐줌
```

### 설정 파일 (eslint.config.js — Flat Config)

```javascript
import js from "@eslint/js";
import prettier from "eslint-config-prettier";
import globals from "globals";

export default [
  js.configs.recommended,   // 추천 규칙 세트
  prettier,                 // Prettier 충돌 규칙 비활성화
  {
    languageOptions: {
      globals: globals.node  // Node.js 전역 변수 허용 (console 등)
    },
    rules: {
      "no-unused-vars": "warn",     // 안 쓰는 변수 → 경고
      "no-console": "warn",         // console.log → 경고
      "eqeqeq": "error",           // == 금지, === 강제
    }
  }
];
```

### 규칙 레벨 3가지

| 값 | 의미 | CI 결과 |
|---|---|---|
| `"off"` (0) | 규칙 끔 | 무시 |
| `"warn"` (1) | 경고 | 노란색, **통과됨** |
| `"error"` (2) | 에러 | 빨간색, **실패 처리** |

### globals — 실행 환경 설정

ESLint는 기본적으로 어떤 환경인지 모름. `console`, `process` 같은 전역 변수를 인식시켜야 함.

```bash
npm install --save-dev globals
```

| 환경 | 설정 | 허용되는 전역 변수 |
|---|---|---|
| Node.js | `globals.node` | console, process, __dirname 등 |
| 브라우저 | `globals.browser` | window, document, console 등 |

---

## Prettier — 코드 포매터

### 설치 및 실행

```bash
npm install --save-dev prettier

npx prettier src/example.js    # 변환 결과를 화면에만 출력 (파일 안 바뀜)
npx prettier --check .         # 검사만 (CI용) — 문제 있으면 exit 1
npx prettier --write .         # 실제로 파일 수정
```

### 설정 파일 (.prettierrc)

```json
{
  "semi": true,           // 세미콜론 붙임
  "singleQuote": true,    // 작은따옴표 사용
  "tabWidth": 2,          // 들여쓰기 2칸
  "trailingComma": "es5", // 마지막 쉼표
  "printWidth": 80        // 한 줄 최대 80자
}
```

### --check vs --write

| 명령어 | 동작 | 용도 |
|---|---|---|
| `npx prettier file.js` | 결과를 화면에 출력 | 미리보기 |
| `npx prettier --check .` | 검사만, 파일 안 바꿈 | CI 자동 검사 |
| `npx prettier --write .` | 파일을 실제로 수정 | 로컬에서 정리 |

---

## ESLint + Prettier 함께 쓰기

둘이 충돌할 수 있음 (ESLint에도 포매팅 관련 규칙이 일부 있어서).

```bash
# 충돌 방지 패키지
npm install --save-dev eslint-config-prettier
```

```javascript
// eslint.config.js에 prettier 추가
import prettier from "eslint-config-prettier";

export default [
  js.configs.recommended,
  prettier,    // ← Prettier와 겹치는 ESLint 규칙 OFF
  { rules: { ... } }
];
```

**역할 분담**:
```
ESLint  → 코드 품질만 (버그 잡기)
Prettier → 코드 모양만 (정렬하기)
```

---

## GitHub Actions로 자동 검사

### 워크플로우 파일

```yaml
# .github/workflows/lint.yml
name: Lint & Format Check

on:
  pull_request:
    branches: [main]

jobs:
  lint-and-format:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - run: npm ci                      # 의존성 설치
      - run: npx eslint src/            # 린트 검사
      - run: npx prettier --check src/  # 포매팅 검사
```

### CI 흐름

```
PR 올림 → 워크플로우 실행
              │
    ┌─────────┼─────────┐
    ▼         ▼         ▼
  ESLint   Prettier   테스트
    │         │         │
    ▼         ▼         ▼
  통과?     통과?     통과?
    │         │         │
    └─────────┼─────────┘
              ▼
       전부 통과해야 머지 가능
```

- step이 실패하면 다음 step은 실행 안 됨
- error가 있으면 실패, warn만 있으면 통과

---

## 다른 언어의 린트/포매팅 도구

| 언어 | 린트 | 포매팅 |
|---|---|---|
| JavaScript/TS | ESLint | Prettier |
| Python | Ruff, Flake8, Pylint | Black, Ruff |
| Go | golangci-lint | gofmt (내장!) |
| Rust | clippy | rustfmt (내장!) |
| C# | dotnet-format | dotnet-format |

---

## 주의사항 & 흔한 실수

1. **package.json에 `"type": "module"` 필요** — eslint.config.js에서 `import` 문법 쓰려면
2. **globals 패키지 설치 잊지 말기** — import만 하고 `languageOptions`에 안 넣으면 효과 없음
3. **.prettierrc 오타 주의** — `"ec5"` 같은 오타가 있으면 Prettier 자체가 실행 실패
4. **node_modules/ 커밋하지 말기** — `.gitignore`에 반드시 추가
5. **ESLint와 Prettier 역할 섞지 말기** — `eslint-config-prettier`로 충돌 방지

## 명령어 정리표

| 명령어 | 용도 |
|---|---|
| `npm install --save-dev eslint @eslint/js` | ESLint 설치 |
| `npm install --save-dev prettier` | Prettier 설치 |
| `npm install --save-dev eslint-config-prettier` | 충돌 방지 |
| `npm install --save-dev globals` | 환경 전역변수 설정 |
| `npx eslint src/` | 린트 검사 |
| `npx eslint --fix src/` | 린트 자동 수정 |
| `npx prettier --check .` | 포매팅 검사 (CI용) |
| `npx prettier --write .` | 포매팅 자동 수정 |
| `npm pkg set type=module` | package.json에 ES 모듈 설정 |
