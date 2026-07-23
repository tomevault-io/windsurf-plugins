---
trigger: always_on
description: AI 에이전트를 위한 프로젝트 컨텍스트 및 작업 가이드
---

# AGENTS.md

AI 에이전트를 위한 프로젝트 컨텍스트 및 작업 가이드

---

## 프로젝트 개요

**azure-sdk-korean**은 [Azure SDK](https://github.com/Azure/azure-sdk) 영문 문서를 한글로 번역하고 유용한 정보를 모으는 커뮤니티 프로젝트입니다.

- **저장소**: https://github.com/Azure/azure-sdk-korean
- **라이브 사이트**: https://azure.github.io/azure-sdk-korean/
- **빌드 시스템**: Jekyll (GitHub Pages)
- **마지막 활동**: 2023년 10월 (2026년 3월 재활성화)

---

## 프로젝트 상태 (2026-03-27 기준)

### ✅ 완료된 작업

#### Infrastructure & Security (2026-03)
- **GitHub Actions 워크플로우** - `.github/workflows/pages.yml` 자동 배포
- **빌드 상태 배지** - README.md에 추가
- **보안 취약점 업데이트**:
  - github-pages gem: 228 → 231
  - activesupport: 6.1.7.5 → 8.1.3
  - Dependabot 알림: 19개 → 17개
  - 남은 17개 알림은 빌드 타임 의존성으로 프로덕션 위험 없음 (#196에 문서화)

#### Issue Triage (2026-03)
- **닫힌 이슈**: #196 (보안), #130 (빌드), #178 (중복), #13 (완료), #149 (하이퍼링크)
- **오픈 이슈**: 26개 → 21개

#### Content Status
- **Python Guidelines**: 
  - `design.md` - 849 lines ✅
  - `implementation.md` - 548 lines ✅
- **.NET Guidelines**: `introduction.md` - 1,090 lines ✅
- **Java Guidelines**: `introduction.md` + `spring.md` - 1,315 lines ✅

### 🔲 남은 작업

#### Phase 2: Stale Issue Review
2021-2022년 오래된 이슈 검토 필요:
- #6 - 번역 진행상황 추적 도구 제안
- #24 - 번역 안된 문서: Azure SDK 저장소 가리키기
- #29 - HTML 내용 번역
- #87 - Python design에서 "Idiomatic" 한글 번역
- #106 - Azure SDK for Python 가이드 페이지 추가
- #109 - Python Contribution 페이지 추가

#### Quick Wins
- #148 - `docs/policies/reviewprocess.md` 용어 수정
- #147 - `docs/general/terminology.md` 용어 수정
- #142 - `general_terminology.html` 용어 추가/수정

#### Documentation Updates Needed
업스트림 Azure SDK 문서 변경으로 outdated:
- #171 - General Guidelines > Implementation > 'Dependencies'
- #170 - General Guidelines > Implementation > 'Distributed Tracing'
- #169 - General Guidelines > Implementation > 'Parameter validation'

#### Governance
- #189 - 커밋 메시지 컨벤션 문서화 (진행 중)
- #155 - Azure SDK Korean contribution 가이드라인 추가
- #180, #179 - 메타데이터 자동화 (낮은 우선순위)

#### UI/UX
- #129 - Sidebar 개선 (작은 화면에서 레이아웃 문제)

#### Translation Coordination
2023년 중단된 번역 작업 재개 여부 결정:
- #158 - Python Guidelines: Implementation (대부분 완료, 검증 필요)
- #157 - General Guidelines: Contribution
- #153 - General Guidelines: Implementation 나머지
- #146 - Policies: Releases 나머지

---

## 기술 스택 & 빌드

### 의존성
- **Ruby**: 3.2.3
- **Bundler**: 2.3.22
- **Jekyll**: 3.9.3 (via github-pages)
- **github-pages gem**: 231

### 로컬 빌드
```bash
bundle install
bundle exec jekyll build
bundle exec jekyll serve  # http://localhost:4000
```

### GitHub Pages 배포
- **트리거**: Push to `main` branch
- **워크플로우**: `.github/workflows/pages.yml`
- **빌드 시간**: ~3초
- **배포 URL**: https://azure.github.io/azure-sdk-korean/

---

## 작업 워크플로우

### 필수 규칙

1. **이슈 우선 생성**: 모든 작업은 GitHub Issue를 먼저 등록하고 진행
2. **커밋 메시지 컨벤션**: Conventional Commits 사용 (아래 참조)
3. **최소 변경 원칙**: Scope creep 방지, focused changes
4. **빌드 검증**: 변경 후 Jekyll 빌드 확인

### Commit Message Convention

**Format:**
```
<type>: <subject>

[optional body]

[optional footer]
```

**Types:**
- `docs:` - 문서 번역, 수정
- `feat:` - 새 기능 추가
- `fix:` - 버그 수정
- `chore:` - 빌드, 의존성 업데이트
- `style:` - 코드 스타일, 포맷 (번역 아님)
- `refactor:` - 리팩토링

**Footer:**
- `Refs #123` - 이슈 참조
- `Closes #123` - 이슈 닫기
- `Co-authored-by: Name <email>` - 공동 작업자

**Examples:**
```
docs: translate Python Guidelines Implementation

Translate API Implementation and Testing sections.

Refs #158

---

fix: correct hyperlink in header

Changed plain text URL to Markdown hyperlink.

Closes #149

---

chore: update github-pages to v231

- Update github-pages gem from 228 to 231
- Regenerate Gemfile.lock
- Update activesupport to 8.1.3

Reduces Dependabot alerts from 19 to 17.

Refs #196
```

### 브랜치 전략
- **main**: 프로덕션 브랜치 (GitHub Pages 배포)
- **feature/***: 기능 개발 브랜치 (선택사항)
- **Pull Request**: 권장되지만 필수 아님 (소규모 변경은 직접 main push 가능)

---

## 보안 정책

### Dependabot 알림 처리

**현재 상태 (2026-03-27):**
- 17개 알림 (1 critical, 2 high, 11 moderate, 3 low)
- 모두 **빌드 타임 의존성** - 프로덕션 노출 없음
- 위험도 평가 및 수용 근거: Issue #196 참조

**정책:**
- 분기별 리뷰 (다음: 2026년 6월)
- Critical/High 알림 발생 시 즉시 검토
- 빌드 전용 취약점은 문서화된 위험 수용 가능

**참고 문서:**
- Issue #196 - 전체 위험도 분석 및 컴포넌트별 분류
- `SECURITY.md` - Microsoft 표준 보안 정책

---

## 디렉토리 구조

```
azure-sdk-korean/
├── .github/
│   └── workflows/
│       └── pages.yml          # GitHub Actions 자동 배포
├── docs/
│   ├── android/               # Android SDK 가이드라인
│   ├── contribution/          # 기여 가이드
│   ├── dotnet/                # .NET SDK 가이드라인
│   ├── general/               # 일반 가이드라인
│   ├── ios/                   # iOS SDK 가이드라인
│   ├── java/                  # Java SDK 가이드라인
│   ├── policies/              # 정책 문서
│   ├── python/                # Python SDK 가이드라인
│   │   ├── design.md          # 849 lines ✅
│   │   └── implementation.md  # 548 lines ✅
│   ├── tables/                # 네임스페이스 테이블
│   └── tracing/               # 분산 추적
├── _includes/                 # Jekyll includes
│   └── info/header.md         # 사이트 헤더
├── _layouts/                  # Jekyll 레이아웃
├── _config.yml                # Jekyll 설정
├── Gemfile                    # Ruby 의존성
├── Gemfile.lock               # 의존성 잠금
├── README.md                  # 프로젝트 README
├── CONTRIBUTING.md            # 기여 가이드
├── SECURITY.md                # 보안 정책
└── AGENTS.md                  # 🤖 이 파일
```

---

## 외부 참조

### Upstream 문서
- **Azure SDK Guidelines**: https://aka.ms/azsdk/guide

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure/azure-sdk-korean](https://github.com/Azure/azure-sdk-korean) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
