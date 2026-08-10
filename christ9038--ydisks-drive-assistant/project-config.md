---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Ydisks批量转存助手 — a Tauri v2 desktop app for batch-transferring cloud drive shares (夸克/百度/迅雷网盘) and creating short links via a Ydisks backend API.

**Stack:** React 19 + TypeScript + Vite (frontend), Rust + Tokio + SQLite (backend), Tauri v2.

## Common Commands

| Task | Command |
|------|---------|
| Web dev (no Rust) | `npm run dev` — serves on `http://localhost:5178` |
| Desktop dev | `npm run tauri:dev` — builds Rust + launches app window |
| Web build | `npm run build` — runs `tsc && vite build`, outputs to `dist/` |
| Desktop build (macOS app) | `npm run tauri:build -- --bundles app` — builds release `.app` |
| Frontend tests | `npm test` — runs Vitest once (`npm run test:watch` to watch) |
| Backend tests | `cargo test` (run in `src-tauri/`) — unit + ignored live tests |
| Backend check | `cargo check` / `cargo build` (run in `src-tauri/`) |

**Note:** The frontend depends on Tauri APIs (`@tauri-apps/api/core`). Many features (provider login, share transfer, folder listing, backend API calls) only work inside the Tauri desktop runtime. When running `npm run dev` in a browser, `isTauriRuntime()` returns false and the UI shows "请在桌面端运行" messages; the SQLite-backed persistence also falls back to browser `localStorage` in dev.

## Architecture

### Frontend (`src/`)

- **`src/App.tsx`** (~700 lines) — thin App shell: page routing + modal orchestration + provider-account CRUD handlers (`handleProviderAdd/Login/Check/Clear/Delete`) + confirm-dialog state + account/theme persistence effects. All state-machine logic delegated to hooks. Pure logic lives in `src/lib/`, presentational/interactive components in `src/components/`, pages in `src/pages/`, state domains in `src/hooks/`. Pages receive data + callbacks via props from `App`.
- **`src/lib/`** — pure, tested modules:
  - `constants.ts` — platform keys, labels, status labels, default folder refs, storage keys.
  - `guards.ts` — type guards + judgments (`isProviderKey`, `isActiveQueueStatus`, `providerOfAccountTypeName`, `normalizeStoredProviderAccounts`, …).
  - `parsing.ts` — share-URL detection / access-code extraction / text & CSV resource parsing.
  - `csv.ts` — `buildHistoryCsv` + `csvEscape`.
  - `format.ts` — `formatBackendError` / `extractXunleiReviewState`.
  - `mappers.ts` — `ProviderAccount`↔`RustAccountRow`, `BatchRecord`↔`RustBatchRow`, `HistoryRecord`↔`RustHistoryRow`, `buildTargetsFromBackendAssets`.
  - `engine.ts` — local batch engine (`advanceLocalBatch`, `normalizePlatformQueues`, `createQueueItemsFromDraft`, factories).
  - `storage.ts` — `localStorage` read/write/clear for accounts / theme / API key.
  - `bridge.ts` — typed `invoke` wrappers + persistence/asset/loading helpers + `isTauriRuntime` / `waitForNextPaint` / `downloadHistoryCsv` / `openLogDir`. The only frontend module that touches `@tauri-apps/api/core` directly.
  - `batchReducer.ts` — pure batch state-machine transitions (`reduceQueueItemUpdate` progress/status derivation, `reducePauseBatch`/`reduceContinueBatch`/`reduceRetryBatch`/`reduceRetryQueueItem`, `batchToHistory`, `hasSamePlatformBlocker`). Unit-tested; `App` calls these instead of inlining the logic.
  - `types.ts` — shared interfaces & Rust row types used across lib + components + pages.
- **`src/components/`** — reusable React components (presentational primitives in `ui.tsx` are `React.memo`-wrapped so the 1.6s engine poll doesn't re-render them when their primitive props are unchanged):
  - `ui.tsx` — primitives (`SectionTitle`, `Field`, `SelectField`, `ThemeModeControl`, `Metric`, `StatusPill`, `TaskPill`, `BatchStatusPill`, `Progress`, `Callout`, `EmptyState`).
  - `tables.tsx` — `ImportTable`, `QueueTable`, `BatchLogPanel`, `ResultsTable`.
  - `modals.tsx` — `ConfirmDialog`, `CookiePasteModal`, `XunleiQrModal`, `XunleiPasswordModal`, `FolderPickerModal`, `DirectorySelectField`.
  - `fields.tsx` — `ShortLinkTargets`.
- **`src/pages/`** — page components rendered by `App` via switch on `activePage`:
  - `TasksPage.tsx`, `StatusPage.tsx` (+ inline `BatchMonitorTable`, renders `BatchDetailPage.tsx`), `BatchDetailPage.tsx`, `HistoryPage.tsx`, `SettingsPage.tsx` (+ inline `ProviderAccountsPanel`/`ProviderCard`).
- **`src/hooks/`** — state domains extracted from `App` (each owns its slice of state + effects, coupled domains receive callbacks as params):
  - `useThemeMode.ts` — `themeMode` state + apply/persist effect.
  - `useToast.ts` — `toast` state + auto-dismiss effect (exports `ToastState`).
  - `useApiConnection.ts` — API key / connection status / quota / backend assets (accounts/channels/domains/shortlink targets) + `detectApiKey` / `setApiKeyDraft` / `clearApiKeyImmediately` + startup auto-detect. Takes `setToast`.
  - `useCookieModal.ts` — cookie-paste modal state + open/close/submit. Takes `providerAccounts` + `updateProviderAccount`.
  - `useXunleiLogin.ts` — 迅雷 scan-code + password-login modals (12 states) + 2s scan poll effect. Takes `providerAccounts` + `updateProviderAccount` + `setToast`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Christ9038/Ydisks-drive-assistant](https://github.com/Christ9038/Ydisks-drive-assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
