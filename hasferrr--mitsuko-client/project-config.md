---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Mitsuko** is an AI-powered subtitle translation and audio transcription frontend application. It connects to a separate backend (chizuru-translator) for AI processing. Key features include subtitle translation, audio transcription, context extraction, and batch processing.

## Commands

```bash
bun dev              # Start development server
bun build            # Production build (rarely used)
bun typecheck        # Type checking (use this instead of build)
bun lint             # Run ESLint
bun test             # Run all tests (uses Bun's built-in test runner)
bun test <file-path> # Run specific test (e.g., bun test src/lib/parser/cleaner.test.ts)
```

## Tech Stack

- **Framework**: Next.js 16 (App Router) with React 19
- **Runtime**: Bun (not Node.js)
- **State**: Zustand (client state), TanStack Query (server state)
- **Persistence**: Dexie.js (IndexedDB) for offline-first project data
- **UI**: Radix UI primitives + Tailwind CSS
- **Auth**: Supabase Auth
- **Analytics**: Sentry (error tracking), PostHog (product analytics)
- **Payment**: Midtrans/Snap
- **Drag & Drop**: @dnd-kit
- **Backend**: External API at `NEXT_PUBLIC_API_URL` (chizuru-translator)

## Architecture

### Route Groups
- `src/app/(landing)/` - Public pages (home, pricing, blog, privacy, terms, changelog)
- `src/app/(main)/` - Authenticated app (translate, transcribe, batch, project, extract-context, history, library, cloud, dashboard, tools)

### Key Directories
- `src/stores/` - Zustand stores:
  - `settings/` - Basic, Advanced, Local, Whisper, and Batch settings stores
  - `services/` - Translation, transcription, extraction service stores (use `createServiceSlice` factory)
  - `factories/` - Store factory functions (e.g., `createServiceSlice` for shared Set + AbortController pattern)
  - `utils/` - Shared store utilities (e.g., `copySettingsKeys` for settings copy/reset)
  - `data/` - Project data caches
  - `ui/` - UI state stores (history, tools, theme, session, upload, etc.)
- `src/lib/db/` - Dexie database schema, migrations, and CRUD operations (only layer that defines `db` transaction/CRUD functions; components and hooks must NOT import `db` directly — use store methods instead)
- `src/lib/subtitles/` - SRT/ASS/VTT parsers and generators
- `src/lib/parser/` - AI response parsing and cleaning
- `src/lib/api/` - Backend API integration (streaming, credit management)
- `src/lib/utils/` - Utility modules split by domain (`cn.ts`, `format.ts`, `math.ts`, `audio.ts`, `file.ts`, `async.ts`, `done-tag.ts`); barrel re-exported from `src/lib/utils.ts`
- `src/lib/transcription/` - Transcription utilities: subtitle generation from word-level timestamps with CPS optimization
- `src/lib/translation/` - Translation utilities: context memory strategies (full, minimal, split) for AI completion requests
- `src/components/` - Feature components organized by domain (translate, batch, transcribe)
- `src/components/ui/` - Shadcn/Radix UI primitives (auto-generated, avoid editing)
- `src/components/ui-custom/` - Shared composed components (combo-box, virtualized-list, drag-and-drop, various dialogs)
- `src/components/shared/` - Cross-feature shared components (card-grid-selection-bar, confirm-dialogue)
- `src/components/transcribe/` - Transcription UI split into sub-components (upload-tab, select-tab, controls, result-panel, next-actions) composed by `transcription-main.tsx`
- `src/types/` - TypeScript interfaces for Project, Translation, Transcription, etc.
- `src/constants/` - App constants and defaults
- `src/constants/model-collection.ts` - AI model definitions (free and paid models with filtering utilities)
- `src/hooks/` - Custom hooks organized by domain:
  - `handler/` - Service operation handlers (translation, transcription, extraction)
  - `batch/` - Batch processing hooks for files, selection, and handlers
  - `project/` - Project data fetching and management
  - Root-level utilities (scroll-to-top, mobile detection, auto-scroll)

### Project Architecture

A **Project** is the central organizational unit. It contains:

```
Project
├── translations: Translation[]      # Subtitle translation histories
├── transcriptions: Transcription[]  # Audio transcription histories (settings stored on entity)
├── extractions: Extraction[]        # Context extraction histories
└── Default Settings (per feature)
    ├── Translation: defaultTranslationBasicSettingsId + defaultTranslationAdvancedSettingsId
    ├── Extraction: defaultExtractionBasicSettingsId + defaultExtractionAdvancedSettingsId
    └── Transcription: defaultTranscriptionId (stores settings directly on transcription)
```

**Entity Relationships:**
- `Translation` - Single subtitle file translation with `basicSettingsId` and `advancedSettingsId`
- `Transcription` - Audio-to-text with word-level timestamps, segments, and settings stored directly on entity (language, selectedMode, models, customInstructions)
- `Extraction` - Context analysis from subtitles with `basicSettingsId` and `advancedSettingsId`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hasferrr/mitsuko-client](https://github.com/hasferrr/mitsuko-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-09 -->
