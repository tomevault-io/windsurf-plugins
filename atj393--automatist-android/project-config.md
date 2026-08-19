---
trigger: always_on
description: > **⚠️ HISTORICAL / SUPERSEDED.** This file is an archived snapshot of an earlier version of
---

# Automatist

> **⚠️ HISTORICAL / SUPERSEDED.** This file is an archived snapshot of an earlier version of
> the project notes. It is **not** canonical — see `CLAUDE.md`. It predates the free/open-source
> migration and describes removed architecture. In the current app: the source is **Apache-2.0**
> (free, open source); Google Play Billing, the Pro tier, `ProductAccess`/entitlement, the Upgrade
> UI, and the one-active-workflow limit have been **removed**; the Room database is **v17**.
> Treat any billing/Pro/proprietary-licence content below as historical only.

## Project Identity

**Type:** Workflow-first AI utility (not chatbot, not agent)

**Purpose:** Transform text content (articles, meeting notes, RSS feeds) into structured outputs (summaries, social posts, briefs) using pluggable AI providers — all on-device, no backend.

**Target Users:** Professionals who consume content and need quick, polished outputs for sharing or review.

**Core Principles:**
- **Output-first** — every workflow produces a concrete, usable artifact
- **Template-driven** — transform types define the output shape, not free-form chat
- **Manual-final-action** — user always reviews before copy/share/save; no auto-posting
- **No scraping / no background automation** — only the Morning Brief worker runs in background; RSS fetches snippets only (title + description), never full articles
- **Privacy-first** — no backend server; all data stays on-device; API keys stored locally

---

## Technical Stack

| Layer | Technology |
|-------|-----------|
| Language | Kotlin |
| UI | Jetpack Compose + Material3 |
| DI | Hilt |
| Local DB | Room |
| Preferences | DataStore |
| Background | WorkManager |
| HTTP | Retrofit + OkHttp |
| Async | Kotlin Coroutines + Flow |
| Serialization | Gson (Retrofit) + kotlinx.serialization (DataStore) |
| Billing | Google Play Billing Library (v7.0.0) |
| Cloud Sync | Google Drive API (appDataFolder) |
| Auth | Google Play Services Auth |
| On-device AI (system) | Google AI Edge AICore (0.0.1-exp01) — Gemini Nano via Android system service |
| On-device AI (download) | MediaPipe LLM Inference (tasks-genai) — Gemma 3 1B int4 via app download |

**Build:** Gradle KTS, version catalog (`libs.versions.toml`), compile/target SDK 35, min SDK 26, Java 17.

---

## Architecture

**Single-activity** app with Jetpack Compose navigation.

```
┌─────────────────────────────────────────────────┐
│  feature/                                       │
│   article/  meeting/  brief/  dashboard/        │
│   history/  vault/    notes/  workflow/          │
│   upgrade/  sync/                               │
├─────────────────────────────────────────────────┤
│  domain/          (interfaces + models)         │
│   models/  providers/  repositories/  access/   │
│   actions/  engine/  templates/  readiness/     │
│   sync/  workflow/  offline/                    │
├─────────────────────────────────────────────────┤
│  data/            (implementations)             │
│   local/  network/  providers/  repositories/   │
│   access/  billing/  sync/  offline/            │
├─────────────────────────────────────────────────┤
│  platform/        (OS-level concerns)           │
│   automation/  security/  notifications/        │
│   scheduling/  onboarding/                      │
├─────────────────────────────────────────────────┤
│  di/              (Hilt modules)                │
│  ui/              (theme + navigation)          │
└─────────────────────────────────────────────────┘
```

**Pattern:** MVVM — each screen has a `@HiltViewModel` with `StateFlow`; UI observes state and dispatches intents.

**Provider routing:** `TransformProviderRouter` implements `ArticleTransformProvider` and delegates to the active provider (Fake/OpenAI/Anthropic/Gemini/LOCAL_AI) based on profile resolution order.

---

## Workflows

### 1. Article Transformer

- **Input:** Shared or pasted text
- **Transform types:** `SUMMARY` | `THREAD` | `PRO_POST`
- **Actions:** Preview, Copy, Share, Save to history
- **State flow:** Input → Loading → Success / Error

### 2. Meeting Strategist

- **Input:** Title + meeting notes
- **Transform types:** `MEETING_BRIEF` | `STRATEGIC_QUESTIONS`
- **Actions:** Preview, Save, Share

### 3. Morning Brief (automation)

**Config flow:**
1. Trigger: `EVERY_N_HOURS` | `DAILY_AT_HOUR`
2. Feeds: `List<String>` (RSS URLs)
3. Output type: `SUMMARY` | `SOCIAL_POST` | `BULLET_INSIGHTS` | `CUSTOM`
4. Platforms (if SOCIAL): `LINKEDIN` | `X` | `FACEBOOK` | `INSTAGRAM` | `THREADS`
5. Notify: `true` / `false`

**Config model fields:**
```
feedUrls[]
outputType
selectedPlatforms[]
notify
scheduleType
intervalHours?
dailyHour?
maxItems = 5
customInstruction?
```

**Worker pipeline (SynthesizerWorker):**
1. Load config from DataStore
2. Fetch RSS via OkHttp
3. Parse XML (supports RSS + Atom) — title + snippet only
4. Sort by recency
5. Pick top N (<=5)
6. Build payload (limit ~6k–10k chars, truncate per item to 500 chars)
7. Call AI provider via router
8. Generate output
9. Save to history (`workflowType = MORNING_BRIEF`)
10. Notify if enabled

**Limits:**
- No full article fetch — snippet only
- Max ~6k–10k chars total payload
- Truncate per item

**Output rules:**
| Type | Style |
|------|-------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [atj393/automatist-android](https://github.com/atj393/automatist-android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
