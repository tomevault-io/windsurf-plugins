---
trigger: always_on
description: > This file provides context and instructions for AI coding agents.
---

# AGENTS.md — Rayburst Connect

> This file provides context and instructions for AI coding agents.
> For human contributors, see [README.md](README.md) and [CONTRIBUTING.md](docs/CONTRIBUTING.md).

> [!IMPORTANT]
> **All changes must meet industrial-grade quality.** Keep the codebase lean: plain functions over classes, one source of truth for every fact, strict TypeScript (no `any`, justify every `as` cast), and verification proportional to the changed behavior.

---

## A. Project Architecture

| Layer               | Stack                                          |
| ------------------- | ---------------------------------------------- |
| **Framework**       | WXT 0.20 (Manifest V3) + Vue 3 Composition API |
| **UI**              | Naive UI + plain CSS custom properties         |
| **Validation**      | Zod 4 (`lib/schema.ts` is the SSOT)            |
| **Testing**         | Vitest + WXT `fakeBrowser` polyfill            |
| **Build**           | Vite (via WXT) → `.output/chromium-mv3/`       |
| **Package Manager** | pnpm 10                                        |

### Key File Paths

```
entrypoints/
├── background.ts                # Service worker — orchestrator wiring, listeners, storage sync
├── content.ts                   # Content script — magnet/ed2k/thunder link interception
├── popup/App.vue                # Browser action popup — status, speed, dashboard
└── options/App.vue              # Full-page settings — one staged-snapshot state model

lib/
├── schema.ts                    # Zod schemas — single source of persisted types + defaults
├── storage.ts                   # Schema-validated load/save over browser.storage.local
├── api.ts                       # DesktopApiClient (ky) + error taxonomy + checkConnection
├── desktop.ts                   # Native Messaging activation + API readiness
├── browser.ts                   # Permissions, context menu, notifications, webRequest types
├── backup.ts                    # Settings backup export/import
├── diagnostics.ts               # Sanitized, serialized diagnostic journal
├── file-extensions.ts           # File extension normalization/matching
└── download/
    ├── contracts.ts             # Consumer validation for ordinary desktop handoff
    ├── pending.ts               # Bounded native local journal for unresolved request IDs
    ├── orchestrator.ts          # Interception flows: automatic, Firefox response, explicit
    ├── chromium-takeover.ts     # Synchronous Chromium cancellation handoff
    ├── filter.ts                # Filter pipeline (pure function stages)
    ├── request-context.ts       # Captured request headers (TTL store)
    ├── duplicate-guard.ts       # Duplicate download reservation window
    ├── firefox-response.ts      # Firefox attachment response parsing
    └── url.ts                   # URL/filename extraction (incl. presigned CD params)

shared/
├── theme.ts                     # Entire theme system: schemes, M3 CSS vars, bootstrap, useAppTheme()
├── i18n/
│   ├── engine.ts                # I18nEngine (worker) + createI18n/useI18n (Vue)
│   ├── locales.ts               # Shared locale registry
│   ├── dictionaries.ts          # Translation data via virtual:locales
│   └── locales-plugin.ts        # Vite plugin aggregating public/_locales/ at build time
├── json.ts                      # jsonClone / deepEqual for JSON-safe data
├── use-polling.ts               # Visibility-aware polling with backoff
├── manifest.ts                  # Manifest builder (per-browser permissions)
└── components/                  # BrandLogo, CollapsePanel

__tests__/                       # Behavior-level unit + integration tests
public/_locales/                 # Chrome i18n bundles (27 languages, SSOT)
```

### A′. Download Filter Pipeline

`lib/download/filter.ts` evaluates candidates through ordered pure-function stages; the
first non-null verdict wins, default is intercept:

enabled → self-trigger → interception-scope → scheme → browser-ownership → site-rule →
file-extension-rule → minimum-file-size

### A″. Persistence Model

- Persisted settings live ONLY in `lib/schema.ts`. Types are `z.infer`, defaults come from
  `Schema.parse({})` — never hand-write a default twice.
- Every parse helper accepts `unknown` and never throws; corrupt fields collapse to
  defaults, invalid array entries are dropped.
- Unresolved download requests use bounded native local storage and strict consumer contracts;
  never repair a malformed request into a new download. See [DOWNLOADS.md](docs/DOWNLOADS.md).
- `lib/storage.ts` validates on read AND write (writes are re-parsed, which also strips
  Vue reactivity proxies).

### Adding a New Storage Field

1. Add the field (with `.catch(default)`) to the schema in `lib/schema.ts` — done: type + default exist.
2. Wire it into the UI and/or background logic.
3. Add i18n keys to all 27 locales (see Section D).
4. Extend `__tests__/unit/storage-schema.test.ts`.

---

## B. Version Management

**`package.json` is the single source of truth.** WXT reads `version` from here for the manifest.

Always bump via the script — never edit the version manually:

```bash
./scripts/bump-version.sh 1.0.6
```

---

## D. i18n / Locale Operations

### Rules


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AnInsomniacy/rayburst-connect](https://github.com/AnInsomniacy/rayburst-connect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
