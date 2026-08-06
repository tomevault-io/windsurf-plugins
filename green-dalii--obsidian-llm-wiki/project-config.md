---
trigger: always_on
description: **Last Updated:** 2026-07-16
---

# LLM Wiki Plugin Project Development Standards

**Last Updated:** 2026-07-16

---

## Current Phase: v1.25.0 PR3 + PR3 follow-up #2 SHIPPED (2026-07-16) — cache-only PDF; awaiting e2e + release; v1.24.1 PATCH RELEASED 2026-07-14

**v1.25.0 scope decision (2026-07-15, user-confirmed post-pivot):**

Cache-only architecture replaces the previously-planned sidecar (`<vault>/<basename>.pdf.md`) approach.

- ✅ **PR2 redo (1-1.5 days)** — delete `pdf-ingest-orchestrator.ts`; refactor `wiki-engine.ingestPdfSource` to feed `convertPdfToMarkdown` result into `analyzeSource` via `contentOverride`; extend `PdfConversionCache` with `purgeExpired/enforceSizeLimit` (100MB / 1000-entry / 10MB-single caps + LRU-by-mtime eviction); add `converterVersion` to cache key; delete 5 dead i18n keys across 10 locales. **`prepareBatchIngest` deferred to PR3 follow-up #2.**
- ✅ **PR3 (1 day)** — settings: `writePdfMarkdownToVault` toggle in Wiki Configuration (always visible, not Advanced-bound); `forcePdfSupport` toggle in Advanced for non-NATIVE providers only (universal escape hatch — user opted in → LLM call attempted → endpoint rejection surfaces via localized Notice); CHANGELOG; ROADMAP sync.
  - Settings types + DEFAULT_SETTINGS
  - 4 i18n keys × 10 locales for both PDF toggles
  - sidecar write via direct vault.create/modify (no createOrUpdateFile cascade)
  - normalizePath for cross-platform sidecar paths
  - 3 new tests: default no-sidecar, write creates sidecar, write updates existing
  - Code-review findings applied: simplified `ingestPdfSource` comment; used `normalizePath`; avoided `createOrUpdateFile` for sidecar to prevent auto-watch cascades.
- ✅ **PR3 follow-up (2026-07-16)** — universal escape hatch + UX moves
  - `forcePdfSupport` toggle: any non-NATIVE provider allowed; toggle hidden for NATIVE providers; provider switch to NATIVE auto-resets value; `FORCE_PDF_PROVIDER_IDS` constant deleted
  - `writePdfMarkdownToVault` moved to Wiki Configuration → Wiki Folder (semantic: vault storage policy, not LLM config); always visible
  - `advancedSettingsMode` default no longer resets `forcePdfSupport`
  - 3 new tests: ollama + forcePdfSupport=true attempts LLM; deepseek same; endpoint-rejects error propagates verbatim
- ✅ **PR3 follow-up #2 (2026-07-16)** — third-party model audit fixes
  - **P0 (cross-platform cache filename safety)**: physical filename = sha256(logicalKey).slice(0,16) (Git short-hash style); fixes Windows ERROR_INVALID_NAME + POSIX unintended subpath when model contains `:` or `/`
  - **P1 (batch-start housekeeping)**: new PdfConversionCache.prepareBatchIngest() wired into runBatchIngest()
  - **P1 (PDF-shaped LLM errors → localized Notice)**: isPdfRelatedLlmError(message) classifier routes obvious PDF-rejection errors to reportSkip('unsupported-pdf')
  - **P1 (settings defaults test)**: new src/__tests__/types/settings.test.ts
  - **P2 (i18n user-perspective rewrite)**: forcePdfSupportDesc + sourceRejectedPdfUnsupported rewritten in 10 locales — drop developer jargon, speak user outcome
- ⏳ **PR4 (optional, by AkaSakana)** — Kimi Files API provider dispatch + error regex classifiers + transient-retry extension. If AkaSakana ships as follow-up PR after v1.25.0 lands, we merge after review. If schedule slips, we port ourselves (1-day).
- ⏳ **Final** — `pnpm build:dev` + HARD STOP + user e2e + push decision.

**AkaSakana PR #286 feedback adopted (2026-07-15):**
- ✅ Cache key includes `converterVersion` so prompt upgrades invalidate stale entries.
- ✅ `forcePdfSupport` is now a **universal escape hatch** (any non-NATIVE provider); default `false` (manual opt-in, NOT opt-out — many compatible endpoints don't reliably support PDF). (2026-07-16 user correction.)
- ⏳ Kimi Files API (PR4, optional contribution): upload → extract → delete, error regex classifiers, transient-retry extension. AkaSakana owns the contribution; we transfer responsibility to TA via PR #286 reply.

**Trust boundary (v1.25.0 PR3 follow-up, 2026-07-16):** the user is the authoritative source on what their endpoint supports. Pre-flight whitelist rejects violate user intent. The provider gate must attempt the call; LLM errors surface as localized Notices guiding the user to disable the toggle or check endpoint config.

**v1.24.1 PATCH release composition (2026-07-13/14 merge window):**
- ✅ Phase 1 (#271): Fix #1 #268 Tier C forceRecreate bypass
- ✅ Phase 2 (#276): page-factory.ts 1297-LOC god-class split (10 modules + 99 tests)
- ✅ Phase 3 (#277/280): Bedrock Stage 1 via bedrock-mantle (~+3 KB, zero new npm deps)
- ✅ Phase 4 (#269): #272 LM Studio no-key ingest fix
- ✅ Phase 5 (#281): 5-stage PPR seed-selection pipeline (lex → LLM keywords → local scan → LLM KB fallback → PPR) + post-e2e noise/correctness fixes. 1825 → 2060 tests.
- ✅ Phase 5.5 (#282): parseJsonResponse empty-body quiet path (`silentOnEmpty` + `throwOnEmpty`). Closes #255 + #274. 2060 → 2073 tests (+13).
- ✅ Phase 6 (#283): #258 entities-page redundant `## 基本信息` body block fix at the prompt + schema + lint layer. Closes #258. 2073 → 2080 tests (+7).

**Issues closed in v1.24.1:**
- #255 — Lint console errors (CLOSED via #282)
- #258 — entities-page `## 基本信息` drift (CLOSED via #283)
- #274 — Ollama Qwen3.5:9b no-key empty body (CLOSED via #282)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [green-dalii/obsidian-llm-wiki](https://github.com/green-dalii/obsidian-llm-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
