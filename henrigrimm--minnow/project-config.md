---
trigger: always_on
description: Guidance for AI coding agents (Cursor, Claude Code, etc.) working in the Minnow repo.
---

# AGENTS.md

Guidance for AI coding agents (Cursor, Claude Code, etc.) working in the Minnow repo.

## Overview

Minnow is a **Vite + TypeScript SPA** plus a **Node tool server** (`server.js`) and an **Electron desktop shell** (Minnow Shell). It is a local-first AI workspace for LM Studio and other OpenAI-compatible providers.

- **Four composer modes** — General, Build, Plan (no-destructive guard), Debug — plus **Orchestrate** (opens from the sidebar hub, not the composer picker), **Super Plan**, **Desktop**, **Email**, and **Onboarding** as non-composer modes. Defined in [`src/chat/modes/registry.ts`](src/chat/modes/registry.ts); prompts in [`src/chat/prompts/modes/`](src/chat/prompts/modes/). Reef mode was removed (MIN-473) — do not reintroduce it.
- **114 built-in tools** across web / utility / files / git / code / agents / browser / lsp ([`src/tools/definitions.ts`](src/tools/definitions.ts)). Includes `issue_*` tools and the read-only `minnow_docs_*` tools (search the shipped user manual under `documentation/manual/` only). Entries with an `appId` (8 calendar/email tools) are filtered out while that app is release-gated or user-disabled (MIN-472), so the shipped catalog is 106.
- **Built-in slash skills** (17): core helpers (`git-commit`, `code-review`, `fix-ci`, `ask-user`, …), `impeccable` (default-on), `caveman`, `ui-designer`, `partymode`. Third-party packs — including the **Matt Pocock pack** (19 skills) — install from **Settings → Skills Library**; nothing else is bundled. See [`documentation/context.md`](documentation/context.md) § Skills.
- **Minnow apps — released:** Chat (desktop), Code, Research, Models, Brain, **Issues**, Scheduler, Settings — all `core` (not user-disableable). **Hidden** (`releaseState: 'hidden'`, MIN-471): Compare, Bench, Experts, Calendar, Email — code stays in tree but is omitted from dock, onboarding, Settings, routes, notifications, and `launch_minnow_app`. Registry: [`src/os/app-registry.ts`](src/os/app-registry.ts).
- **Persistence** lives under `~/.minnow` when the tool server runs.

The **authoritative reference** is [`documentation/context.md`](documentation/context.md) — read it before touching unfamiliar subsystems. Product overview: [`README.md`](README.md). Setup and scripts: [`documentation/contributor/setup-from-source.md`](documentation/contributor/setup-from-source.md) and [`documentation/contributor/commands.md`](documentation/contributor/commands.md). Full doc index: [`documentation/`](documentation/README.md).

## Running the app

- **`npm start`** is the recommended dev command — Vite + the Node tool server on port **9473** (or next free port if `PORT` is set) and **launches the Electron desktop shell by default**. `MINNOW_BROWSER=1` opens the system browser instead; `BROWSER=none` or `MINNOW_HEADLESS=1` suppresses auto-open. `npm run desktop` / `npm run electron:dev` are HMR-friendly Electron aliases.
- **`npm run dev`** is Vite-only (no tool server) — fine for pure UI work, but most tool-dependent features won't function.
- **Headless CLI:** `minnow run --prompt "…"` (or `npm run minnow:run -- --prompt "…"`) drives the same generations + server tools without the SPA. Requires `npm start` (or `--start-server`). See `minnow run --help`.
- Health checks: `curl http://localhost:9473/api/tools/ping`, `/api/config/ping`, `/api/memory/ping`, `/api/brain/ping` (substitute your `PORT` if overridden).
- **LM Studio headless daemon** (`llmster`): install with `curl -fsSL https://lmstudio.ai/install.sh | bash`; `lms daemon up && lms server start`; `lms get <model> -y`; `lms load <model> -y`. CLI at `~/.lmstudio/bin/lms`.

## Testing

- **`npm test`** runs the full suite via [`test/run-all.mjs`](test/run-all.mjs) — auto-discovers `test/**/*.test.{js,mjs,mts,ts}` and batches by runner (`node --test`, `tsx` + [`test/test-loader.mjs`](test/test-loader.mjs)). New test files under `test/` are picked up with zero `package.json` edits.
- **`npm run test:check-coverage`** fails if any discoverable test file is not covered (CI gate).
- **CI:** [`.github/workflows/ci.yml`](.github/workflows/ci.yml) on every PR + push to `main` — `npm ci`, `test:check-coverage`, `npx tsc --noEmit`, `npm test` (Windows + Ubuntu). Enable branch protection per [`.github/BRANCH_PROTECTION.md`](.github/BRANCH_PROTECTION.md).
- **`npx tsc --noEmit`** for type checking (no separate ESLint config).
- Scoped suites: `npm run test:memory|brain|engine|lsp|mcp|browser|skills|attachments|research|benchmark|evals|calendar|email|webhooks|notifications|voice|servers|plugins|terminal-pty|ui-designer|scheduler|onboarding|issues|board`. See `package.json` for exact globs. Board testing: [documentation/contributor/orchestrate-board-testing.md](documentation/contributor/orchestrate-board-testing.md).
- Many TS/UI suites run under `tsx` with `--import ./test/test-loader.mjs` (the loader stubs `.css` and xterm). Some use `--experimental-test-module-mocks`.

## Building & packaging

- **`npm run build`** → `tsc && vite build` → `dist/`. The `prebuild` step generates `src/skills/builtin-manifest.json`.
- **`npm run package`** → build + `electron:build` + `electron-builder` (Windows NSIS → `release/`). `package:dir` produces an unpacked directory.

## Performance budgets (MIN-400)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HenriGrimm/Minnow](https://github.com/HenriGrimm/Minnow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-05 -->
