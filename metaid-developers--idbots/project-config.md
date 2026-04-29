---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Start Here

Read in this order:

1. This `AGENTS.md`
2. `localdocs/README.md` if it exists locally
3. Relevant docs under `docs/superpowers/` when the task touches the current P2P / release baseline

Keep this file stable. Put fast-changing local context, current baselines, recent pitfalls, and reusable new-session prompts in `localdocs/`.

## Project Overview

**IDBots** is a local-first desktop MetaID / MetaBot platform built with Electron, React, and TypeScript.

The app has two major halves:
- **Main process**: orchestration, local services, wallet / MetaBot persistence, subprocess management, packaging/runtime path logic
- **Renderer**: onboarding, MetaBot management, cowork flows, skills, messaging, scheduled tasks, P2P status/settings UI

As of March 23, 2026, `main` includes the local-first `man-p2p` Alpha baseline used for packaged macOS and Windows releases. The expected healthy peerless UI state is `0 peers` with `p2p-only`, not `Connecting...`.

## Repository Layout

| Path | Role |
| --- | --- |
| `src/main/` | Electron main process entrypoints, IPC, services, SQLite-backed app logic |
| `src/main/services/` | Runtime services such as P2P, MetaID RPC, restore, proxying, orchestration |
| `src/main/im/` | Gateway integrations (Telegram, Discord, Feishu, DingTalk, NIM, etc.) |
| `src/renderer/` | React renderer UI and client-side services |
| `src/renderer/components/` | Main UI surfaces: onboarding, MetaBots, cowork, skills, IM, P2P, updates |
| `resources/man-p2p/` | Bundled `man-p2p` runtime binaries + manifest/config for packaging |
| `scripts/` | Build, packaging, sync, runtime bootstrap, CI helper scripts |
| `tests/` | Node-based tests for main-process services, packaging helpers, and P2P runtime contracts |
| `docs/superpowers/` | Specs, plans, acceptance notes from the recent P2P / alpha integration work |

## Build & Run Commands

```bash
# Install dependencies
npm install

# Main local dev loop
npm run electron:dev

# Compile Electron TypeScript only
npm run compile:electron

# Build renderer + main bundles
npm run build

# Refresh bundled man-p2p binaries from the sibling man-p2p repo/build output
npm run sync:man-p2p

# Package release artifacts
npm run dist:mac
npm run dist:win

# Run the node-based test suite
node --test tests/*.test.mjs
```



## Current Development Workflow

- If the task changes `man-p2p` behavior, make and verify the runtime change in the `man-p2p` repo first.
- After rebuilding the relevant binary, run `npm run sync:man-p2p` here to refresh `resources/man-p2p/`.
- Use `npm run electron:dev` for fast integration iteration.
- Fresh git worktrees do not inherit ignored `node_modules` directories. In this repo that especially affects `SKILLs/web-search/node_modules`.
- If `npm run electron:dev` or `npm run build:skills` fails in a fresh worktree with `TS2307` for `express` / `playwright-core` under `SKILLs/web-search`, treat it as a missing skill-runtime install in the current worktree first, not as a source-level TypeScript regression.
- `npm run build:skill:web-search` and `npm run build:skills` are expected to bootstrap missing `SKILLs/web-search` dependencies for the current worktree before compiling; if that bootstrap is ever removed or broken, fix it rather than repeatedly hand-installing in each new worktree.
- Fresh nested worktrees can also hit `sql.js` startup failures even when the app compiles: `sql-wasm.wasm` may be missing under the worktree-local path while Node actually resolved `sql.js` from a parent repo `node_modules`.
- If `electron:dev` logs `ENOENT ... node_modules/sql.js/dist/sql-wasm.wasm` under a worktree path, the root cause is usually incorrect wasm path resolution from `app.getAppPath()`, not a broken `sql.js` install in the shared parent repo.
- Keep `src/main/sqliteStore.ts` resolving the wasm file from the nearest real ancestor `node_modules/sql.js/dist/sql-wasm.wasm` in development, so fresh worktrees reuse the parent repo dependency tree without needing a duplicated root `node_modules`.
- Use packaged app builds for alpha acceptance and release validation. Do not treat dev runtime behavior as sufficient release evidence.
- `electron:dev` assumes Vite owns port `5175`. If another repo already has that port open, Electron may load the wrong frontend.
- Before any commit intended to be kept or pushed, run `npm run lint` and do not submit changes while lint is failing.

## Commit and Merge Rules

- If you notice unfamiliar or unrelated file changes, continue working and stay focused on your own scoped edits unless the user asks you to inspect them.
- Before creating a new git worktree or branch, ask for explicit user confirmation first.
- When the user says "commit", stage and commit only the files you changed and understand.
- Prefer small, frequent commits. Commit each independent, verifiable unit of work as soon as it is complete.
- For every modification or newly added feature, create one commit.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [metaid-developers/IDBots](https://github.com/metaid-developers/IDBots) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
