# 자동 배포 (Automated Deployment)

## 핵심 개념

자동 배포는 **main에 머지되면 자동으로 배포**하는 것. CI/CD에서 CD(Continuous Deployment) 부분이야.

```
PR 생성 → 테스트 자동 실행 (CI)
    ↓ 통과
리뷰 & 머지
    ↓
main에 push 발생 → 배포 자동 실행 (CD)
    ↓
빌드 → 배포 → 완료!
```

### CI vs CD

| 구분 | CI (Continuous Integration) | CD (Continuous Deployment) |
|------|---------------------------|---------------------------|
| 트리거 | `on: pull_request` | `on: push, branches: [main]` |
| 시점 | PR 생성/업데이트 시 | main에 머지(push) 시 |
| 용도 | 테스트 | 배포 |

---

## 배포 트리거

```yaml
on:
  push:
    branches: [main]
```

PR이 머지되면 main에 push가 발생하므로, 이 트리거가 자동 배포의 핵심!

---

## Secrets (비밀 값 관리)

배포에 필요한 민감한 정보(API 키, 비밀번호, 토큰)를 안전하게 관리하는 기능.

### 설정 방법

```
Settings → Secrets and variables → Actions → New repository secret
```

### 워크플로우에서 사용

```yaml
steps:
  - name: Deploy
    env:
      API_KEY: ${{ secrets.API_KEY }}
    run: echo "Deploying..."
```

### Secrets의 특징

- **한번 저장하면 다시 볼 수 없음** (수정/삭제만 가능)
- **로그에 자동 마스킹** — `***`로 표시
- **Fork에서는 접근 불가** — 외부 PR이 secrets 훔치는 것 방지

### Secrets vs Variables

| 구분 | Secrets | Variables |
|------|---------|-----------|
| 용도 | API 키, 비밀번호, 토큰 | 환경 이름, 설정값 |
| 보안 | 암호화, 마스킹 | 평문, 로그에 노출 |
| 참조 | `${{ secrets.NAME }}` | `${{ vars.NAME }}` |
| 다시 보기 | ❌ 불가 | ✅ 가능 |

---

## Environment (배포 환경)

배포 대상을 구분하는 기능. Settings → Environments에서 설정.

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production    # ← 환경 지정
```

### Environment의 기능

- **환경별 secrets** — staging과 production에 다른 API 키 설정 가능
- **보호 규칙** — production 배포 전 승인 필요하게 설정 가능
- **배포 이력** — 누가 언제 어디에 배포했는지 추적

---

## 실전 워크플로우: GitHub Pages 배포

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Pages
        uses: actions/configure-pages@v4

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: './docs'

      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

### 주요 포인트

- `permissions`: 워크플로우에 필요한 권한 명시
- `actions/upload-pages-artifact@v3`: 배포할 파일을 artifact로 업로드
- `actions/deploy-pages@v4`: GitHub Pages에 실제 배포
- GitHub Pages 설정에서 Source를 **GitHub Actions**로 변경해야 함

---

## SSH로 서버 배포 예시

```yaml
- name: Deploy via SSH
  uses: appleboy/ssh-action@v1
  with:
    host: ${{ secrets.SERVER_HOST }}
    username: ${{ secrets.SERVER_USER }}
    key: ${{ secrets.SSH_PRIVATE_KEY }}
    script: |
      cd /var/www/myapp
      git pull origin main
      npm install
      npm run build
      pm2 restart myapp
```

---

## 명령어 정리표

| 명령어/문법 | 용도 |
|-------------|------|
| `on: push, branches: [main]` | 배포 트리거 |
| `${{ secrets.NAME }}` | Secrets 참조 |
| `${{ vars.NAME }}` | Variables 참조 |
| `environment: production` | 배포 환경 지정 |
| `permissions:` | 워크플로우 권한 설정 |
| `gh run list` | 워크플로우 실행 목록 확인 |

---

## 주의사항 & 흔한 실수

- **GitHub Pages Source 설정**: "Deploy from a branch"가 기본값. **GitHub Actions**로 바꿔야 워크플로우 배포가 작동함
- **Squash merge 후 충돌**: 첫 번째 PR이 squash merge되면 해시가 달라져서, 같은 파일을 수정한 두 번째 브랜치에서 충돌 발생 가능. `git fetch origin main && git merge origin/main`으로 해결
- **Secrets는 Fork PR에서 접근 불가**: 외부 기여자의 PR에서 배포 워크플로우가 실패할 수 있음
