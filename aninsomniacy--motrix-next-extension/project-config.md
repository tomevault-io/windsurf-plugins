---
trigger: always_on
description: > This file provides context and instructions for AI coding agents.
---

# AGENTS.md — Motrix Next Extension

> This file provides context and instructions for AI coding agents.
> For human contributors, see [README.md](README.md) and [CONTRIBUTING.md](docs/CONTRIBUTING.md).

> [!IMPORTANT]
> **All changes must meet industrial-grade quality.** Enforce DRY (extract services/utilities over duplication), strict TypeScript (no `any`, justify every `as` cast), dependency injection for all Chrome API surfaces, and full verification (`vue-tsc` + tests pass) before completion.

---

## A. Project Architecture

| Layer               | Stack                                           |
| ------------------- | ----------------------------------------------- |
| **Framework**       | WXT 0.20 (Manifest V3) + Vue 3 Composition API  |
| **UI**              | Naive UI + Tailwind CSS 4                       |
| **Validation**      | Zod 4 (storage schemas)                         |
| **Testing**         | Vitest (335 tests, DI-based — no browser mocks) |
| **Build**           | Vite (via WXT) → `.output/chrome-mv3/`          |
| **Package Manager** | pnpm 10                                         |

### Key File Paths

```
entrypoints/
├── background.ts               # Service worker — orchestrator, listeners, heartbeat polling
├── content.ts                   # Content script — magnet/torrent link detection
├── popup/
│   ├── App.vue                  # Browser action popup — status, speed, task dashboard
│   └── components/              # PopupHeader, SpeedBar, StatDashboard
└── options/
    ├── App.vue                  # Full-page settings — connection, behavior, rules, appearance
    └── composables/
        ├── use-appearance.ts     # Theme and color scheme switching
        ├── use-connection-test.ts # RPC connection testing
        ├── use-diagnostics.ts    # Diagnostic log viewer
        └── use-site-rules.ts     # Per-site interception rules CRUD

lib/                             # Core logic — all services use dependency injection
├── download/
│   ├── orchestrator.ts          # Download interception entry point, retry-after-wake
│   ├── filter.ts                # 6-stage filter pipeline (see Section A′)
│   └── metadata-collector.ts    # Filename, cookie, referer extraction
├── rpc/
│   └── aria2-client.ts          # aria2 JSON-RPC 2.0 client with retry and auth
├── services/
│   ├── connection.ts            # Heartbeat polling, connect/disconnect state
│   ├── completion-tracker.ts    # Poll active tasks, detect completion, notify
│   ├── context-menu.ts          # Right-click "Download with Motrix Next"
│   ├── download-bar.ts          # chrome.downloads.setUiOptions (Chrome 115+)
│   ├── notification.ts          # Desktop notification builder
│   ├── theme.ts                 # Material You theme resolution
│   └── wake.ts                  # motrixnext:// protocol launcher
├── protocol/
│   └── launcher.ts              # Protocol URL builder and tab management
└── storage/
    ├── schema.ts                # Zod schemas + safe parse functions (see Section C)
    ├── storage-service.ts       # Typed get/set wrappers over chrome.storage.local
    ├── migration.ts             # Forward-only versioned schema migration (see Section C′)
    └── diagnostic-log.ts        # Capped event log with severity levels

shared/
├── i18n/
│   ├── engine.ts                # Compile-time i18n with positional $placeholder$ support
│   ├── dictionaries.ts          # Locale module registry (26 languages)
│   └── locale-modules.d.ts      # Virtual module type declarations for locale:* imports
├── types.ts                     # TypeScript interfaces (RpcConfig, DownloadSettings, etc.)
├── constants.ts                 # Default configs, timing constants, URL schemes
├── color-schemes.ts             # Material You color scheme definitions
├── url.ts                       # URL validation and scheme classification
├── thunder.ts                   # Thunder (迅雷) link decoder
├── errors.ts                    # Typed error constructors
├── use-color-scheme.ts          # Color scheme composable with dynamic CSS injection
├── use-polling.ts               # Generic polling composable with lifecycle management
├── use-preference-form.ts       # Two-way preference form binding composable
└── use-theme.ts                 # System/light/dark theme detection composable

__tests__/
├── unit/                        # 28 isolated service test files
└── integration/                 # End-to-end interception flow

public/_locales/                 # Chrome i18n message bundles (26 languages, see Section D)

.github/workflows/
├── ci.yml                       # Quality gate: compile → test → lint → i18n → format → build
└── release.yml                  # Package → upload to GitHub Release → publish to stores
```

### A′. Download Filter Pipeline

The 6-stage filter (`lib/download/filter.ts`) evaluates downloads in strict order:

| Stage | Gate               | Pass                               | Reject                             |
| ----- | ------------------ | ---------------------------------- | ---------------------------------- |
| 1     | Global toggle      | `enabled === true`                 | Skip — extension disabled          |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AnInsomniacy/motrix-next-extension](https://github.com/AnInsomniacy/motrix-next-extension) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
