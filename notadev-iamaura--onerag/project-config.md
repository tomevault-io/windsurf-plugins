---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 프로젝트 개요

React 19 + TypeScript + Material-UI로 구현된 RAG(Retrieval-Augmented Generation) 챗봇 프론트엔드 애플리케이션입니다. 사용자는 문서를 업로드하고, 업로드된 문서를 기반으로 AI 챗봇과 대화할 수 있으며, 문서 관리 및 시스템 통계를 확인할 수 있습니다.

## 📚 주요 개발 문서

프로젝트의 핵심 기능과 시스템에 대한 상세 문서는 `docs/` 디렉토리에서 확인할 수 있습니다:

### 필수 읽기 문서
- **[색상 관리 시스템 가이드](docs/COLOR_SYSTEM_GUIDE.md)** ⭐ **중요**
  - 중앙 집중식 색상 관리 시스템 (`src/config/colors.ts`)
  - `COLORS` 객체 및 `getColor()` 헬퍼 함수 사용법
  - Light/Dark 모드 지원 방법
  - ESLint 규칙: 하드코딩된 hex/rgba 색상 사용 금지
  - **신규 컴포넌트 개발 시 반드시 참조**

### 기능별 문서
- **[ChatEmptyState 설정 관리](docs/CHAT_EMPTY_STATE_SETTINGS.md)**: 챗봇 Empty State 메시지 및 추천 질문 설정
- **[브랜드 설정 가이드](docs/BRAND_CONFIGURATION_GUIDE.md)**: 브랜드 색상, 로고, 타이틀 커스터마이징
- **[기능 플래그 가이드](docs/FEATURE_FLAGS_GUIDE.md)**: Feature Flag 시스템 사용법
- **[Railway 배포 가이드](docs/RAILWAY_DEPLOYMENT_GUIDE.md)**: Railway 플랫폼 배포 방법

## 핵심 개발 명령어

### 개발 서버
```bash
npm run dev
# Vite 개발 서버를 http://localhost:5000에서 시작합니다.
# API 프록시는 vite.config.ts에서 Railway 백엔드로 설정되어 있습니다.
```

### 빌드 및 배포
```bash
npm run lint              # ESLint 검사 실행
npm run lint -- --fix     # ESLint 자동 수정
npm run build:warning-gate # 경고 게이트 포함 프로덕션 빌드 (dist/ 폴더에 생성)
npm run preview           # 빌드 결과 미리보기
npm run build:railway     # Railway 배포용 빌드 (lint + build + 런타임 설정 생성)
```

### 테스트
```bash
npm run test              # Vitest 워치 모드로 테스트 실행
npm run test:warning-gate # 경고 게이트 포함 테스트 1회 실행 (CI용)
npm run warning-gate:self-test # 경고 게이트 자체 검증
npm run test:ui           # Vitest UI로 테스트 실행
npm run test:coverage     # 테스트 커버리지 리포트 생성
```

### 환경 변수 설정
개발 환경에서는 `.env` 파일에 다음 변수를 설정:
```env
VITE_API_BASE_URL=http://localhost:8000
VITE_DEV_API_BASE_URL=http://localhost:8000
VITE_DEV_WS_BASE_URL=ws://localhost:8000
VITE_ACCESS_CODE=1127
```

## 아키텍처 및 주요 설계 결정사항

### 1. 라우팅 및 Feature Flag 시스템
`App.tsx`는 애플리케이션의 진입점이며, Feature Flag에 따라 라우트를 조건부로 활성화합니다:
- `/` - 랜딩 페이지 (활성화된 첫 번째 모듈로 자동 리다이렉션)
- `/bot` - 챗봇 인터페이스 (`ChatPage`)
- `/upload` - 문서 업로드 및 관리 (`UploadPage`)
- `/prompts` - 프롬프트 관리 (`PromptsPage`)
- `/analysis` - 분석 페이지 (`AnalysisPage`)
- `/admin` - 관리자 대시보드 (`AdminDashboard`)

**Context Provider 계층 구조**:
```
ConfigProvider (런타임 설정)
  └─ FeatureProvider (Feature Flag)
       └─ Router
            └─ AppRoutes
```

모든 라우트는 `ErrorBoundary`와 `ProtectedRoute`로 감싸져 있습니다.

### 2. API 통신 레이어 (`src/services/`)
#### API URL 우선순위 (src/services/api.ts)
1. **개발 모드**: `VITE_DEV_API_BASE_URL` 또는 Railway 프로덕션 백엔드
2. **프로덕션**: `VITE_API_BASE_URL` 환경변수
3. **런타임 설정**: `window.RUNTIME_CONFIG.API_BASE_URL`
4. **폴백**: `http://localhost:8000`

#### Axios 설정
- **타임아웃**: 5분 (대용량 문서 처리 대응)
- **재시도**: 최대 3회, 지수 백오프 (네트워크 오류, 5xx, 429 시)
- **Request 인터셉터**: API Key, JWT 토큰, 세션 ID, CSRF 토큰 자동 추가
- **Response 인터셉터**: 401 시 토큰 갱신, 전화번호 자동 마스킹

#### API 모듈 구조
- `documentAPI` - 문서 업로드, 조회, 삭제 (단일/일괄/전체)
- `chatAPI` - 메시지 전송, 채팅 기록, 세션 관리
- `healthAPI` - 서버 헬스 체크 (별도 타임아웃)
- 추가 서비스: `promptService.ts`, `qdrantService.ts`, `adminService.ts`, `authService.ts`, `chatSettingsService.ts`

### 3. 세션 관리
- **저장 위치**: `localStorage` (`chatSessionId` 키)
- **생성**: `chatAPI.startNewSession()` 호출 시 백엔드에서 생성
- **사용**: 모든 채팅 API 요청에 `X-Session-Id` 헤더로 자동 포함
- **새 세션 요청 시**: 기존 세션 ID를 보내지 않음 (인터셉터에서 처리)

### 4. Feature Flag 시스템 (`src/config/features.ts`)
모듈별 기능을 세밀하게 제어할 수 있는 Feature Flag 시스템:
- **chatbot**: streaming, history, sessionManagement, markdown
- **documentManagement**: upload, bulkDelete, search, pagination, dragAndDrop, preview
- **admin**: userManagement, systemStats, qdrantManagement, accessControl
- **prompts**: templates, history
- **analysis**: realtime, export, visualization
- **privacy**: maskPhoneNumbers

우선순위: 런타임 구성 (`window.RUNTIME_CONFIG.FEATURES`) > 환경변수 (`VITE_FEATURE_*`) > 기본값

### 5. 테마 시스템 (`src/theme/index.ts`)
모노톤 디자인 시스템 기반의 MUI 테마:
- **모든 색상**: `src/config/colors.ts`에서 중앙 관리
- **Dark/Light 모드**: `COLORS` 객체에서 모드별 색상 자동 적용
- **Typography**: Apple 시스템 폰트
- **그림자/보더**: 모노톤 스타일로 통일

### 6. 에러 처리 전략
- **ErrorBoundary 컴포넌트**: React 컴포넌트 트리에서 발생하는 JavaScript 오류를 포착
- **API 오류**: Axios 인터셉터에서 중앙 집중식 처리, 자동 재시도
- **401 처리**: 토큰 갱신 시도 후 실패 시 로그아웃

## 주요 디렉토리 구조

```
src/
├── main.tsx                 # React 앱 진입점
├── App.tsx                  # 라우팅 및 Provider 설정
├── components/              # 재사용 가능한 컴포넌트
│   ├── ChatTab.tsx         # 채팅 인터페이스
│   ├── ChatEmptyState.tsx  # 채팅 초기 상태 UI
│   ├── DocumentsTab.tsx    # 문서 목록 (검색, 페이지네이션, 삭제)
│   ├── UploadTab.tsx       # 문서 업로드 (드래그앤드롭)
│   ├── StatsTab.tsx        # 통계 대시보드
│   ├── AppLayout.tsx       # 전역 레이아웃
│   ├── AppHeader.tsx       # 앱 헤더
│   ├── Sidebar.tsx         # 사이드바 네비게이션
│   ├── MarkdownRenderer.tsx # 마크다운 렌더링
│   ├── ErrorBoundary.tsx   # 에러 경계
│   └── icons/              # 커스텀 아이콘 컴포넌트
├── pages/                   # 페이지 컴포넌트 (lazy loading)
│   ├── ChatPage.tsx        # /bot
│   ├── UploadPage.tsx      # /upload
│   ├── PromptsPage.tsx     # /prompts
│   ├── AnalysisPage.tsx    # /analysis
│   └── Admin/              # 관리자 페이지
├── services/                # API 클라이언트
│   ├── api.ts              # Axios 설정 및 주요 API
│   ├── authService.ts      # 인증 관련 API
│   ├── promptService.ts    # 프롬프트 관련 API
│   ├── qdrantService.ts    # Qdrant 벡터 DB API

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [notadev-iamaura/OneRAG](https://github.com/notadev-iamaura/OneRAG) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
