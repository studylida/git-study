# README & Markdown

## 마크다운 핵심 문법

```markdown
# 제목 1
## 제목 2
### 제목 3

**굵게**  _기울임_  ~~취소선~~

- 목록
  - 중첩 목록

1. 순서 있는 목록

`인라인 코드`

```bash
코드 블록 (언어 지정 가능)
```

[링크 텍스트](URL)
![대체텍스트](이미지URL)   ← 링크 앞에 ! 만 붙이면 이미지

| 열1 | 열2 |
|-----|-----|
| 값1 | 값2 |

> 인용문

- [ ] 체크박스 (미완료)
- [x] 체크박스 (완료)
```

## 링크 vs 이미지 차이

- 링크: `[텍스트](URL)` → 클릭하면 URL로 이동
- 이미지: `![대체텍스트](이미지URL)` → 이미지를 직접 렌더링
- 차이: 앞에 `!` 유무

## 뱃지 (Badge)

뱃지는 단순한 이미지다. shields.io 같은 서비스가 상태를 실시간으로 이미지 URL로 제공.

```markdown
![빌드상태](https://img.shields.io/badge/build-passing-green)
```

GitHub Actions와 연결하면 실제 빌드 결과가 자동으로 반영됨.

## 잘 만든 README 구성

```
# 프로젝트 이름
한 줄 설명

## 설치 방법
## 사용 방법
## 기여 방법
## 라이센스
```

## GitHub 전용 문법

- `- [ ]` / `- [x]` → 체크박스
- `@username` → 멘션
- `#123` → 이슈/PR 링크 자동 연결

---

## 실습 가이드: README.md 직접 만들어보기

### Step 1: 프로젝트 레포 준비

```bash
mkdir ~/readme-practice && cd ~/readme-practice
git init

# 간단한 프로젝트 파일 만들기
echo 'print("Hello World")' > main.py
git add . && git commit -m "init: 프로젝트 시작"
```

### Step 2: README.md 작성

```bash
cat > README.md << 'EOF'
# My Awesome Project

간단한 인사 프로그램입니다.

![Python](https://img.shields.io/badge/python-3.8+-blue)

## 설치 방법

```bash
git clone https://github.com/username/my-project.git
cd my-project
```

## 사용 방법

```bash
python main.py
```

## 기능

- [x] 인사 출력
- [ ] 사용자 이름 입력 받기
- [ ] 다국어 지원

## 기여 방법

1. Fork합니다
2. 브랜치를 만듭니다 (`git switch -c feature/amazing`)
3. 변경사항을 커밋합니다
4. Push합니다 (`git push origin feature/amazing`)
5. PR을 보냅니다

## 라이센스

MIT License
EOF

git add README.md && git commit -m "docs: README 추가"
```

### Step 3: GitHub에서 렌더링 확인

```bash
# GitHub에 올려서 확인
gh repo create readme-practice --public --source=. --push

# 브라우저에서 열기
gh repo view --web
# → README.md가 예쁘게 렌더링된 걸 확인!
```

### Step 4: 뱃지 추가해보기

```markdown
<!-- shields.io에서 다양한 뱃지를 만들 수 있다 -->
![Build](https://img.shields.io/badge/build-passing-brightgreen)
![License](https://img.shields.io/badge/license-MIT-blue)
![Version](https://img.shields.io/badge/version-1.0.0-orange)
```

### 정리

```bash
# 실습 레포 삭제 (선택)
gh repo delete readme-practice --yes
rm -rf ~/readme-practice
```

---

## 주의사항 & 흔한 실수

- 마크다운에서 **줄바꿈은 빈 줄 하나**가 필요함. 그냥 엔터 한 번은 같은 줄로 이어짐
- 코드 블록 안에 코드 블록을 넣을 때는 **백틱 개수를 다르게** (바깥은 4개, 안쪽은 3개)
- 이미지 링크가 깨지면 `![대체텍스트]`만 텍스트로 보임 → 이미지 URL 확인
- GitHub에서 마크다운 미리보기: 파일 편집 시 **Preview** 탭으로 확인 가능
- 테이블에서 `|` 파이프 문자를 내용으로 쓰려면 `\|`로 이스케이프
