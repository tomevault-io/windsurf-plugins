---
trigger: always_on
description: WorldScript Studio is an AI-powered creative writing application built as an offline-first PWA. It combines a React 19 SPA with Google Gemini AI integration, IndexedDB persistence, and optional Tauri desktop packaging.
---

# Copilot Instructions — WorldScript Studio

## Project Overview

WorldScript Studio is an AI-powered creative writing application built as an offline-first PWA. It combines a React 19 SPA with Google Gemini AI integration, IndexedDB persistence, and optional Tauri desktop packaging.

**Live:** `https://qnbs.github.io/WorldScript-Studio/`

**Documentation map:** [`README.md`](../README.md#-documentation-hub) § Documentation Hub lists every maintainer `.md` guide (see also [`AUDIT.md`](../AUDIT.md)).

## Architecture

### Tech Stack

- **Frontend:** React 19 + TypeScript (strict mode), Vite 8
- **State:** Redux Toolkit 2.x + Redux-Undo, feature-sliced design
- **Styling:** Tailwind CSS 4.x with CSS custom properties for theming
- **AI:** Google Gemini API via `@google/genai`, multi-provider abstraction (`aiProviderService.ts`)
- **Storage:** Dual IndexedDB via `services/storage/` (decomposed from `dbService.ts` in Phase 1); LZ-String compression + AES-256-GCM key encryption; `storageEncryptionService.ts` for at-rest IDB encryption (B-1, v1.19.0); `storageService.ts` switches browser vs Tauri filesystem
- **Collaboration:** Yjs + `packages/collab-transport` (vendor fork of y-webrtc 10.3.0, RTCDataChannel E2E AES-256-GCM) for P2P real-time editing
- **Desktop:** Tauri 2 (optional)
- **Package manager:** pnpm@11.x
- **Testing:** Vitest + @testing-library/react (unit), Playwright (E2E)

### Directory Structure

```text
app/              → Redux store, hooks (useAppDispatch/useAppSelector), listener middleware, utils
components/       → React view components (one per view)
  ui/             → Reusable design system primitives (Button, Modal, Card, Toast, etc.)
contexts/         → React context providers (one per major view + I18nContext + CommandExecutorContext)
features/         → Redux Toolkit slices: project, settings, status, writer, versionControl, featureFlags
hooks/            → Custom hooks with view business logic (one hook per view)
services/         → External adapters: geminiService, aiProviderService, dbService (dual IndexedDB + migration), storageService, collaborationService; **ai/** (aiModeService — execution modes, aiPolicy, aiRetry; **providers/** — openrouterProvider with circuit breaker); **copilot/** (heuristicEngine 8 rules, insightGenerator, copilotContextService, actionApplier); **commands/** (palette registry); **keyboard/** (shortcut matching); **help/** (doc retrieval for AI); **settingsExchange** (settings JSON)
locales/          → i18n source files — de/en/es/fr/it (core) + ar/he/fa (RTL Beta) + el/ja/pt/zh/fi/sv/hu/is/eu/ru/ko (Beta) × 21 JSON modules (19 locales; see the README badge for the live key count)
public/locales/   → i18n runtime files served at BASE_URL
tests/            → Unit + E2E tests (Vitest + Playwright)
types/            → Additional TypeScript type definitions
types.ts          → Core shared interfaces and types
```

### Key Patterns

1. **View = Component + Hook + Context:** Each major view (e.g., Dashboard) has:
   - `components/Dashboard.tsx` — Pure rendering
   - `hooks/useDashboard.ts` — Business logic, Redux selectors, thunk dispatches
   - `contexts/DashboardContext.ts` — React context to pass hook return to child components

2. **Redux:** All state mutations go through Redux slices. Async operations use `createAsyncThunk`. Side effects (auto-save) run in the listener middleware. The `project` slice is wrapped with `redux-undo` for undo/redo.
   - `features/project/aiThunkUtils.ts` provides a reusable deduplicated async-thunk wrapper for AI requests.

3. **AI Service:** `services/ai/index.ts` is the canonical entry (Vercel AI SDK layer). `geminiService.ts` is the primary legacy adapter. `aiProviderService.ts` provides the multi-provider abstraction (Gemini, OpenAI, OpenRouter, Claude, Grok, Ollama, WebLLM, ONNX, Transformers.js). **AI Execution Modes** (`aiModeService.ts`): `hybrid` | `cloud` | `local` | `eco` — control routing strategy, persisted to `settings.aiMode`. **OpenRouter** (`services/ai/providers/openrouterProvider.ts`): Cloud 5 in the routing chain, circuit breaker (4×429 → 5 min pause), free-tier catalog (`:free` suffix models). All cloud AI calls gated by `assertCloudAiAllowed` from `aiPolicy.ts`; retries via `withTransientRetry` in `aiRetry.ts`.

4. **Storage:** `dbService.ts` wraps **dual** IndexedDB (state vs data stores, legacy migration) with compression (LZ-String for payloads > 10KB) and encryption (AES-256-GCM for API keys). `storageService.ts` provides a unified interface that auto-detects IndexedDB vs Tauri filesystem.

5. **i18n:** Custom React Context system in `I18nContext.tsx`. Translation keys use dot notation (`common.save`, `dashboard.wordCount`). All user-facing strings MUST be translation keys, never hardcoded text.

6. **Code Splitting:** All views are lazy-loaded in `App.tsx` via `React.lazy()`. Heavy dependencies (Konva, Leaflet, react-force-graph) are in separate Vite manual chunks. The export stack also uses dynamic imports for `docx` and `jszip` so large document libraries are only loaded when export actions are executed.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qnbs/WorldScript-Studio](https://github.com/qnbs/WorldScript-Studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
