---
trigger: always_on
description: Use live code evidence before planning or editing. Make the smallest safe change. Preserve existing behavior unless the user explicitly asks to change it.
---

# Project Instructions

Use live code evidence before planning or editing. Make the smallest safe change. Preserve existing behavior unless the user explicitly asks to change it.

## Context

OpenOrbit — a desktop app that runs a team of AI agents locally, with a central orchestrator delegating to specialist sub-agents, each with its own tools and access to the user's files, apps, and Google account.

**Status:** implemented and under active development on `develop-ai`. ~290 source files across an Electron main process, a preload bridge, and a React renderer. The source tree is currently untracked in git (only `README.md`, `LICENSE`, `.gitignore`, and this file are committed) — `README.md` still describes the project as "idea phase" and is stale.

Four built-in sub-agents ship seeded from `electron/main/ai/defaultAgents.json`: Cipher (`configAgent`), Atlas (`knowledgeAgent`), Explorer (`explorerAgent`), Chrono (`taskAgent`). The orchestrator ("Orbit") is singular, not an `agents` row, and is configured from its own prompt file plus settings.

## Tech Stack

| Layer | Technology |
|---|---|
| Language | TypeScript 5, strict, ESM (`"type": "module"`) |
| Shell | Electron 43 |
| Build | electron-vite 5 (main / preload / renderer) + Vite 7 |
| Renderer | React 19 + framer-motion, react-markdown, driver.js |
| Orchestration | `@openai/agents` SDK + `openai` client |
| Persistence | better-sqlite3 (WAL, foreign keys on) |
| Validation | zod 4 |
| Test | vitest 4 + jsdom + @testing-library/react |
| Lint | eslint 9 (flat config) + typescript-eslint |
| Packaging | electron-builder (no config file yet — defaults only) |

## Project Structure

```
/
  index.html                  — renderer HTML entry (Vite root)
  electron.vite.config.ts     — main / preload / renderer builds; injects release defines
  vite.config.ts              — renderer-only web build (npm run dev:web)
  vitest.config.ts            — jsdom, @ alias, release-define stubs, exclusions
  tsconfig.json               — project refs → tsconfig.node.json + tsconfig.web.json
  eslint.config.mjs
  electron/
    main/
      index.ts                — bootstrap: CSP, userData migration, IPC registration, window
      appDirs.ts              — app-owned userData dirs + legacy-root migration
      csp.ts, devLog.ts
      ai/                     — agents.ts, provider.ts, mcp.ts, webSearchDaemon.ts,
                                approvalPolicy.ts, httpTools.ts, documentExtract.ts,
                                skillDistill.ts, userInfoStore.ts, xlsWorker.ts
        prompts/*.md          — orchestrator + built-in sub-agent system prompts
        tools/                — per-agent tool sets (task, knowledge, explorer, folder, …)
        defaultAgents.json    — seed rows for the four built-in agents
      db/                     — index.ts (bootstrap), migrations.ts, one store per table
      ipc/                    — one module per surface, each exporting register*Handlers()
      connectors/             — Google OAuth: account, Gmail, Calendar, Drive, Contacts
      security/               — secretStorage.ts (AES-256-GCM), externalUrl.ts
      net/urlSafety.ts        — SSRF/private-host guards for user-authored HTTP tools
      tasks/scheduler.ts      — recurring + one-shot task runner
      github/skillSearch.ts
    preload/index.ts          — contextBridge surface exposed to the renderer
  src/                        — React renderer
    main.tsx                  — root render (ErrorBoundary → AgentsApp)
    components/               — atoms / molecules / organisms
    hooks/                    — one hook per feature surface
    lib/                      — pure helpers, each with a colocated .test.ts
    vendor/tabler-icons/      — bundled icon webfont
    globals.css
  scripts/releaseInfo.ts      — build-time release metadata (no runtime network call)
  public/                     — audio, video, favicon
  dist-electron/              — build output (untracked)
  wiki/                       — GitHub wiki, separate repo cloned locally (gitignored)
  0-cowork/                   — agent working state (gitignored)
```

## Key Entry Points

- **Main process**: `electron/main/index.ts` — every `register*Handlers()` call is wired here
- **Preload bridge**: `electron/preload/index.ts` — the only channel the renderer may use
- **Renderer**: `index.html` → `src/main.tsx` → `src/components/organisms/AgentsApp.tsx`
- **Schema**: `electron/main/db/migrations.ts` — append-only, ledger-based
- **DB bootstrap**: `electron/main/db/index.ts` — `getDb()`, WAL, FK pragma, legacy renames
- **Agents**: `electron/main/ai/agents.ts` + `ai/defaultAgents.json` + `ai/prompts/*.md`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maneja81/OpenOrbit](https://github.com/maneja81/OpenOrbit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-03 -->
