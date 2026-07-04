---
trigger: always_on
description: A Google Docs sidebar add-on for Islamic scholars to search and insert Quranic ayat. MVP is Quran-only (Hadith is post-MVP).
---

# Tasneef AI — Google Docs Add-on

A Google Docs sidebar add-on for Islamic scholars to search and insert Quranic ayat. MVP is Quran-only (Hadith is post-MVP).

> **Deep architecture reference:** see `ARCHITECTURE.md` for file structure, include order, data flows, and test commands.

---

## Stack & Constraints

- **Server:** Google Apps Script. All server files are `.js` under **`src/`** (clasp `rootDir`) — clasp pushes them as `.gs`. Never create `.gs` files directly.
- **Client:** HTML/CSS/JS via HtmlService under **`src/`** (e.g. `src/client/`, `src/sidebar/`). No npm, no bundler, no import/require, no ES modules.
- **Communication:** Client ↔ server via `google.script.run`.
- **Project size limit:** ~2MB (code only; all data is external).

---

## Data Architecture

### Quran text — client-side only

All Quran text, metadata, and translations are fetched **client-side** via `makeClientCache` (browser `fetch()`). The server never loads or caches Quran data.

Canonical URLs live in `src/sidebar/js/quran-caches.html`:

| Feed | URL |
|------|-----|
| Arabic display (imlaei-script) | `tarazis.github.io/tasneef-data/quran/imlaei-script.json` |
| Search index (imlaei-simple) | `tarazis.github.io/tasneef-data/quran/imlaei-simple.json` |
| Surah metadata | `tarazis.github.io/tasneef-data/quran/quran-metadata-surah-name.json` |
| English translation | `tarazis.github.io/tasneef-data/quran/en-sahih-international-simple.json` |

**Legacy naming:** Code uses "uthmani" identifiers (`UTHMANI_URL`, `ensureUthmaniCache`, `lookupUthmaniAyah`, `arabicStyle: 'uthmani'`, `textUthmani`) — these all reference the **imlaei-script** source, not traditional Uthmani rasm.

**Translation JSON structure:** Flat object keyed by `"surah:ayah"` with `{t: "text"}` values. Inspect actual structure before writing code against it.

### Arabic normalization

`src/NormalizeArabic.js` (server) and `src/client/normalizeArabic.html` (client) **must stay in sync** — parity enforced by `tests/normalizeArabic.test.js`. Strips tashkeel/diacritics, normalizes alef variants (آ أ إ ٱ → ا). Server copy exists solely for parity testing; all production search runs client-side.

### Typography

Arabic ayah text in Google Docs is always **Amiri**, regular weight, not bold — `src/FormatService.js` enforces this regardless of client payload. The sidebar loads Amiri via Google Fonts CSS for card preview. If a selected font fails, fall back to Amiri and show a toast.

---

## AI Search Pipeline

### Claude — intent classifier only

- Model: `claude-haiku-4-5-20251001`, temperature 0.
- Claude classifies user intent and returns JSON actions (`fetch_ayah`, `exact_search`, `semantic_search`, `clarify`) — **never Quranic text**.
- Every reference from Claude must be validated against local data before display.

### RAG semantic search flow

1. **Classification:** `performAISearch` sends user query to Claude. Claude returns action JSON with `queries` array, optional `filter.surah`, and `rag_supported` flag.
2. **Routing:** If `rag_supported === false`, uses Claude references directly. Otherwise tries RAG with fallback to Claude references on error/empty result.
3. **Retrieval:** Embeds queries via OpenAI (`text-embedding-3-small`), queries Pinecone in parallel (`topK=20`, optional surah filter). Translation JSON fetched in same `fetchAll`.
4. **Merge & filter:** Deduplicate across query expansions (keep highest score per ayah), apply `RAG_SCORE_THRESHOLD = 0.35`. If nothing passes threshold, fall back to Claude references.
5. **Rerank:** If ≥3 candidates and Claude key available, Claude reranks the candidate pool (max 20).
6. **Finalize:** `_finalizeRagAyahRefs_` validates numeric ranges (surah 1–114, ayah ≥ 1), deduplicates, caps results. Merges consecutive ayahs into range groups.
7. **Client validation (hallucination guard):** Sidebar resolves references against local caches — drops any ref where surah doesn't exist, ayah exceeds surah count, or Arabic text missing from cache.

### Daily AI quota

- Default-tier per-user cap is read from Script Property `ai_limit` (positive integer; defaults to 20 if unset or invalid), enforced in `src/SettingsService.js` / `src/ClaudeAPI.js`.
- Counter stored in User Properties as JSON (`{count, date}`). Resets on first query of new UTC day.
- Dev users listed in Script Property `users_dev` (comma-separated emails) have unlimited quota.
- Pro users listed in Script Property `users_pro` (comma-separated emails) use `ai_limit_pro_users` instead of `ai_limit`. Dev membership wins if a user appears in both lists.

### API keys

All API keys (Claude, OpenAI, Pinecone, optional Google Fonts Web API) stored in **Script Properties** via `PropertiesService.getScriptProperties()`. Never exposed in source code (`google_fonts_api_key` is injected into the sidebar HTML at render time only when set).

---

## Hard Rules

1. **All Quranic Arabic text and English translations come from GitHub Pages JSON. Never from Claude. Never generated.**
2. **Claude is an intent classifier only.** It returns `{surah, ayah}` pairs, Arabic search queries, or RAG search framing. We resolve real text client-side.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tarazis/tasneef-ai](https://github.com/tarazis/tasneef-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
