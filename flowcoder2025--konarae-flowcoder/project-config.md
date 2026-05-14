---
trigger: always_on
description: > **프로젝트 정보**: `/docs/prd.md` 참조
---

# 루트 가이드 (헌법) - 허브

> **프로젝트 정보**: `/docs/prd.md` 참조
>
> **🏛️ 역할**: 전역 원칙 + 중앙 허브 (모든 하위 가이드의 진입점)
> **🔗 구조**: Hub-Spoke 아키텍처 - 이 파일에서 모든 하위 가이드로 연결

```
                    ┌─────────────────┐
                    │   CLAUDE.md     │
                    │   (루트 허브)    │
                    └────────┬────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
        ▼                    ▼                    ▼
   ┌─────────┐         ┌─────────┐         ┌─────────┐
   │ /src/app│         │/src/lib │         │ /prisma │
   │  (라우팅) │         │ (백엔드) │         │  (DB)   │
   └─────────┘         └─────────┘         └─────────┘
        │
        ├── /components ── /features ── /hooks ── /types
```

---

## 1. 프로젝트 컨텍스트

> ⚠️ **필수**: 작업 시작 전 `/docs/prd.md`를 먼저 읽어 프로젝트 정보를 파악하세요.

| 항목 | 참조 위치 |
|-----|----------|
| 프로젝트명 | `/docs/prd.md` → 프로젝트명 섹션 |
| 목적 | `/docs/prd.md` → 목적 섹션 |
| 범위 | `/docs/prd.md` → 범위/스코프 섹션 |
| 대상 사용자 | `/docs/prd.md` → 타겟 유저 섹션 |

### 1.1 이 Skill의 핵심 가치

**"Primary Color만 바꾸면 브랜드 완성"**

- 디자인 토큰 기반 일관된 UI 시스템
- 자연어 요청 시 정의된 룰에 따라 완성형 UI 생성
- 프로젝트 독립적 재사용 가능한 구조

---

## 2. 기술 스택

| 영역 | 기술 |
|-----|------|
| Framework | Next.js 15 + React 19 + TypeScript |
| Styling | Tailwind CSS 4 + shadcn/ui (new-york) |
| Database | Supabase PostgreSQL + Prisma ORM + pgvector |
| Auth | NextAuth.js + ReBAC 권한 시스템 |
| AI/ML | Gemini 2.5 Pro Vision (문서 분석) + OpenAI Embeddings |
| Storage | Supabase Storage |
| Deploy | Vercel (Main) + Railway (Worker) |

**참조 스킬**: `fdp-backend-architect` (백엔드 아키텍처)

---

## 3. 허브-스포크 네비게이션 맵

### 3.1 📍 빠른 참조 (Quick Navigation)

| 작업 목적 | 가이드 위치 | 핵심 내용 |
|----------|------------|----------|
| **페이지/API 개발** | `/src/app/claude.md` | App Router, API Routes, 라우팅 패턴 |
| **UI 컴포넌트** | `/src/components/claude.md` | shadcn/ui, Button, Modal, 접근성 |
| **기능 모듈** | `/src/features/claude.md` | Feature 구조, 상태관리, 네이밍 |
| **백엔드/유틸** | `/src/lib/claude.md` | Prisma, Auth, ReBAC, i18n |
| **커스텀 훅** | `/src/hooks/claude.md` | 훅 패턴, 네이밍, 테스트 |
| **타입 정의** | `/src/types/claude.md` | 타입 체계, 네이밍 컨벤션 |
| **DB 스키마** | `/prisma/claude.md` | 모델 관계, 마이그레이션, pgvector |

### 3.2 🗂️ 전체 디렉토리 구조

```
/{project_root}
├── CLAUDE.md                    # 🏛️ [현재 파일] 루트 허브 (헌법)
│
├── /src
│   ├── /app                     # 🚀 Next.js App Router
│   │   ├── claude.md            # → App Router 가이드
│   │   ├── globals.css          #    디자인 토큰 정의
│   │   ├── /(app)               #    인증된 사용자 라우트
│   │   ├── /admin               #    관리자 라우트
│   │   ├── /api                 #    API Routes
│   │   └── /login               #    로그인 페이지
│   │
│   ├── /components              # 🎨 UI 컴포넌트
│   │   ├── claude.md            # → 컴포넌트 가이드
│   │   ├── /ui                  #    shadcn/ui 기반 컴포넌트
│   │   ├── /layout              #    레이아웃 컴포넌트
│   │   ├── /common              #    공통 컴포넌트
│   │   └── /{domain}            #    도메인별 컴포넌트
│   │
│   ├── /features                # 📦 기능 모듈
│   │   └── claude.md            # → 기능 개발 가이드
│   │
│   ├── /lib                     # ⚙️ 유틸리티 & 백엔드
│   │   ├── claude.md            # → 백엔드/권한 가이드
│   │   ├── auth.ts              #    NextAuth 설정
│   │   ├── prisma.ts            #    Prisma 클라이언트
│   │   ├── rebac.ts             #    ReBAC 권한 시스템
│   │   ├── matching.ts          #    RAG 매칭 엔진
│   │   ├── /crawler             #    크롤러 시스템
│   │   ├── /documents           #    문서 분석 시스템
│   │   └── /supabase            #    Supabase 연동
│   │
│   ├── /hooks                   # 🪝 커스텀 훅
│   │   └── claude.md            # → 훅 개발 가이드
│   │
│   └── /types                   # 📝 타입 정의
│       └── claude.md            # → 타입 체계 가이드
│
├── /prisma                      # 🗄️ 데이터베이스
│   ├── claude.md                # → DB 스키마 가이드
│   ├── schema.prisma            #    Prisma 스키마
│   └── /migrations              #    마이그레이션
│
├── /scripts                     # 🔧 유틸리티 스크립트
│   └── cleanup-db.ts            #    DB 정리
│
└── /docs                        # 📚 상세 스펙 (자동 생성 금지)
    ├── /architecture            #    시스템 개요
    ├── /foundations             #    전역 규칙
    ├── /checklists              #    품질 검증
    └── /changes                 #    변경 이력
```

### 3.3 🔄 가이드 계층 구조

```
CLAUDE.md (루트 허브 - 헌법)
    │
    ├─→ /src/app/claude.md
    │       └─→ API Routes, Pages, Layouts
    │
    ├─→ /src/components/claude.md
    │       └─→ UI 컴포넌트, shadcn/ui, 접근성
    │
    ├─→ /src/features/claude.md
    │       └─→ Feature 모듈, 상태관리
    │
    ├─→ /src/lib/claude.md
    │       └─→ Auth, ReBAC, Prisma, i18n
    │
    ├─→ /src/hooks/claude.md
    │       └─→ 커스텀 훅 패턴
    │
    ├─→ /src/types/claude.md
    │       └─→ 타입 정의 체계
    │
    └─→ /prisma/claude.md
            └─→ DB 스키마, 마이그레이션
```

### 3.4 📋 작업별 가이드 선택

| 작업 유형 | 1차 참조 | 2차 참조 |
|----------|---------|---------|
| 새 페이지 생성 | `/src/app/claude.md` | `/src/components/claude.md` |
| API 엔드포인트 | `/src/app/claude.md` | `/src/lib/claude.md` |
| UI 컴포넌트 | `/src/components/claude.md` | 루트 `CLAUDE.md` (버튼 규칙) |
| 새 기능 모듈 | `/src/features/claude.md` | `/src/lib/claude.md` |
| DB 스키마 변경 | `/prisma/claude.md` | `/src/lib/claude.md` |
| 권한 시스템 | `/src/lib/claude.md` | `/prisma/claude.md` |
| 문서 분석/AI | `/src/lib/claude.md` | `/src/app/claude.md` (API) |

---

## 4. 전역 원칙 (변경 금지)

### 4.1 디자인 토큰 원칙


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flowcoder2025/Konarae_FlowCoder](https://github.com/flowcoder2025/Konarae_FlowCoder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
