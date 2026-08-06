---
trigger: always_on
description: This file applies to the entire repository. It is the first technical map an agent
---

# AGENTS.md — Vellium contributor and coding-agent guide

This file applies to the entire repository. It is the first technical map an agent
should read before changing Vellium. More specific documentation under `docs/`
explains user-facing behavior; this file focuses on implementation boundaries,
invariants, safe workflows, and verification.

## 1. Product and runtime model

Vellium is a local-first AI workbench for:

- character chat and multi-character roleplay;
- branching conversations, personas, LoreBooks, RAG, attachments, tool calls,
  reasoning traces, translation, TTS, and chat export;
- long-form writing with projects, chapters, scenes, summaries, and DOCX/Markdown;
- OpenAI-compatible, KoboldCpp, local, and custom provider adapters;
- plugins, MCP tools, endpoint extensions, and an optional desktop pet;
- a deprecated Agents workspace that remains available through Settings > Legacy.

The application has four cooperating runtimes:

1. React + TypeScript renderer in `src/`.
2. Express API in `server/`.
3. SQLite persistence through `better-sqlite3`.
4. Electron desktop shell in `electron/`.

The browser development topology is:

```text
React/Vite http://127.0.0.1:1420
        │ /api proxy
        ▼
Express   http://127.0.0.1:3002
        │
        ▼
SQLite + providers + MCP + local files
```

In packaged Electron and headless static mode, Express serves both `dist/` and
`/api`, normally from `http://127.0.0.1:3001`. Electron loads that HTTP URL; it
does not normally load the production UI directly from `file://`.

## 2. Start every task safely

Before editing:

1. Run `git status --short` and preserve unrelated user changes.
2. Identify the smallest owning subsystem and read its route, service/repository,
   client, contract, UI, and existing tests before introducing a second path.
3. Search with `rg`; do not assume a feature is absent from a large screen.
4. Treat `data/`, `output/`, `release/`, `dist/`, `dist-electron/`, generated
   bundles, and user-imported assets as user or generated state. Do not modify,
   delete, or commit them unless the task explicitly requires it.
5. Do not create commits, tags, releases, or pushes unless explicitly requested.
6. If asked to checkpoint before risky work, make the checkpoint first and keep
   the later hardening/refactor in a separate commit.

Use focused patches and avoid broad mechanical rewrites of dirty files. Existing
uncommitted changes belong to the user unless their provenance is certain.

## 3. Repository map

### Renderer: `src/`

- `src/App.tsx` — app shell, lazy screen loading, navigation groups, plugin tabs,
  Settings/Legacy routing, global display settings, and persistent Chat mounting.
- `src/features/chat/` — chat/RP UI, hooks, derived state, message rendering,
  attachments, branches, streaming, and chat export.
- `src/features/writer/` — books, character forge, projects, chapters, scenes,
  writing tasks, exports, and writing-side context.
- `src/features/characters/` — character library, import/editing, and manual order.
- `src/features/lorebooks/` — LoreBook and world-info editing/import/export.
- `src/features/knowledge/` — RAG collections and documents.
- `src/features/settings/` — providers, runtime behavior, appearance, security,
  tools, plugins, and the embedded Legacy section.
- `src/features/legacy/` — deprecated interface/Agents controls. Agents are not a
  normal top-level core tab; they are opened from Settings > Legacy when enabled.
- `src/features/agents/` — deprecated Agents implementation retained behind the
  Legacy surface. Its API/runtime still has tests and must remain functional.
- `src/features/pets/` — desktop pet configuration and renderer-side integration.
- `src/features/plugins/` — plugin frames, slots, actions, and renderer security.
- `src/components/` — reusable UI primitives and the global task manager.
- `src/shared/api/` — typed API clients split by domain. `src/shared/api.ts`
  combines them into the `api` object.
- `src/shared/types/contracts.ts` — main renderer/server JSON contracts.
- `src/shared/types/chatExport.ts` — stable chat export contract.
- `src/shared/backgroundTasks.ts` — long-running task registry used by global UI.
- `src/shared/i18n.ts` and `src/shared/locales/` — localization infrastructure and
  dictionaries for English, Russian, Chinese, and Japanese.
- `src/styles.css` — theme tokens, app chrome, responsive behavior, glass/fallback
  surfaces, and feature-specific styling beyond utility classes.

### API and persistence: `server/`

- `server/index.ts` — parses runtime options and starts the server with short
  retry handling for an occupied port.
- `server/runtimeConfig.ts` — CLI/env normalization for API-only, headless,
  loopback, public, and Basic Auth modes.
- `server/app/createApp.ts` — Express composition root, request security headers,
  origin policy, uploads, static frontend, health route, and all route mounts.
- `server/app/createApp.integration.test.ts` — broad end-to-end API regression
  suite using an isolated temporary data directory and local mock providers.
- `server/routes/` — HTTP validation and response handling by domain.
- `server/modules/chat/` — prompt/context construction, provider streaming,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tg-prplx/vellium](https://github.com/tg-prplx/vellium) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
