---
trigger: always_on
description: Stanford University의 "The Modern Software Developer" 강좌를 한국어로 번역하여 제공하는 웹 플랫폼입니다.
---

# Stanford CS146S Korean Edition

Stanford University의 "The Modern Software Developer" 강좌를 한국어로 번역하여 제공하는 웹 플랫폼입니다.

## 프로젝트 개요

- **URL**: https://kr.themodernsoftware.dev
- **원본**: Stanford CS146S by Mihail Eric
- **목적**: AI 지원 개발(Coding with LLMs) 교육 콘텐츠 한국어 번역

## 기술 스택

- **Framework**: React 19 + TypeScript
- **Build**: Vite 6
- **Styling**: Tailwind CSS 4
- **Routing**: React Router 7
- **Deploy**: Vercel (서울 리전)
- **Package Manager**: pnpm

## 프로젝트 구조

```
src/
├── components/
│   ├── layout/     # Layout, Header, Footer, TabNav
│   ├── home/       # CourseDescription, InfoGrid, TeamGrid, TranslatorGrid
│   ├── syllabus/   # WeekCard, ReadingList, LectureList
│   └── faq/        # FaqAccordion
├── pages/          # HomePage, ReadingPage
├── content/        # syllabus.ts, readings.ts, faq.ts
└── types/          # TypeScript 인터페이스
```

## docs 디렉토리 구조

Reading 콘텐츠는 eng/kr 폴더로 원본과 번역을 분리합니다.

```
docs/week{N}/{slug}/
├── eng/                      # 원본 (영어)
│   ├── index.md              # 단일 페이지
│   ├── _index.md             # 부모 인덱스 (챕터 있는 경우)
│   └── {child}.md            # 챕터/자식 페이지
├── kr/                       # 번역본 (한국어)
│   ├── index.md
│   ├── _index.md
│   └── {child}.md
└── media/                    # 원본 미디어 파일
    ├── *.vtt                 # YouTube 자막
    └── *.pdf                 # PDF 원본
```

### 파일명 규칙

| 콘텐츠 유형 | 원본 | 번역 |
|------------|------|------|
| 단일 페이지 | `{slug}/eng/index.md` | `{slug}/kr/index.md` |
| 부모 인덱스 | `{slug}/eng/_index.md` | `{slug}/kr/_index.md` |
| 챕터/자식 | `{slug}/eng/{child}.md` | `{slug}/kr/{child}.md` |

### 미디어 파일

각 콘텐츠의 원본 미디어는 `{slug}/media/` 폴더에 저장:
- YouTube VTT 자막: `{slug}/media/{slug}.en.vtt`
- PDF 원본: `{slug}/media/original.pdf`

## 주요 명령어

```bash
pnpm dev      # 개발 서버 시작
pnpm build    # TypeScript 체크 + 프로덕션 빌드
pnpm lint     # ESLint 검사
pnpm preview  # 빌드 결과 미리보기
```

## 개발 가이드라인

### 컴포넌트 작성

- 모든 컴포넌트는 TypeScript로 작성
- Props는 인터페이스로 정의
- 경로 임포트는 `@/` alias 사용 (예: `@/components/layout/Header`)

### 스타일링

- Tailwind CSS 유틸리티 클래스 사용
- 커스텀 색상은 index.css의 CSS 변수 사용:
  - `--color-stanford-red`: #8B0000 (Stanford 공식 색상)
  - `--color-kr-accent`: #0066CC (한국어 강조)

### Reading 페이지 타이포그래피

Reading 페이지는 Medium 스타일 타이포그래피를 적용합니다.

**CSS 클래스**: `.prose-reading` (src/index.css)

**핵심 특징**:
- 본문: Noto Serif KR (세리프), 18-20px, line-height 1.78-1.82
- 제목: Inter (산세리프) - 본문과 대비 효과
- 최대 너비: 680px (65-75자, 최적 가독성)
- 문단 간격: 1.5em

**CSS 변수**:
- `--font-serif`: Noto Serif KR 폰트 스택
- `--color-text-reading`: #292929 (부드러운 검정)
- `--color-text-reading-secondary`: #6B6B6B (보조 텍스트)

**사용 예시**:
```tsx
<div className="prose-reading">
  <h2>제목</h2>
  <p>본문 내용...</p>
</div>
```

**특수 요소 오버라이드**: Tailwind `!` 접두사 사용
```tsx
<p className="!text-sm !font-sans">특수 요소</p>
```

### 콘텐츠 추가

- 새 Reading 번역: `src/content/readings.ts`에 추가
- 강의 정보 수정: `src/content/syllabus.ts` 수정
- FAQ 추가: `src/content/faq.ts` 수정

### 번역 상태 관리

`TranslationStatus` 타입 사용:
- `complete`: 번역 완료 (한국어 링크 활성화)
- `in_progress`: 번역 중 ("번역중" 라벨)
- `none`: 예정 ("예정" 라벨)

## 라우팅

- `/`: 메인 페이지 (Overview, Syllabus, FAQ 탭)
- `/readings/week/:week/:slug`: Reading 상세 페이지

## 배포

main 브랜치에 push하면 Vercel에서 자동 배포됩니다.

## 커밋 메시지

Conventional Commits 형식을 따릅니다:
- `feat:` 새 기능
- `fix:` 버그 수정
- `refactor:` 리팩토링
- `docs:` 문서 수정
- `style:` 스타일 수정

## 주의사항

- TypeScript strict 모드 활성화됨 - 타입 에러 해결 필수
- 빌드 전 `pnpm lint` 실행 권장
- 한글 콘텐츠 작성 시 맞춤법 검토
- **대량 한글 파일 수정 시 Write 도구 사용** - Edit 도구는 UTF-8 한글 3바이트 경계 오류 발생 가능

---

## 작업 완료 후 검증 및 커밋 워크플로우

코드나 문서 수정 작업을 완료한 후 다음 순서를 따릅니다:

1. **수정사항 재검증**
   - 수정한 파일의 내용이 올바른지 확인
   - 번역 파일의 경우 포맷과 마크다운 구문 검증
   - 코드 파일의 경우 `pnpm build` 및 `pnpm lint` 실행

2. **검증 완료 후 커밋 스킬 실행**
   - 검증이 완료되면 `/commit` 스킬을 실행하여 변경사항 커밋
   - 커밋 메시지는 conventional commit 형식 준수

---

## 번역 워크플로우

Reading 콘텐츠의 수집 → 번역 → 웹 게시를 3단계 스킬로 자동화합니다.

```
/fetch-reading <url>          # 1. 원본 수집
        ↓
/translate-reading <week/slug> # 2. 한국어 번역
        ↓
/upload-reading <week/slug>    # 3. 웹 게시
```

### 진행 상황

| 스킬 | 상태 | 설명 |
|------|------|------|
| `/fetch-reading` | 완료 | URL에서 원본 수집 (YouTube 챕터 자동 분리) |
| `/translate-reading` | 완료 | 한국어 번역 (`--all-chapters` 지원) |
| `/upload-reading` | 완료 | 웹 게시 → `readings.ts` + `syllabus.ts` 업데이트 |
| `/nanobanana` | 완료 | 치트시트 프롬프트 생성 (`--per-chapter` 지원) |
| `/publish-cheatsheet` | 완료 | 치트시트 이미지 게시 |
| `/review-translation` | 완료 | 번역 품질 AI 검증 (Claude, Codex, Gemini 교차 검증) |
| `/review-cheatsheet` | 완료 | 치트시트 이미지 검증 (Gemini Vision API) |
| `/eval-summary` | 완료 | readings.ts 요약 필드 품질 검증 및 수정 |
| `/commit` | 완료 | 커밋 메시지 자동 생성, 사용자 확인 후 커밋 |
| `/create-pr` | 완료 | GitHub PR 생성, fork 워크플로우 지원 |

---

### YouTube 콘텐츠 전체 파이프라인

YouTube 영상은 다음 순서로 처리합니다. 챕터가 있는 경우 **자동으로 분리**됩니다.

```
1. /fetch-reading <youtube-url>
   → 챕터 있음: docs/week{N}/{slug}/ 디렉토리 생성
       ├── eng/                    # 원본 (영어)
       │   ├── _index.md (인덱스)
       │   ├── introduction.md (챕터 1)
       │   ├── tokenization.md (챕터 2)
       │   └── ... (N개 챕터)
       ├── kr/                     # 번역 시 생성
       └── media/                  # 원본 미디어 (VTT 자막 등)
   → 챕터 없음: docs/week{N}/{slug}/eng/index.md (폴더 구조)

2. /translate-reading week{N}/{slug} --all-chapters
   → docs/week{N}/{slug}/kr/*.md (챕터별 번역)
   또는 개별 챕터만:
   /translate-reading week{N}/{slug}/tokenization


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [team-attention/stanford-cs146s-kr](https://github.com/team-attention/stanford-cs146s-kr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
