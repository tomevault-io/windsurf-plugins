---
trigger: always_on
description: This guide is for humans and AI coding agents contributing to Brigade. It covers
---

# AGENTS.md — Working in the Brigade codebase

This guide is for humans and AI coding agents contributing to Brigade. It covers
the layout, the commands, the conventions you must follow, and the recipes for
common changes. Read it before you touch the tree.

For user-facing docs see [README.md](README.md). For the release flow see
[docs/RELEASING.md](docs/RELEASING.md).

---

## What Brigade is

A single-operator, multi-agent AI-crew runtime written in TypeScript. It runs as a
headless WebSocket **gateway** (the state-holding daemon) with thin clients (a chat
TUI, `connect`, channel adapters). It is built on the **pi** SDK
(`@mariozechner/pi-coding-agent` + `@mariozechner/pi-tui`, pinned exact at
`0.73.1`) and adds product layers on top: memory, skills, sub-agents, tools, an org
hierarchy, channels, cron, an extension SDK, and a dual-mode storage layer.

- **Stack:** TypeScript (strict), ESM, Node **≥ 22.12**, **npm** (never pnpm).
- **Build:** `tsc` → `dist/`. No bundler.
- **Tests:** Node's built-in `node:test`, run via `tsx`.
- **State:** everything under `~/.brigade/` (override: `BRIGADE_STATE_DIR`).

---

## Naming & brand conventions (MUST follow)

- The product is **Brigade**. In code, comments, copy, CLI help, error messages,
  env vars, daemon labels, and docs, refer to the agent SDK **only** as `pi` /
  `@mariozechner/pi-coding-agent`. Do not name other third-party AI agent projects.
- **Never** commit secrets, real personal data (names, phone numbers, emails), or
  local absolute paths. Use placeholders (`+1 555 010 0001`, `/path/to/project`).
- The mascot is 🦁 the Pride. No other animal/symbol.
- `templates/workspace/` holds default persona files and is **off-limits**:
  behavioural fixes go in code, never in template content.
- No `Co-Authored-By` trailers on commits.

Daemon labels (don't rename): macOS launchd `com.brigade.gateway`, Linux systemd
`brigade-gateway`, Windows Task Scheduler `BrigadeGateway`.

---

## Commands

| Task | Command |
|---|---|
| Build | `npm run build` (`tsc -p tsconfig.build.json` → `dist/`) |
| Build (watch) | `npm run build:watch` |
| Typecheck | `npm run typecheck` (`tsc -p tsconfig.json --noEmit`, includes tests) |
| Test (all) | `npm test` (`node scripts/run-tests.mjs`) |
| Test (one file) | `npm test -- src/agents/tools/registry.test.ts` |
| Memory eval/bench | `npm run bench` |
| Run built binary | `npm run brigade -- <args>` (e.g. `npm run brigade -- agent -m "hi"`) |
| Dev (build-then-run) | `npm run dev -- <args>` |
| Dev (no build, tsx) | `npm run dev:tsx -- <args>` |
| Dev (auto-restart) | `npm run watch` |
| Clean | `npm run clean` |

Before opening a PR, all three of these must pass: `npm run typecheck`,
`npm test`, `npm run build`.

**Test isolation:** `scripts/run-tests.mjs` pins `BRIGADE_STATE_DIR` to a fresh
tempdir so tests never touch your real `~/.brigade`. There are **281** `.test.ts`
files under `src/`.

---

## Repository layout

```
F:\Brigade\  (working tree; storage is ~/.brigade)
├── brigade.mjs                 # bin shim: enforces Node ≥22.12, routes to dist/entry.js
├── package.json                # npm; pi 0.73.1 pinned exact
├── tsconfig.json               # typecheck config (includes tests)
├── tsconfig.build.json         # build config (emits dist/, excludes tests/templates)
├── scripts/                    # run-brigade.mjs, run-tests.mjs, build-done.mjs, convex-*.mjs
├── skills/                     # 56 bundled skill directories
├── templates/workspace/        # default persona files — OFF LIMITS for edits
├── convex/                     # Convex schema + functions (optional storage backend)
└── src/
    ├── entry.ts                # CLI entry (fast-path version/help, lazy dispatch)
    ├── extension-sdk.ts        # public plugin SDK (defineModule + re-exports)
    ├── cli/                    # command files + program/build-program.ts (command registry)
    ├── core/                   # gateway server.ts, daemon/ installers, dispatch
    ├── agents/                 # the runtime (see below) — the bulk of the code
    │   ├── agent-loop.ts       # per-turn loop
    │   ├── session-wiring.ts   # toolset assembly + before-tool-call guards
    │   ├── tools/              # Brigade-native tools + registry.ts
    │   ├── memory/             # facts store, decay, auto-recall, consolidate
    │   ├── skills/             # 6-source discovery, eligibility, manage
    │   ├── subagent-*.ts       # spawn, policy, abort cascade, completion bridge
    │   ├── channels/           # adapter contract + inbound-pipeline + whatsapp/
    │   ├── routing/            # inbound → (agentId, sessionKey)
    │   ├── org/                # org/Pride hierarchy + A2A policy
    │   └── extensions/         # plugin engine + bundled modules/ (web search, etc.)
    ├── tideline/               # long-term memory engine (hybrid recall, link graph)
    ├── system-prompt/          # assembler.ts + sections (persona pin, org anchor)
    ├── sessions/               # session store, write-lock, transcript repair
    ├── storage/                # dual-mode store: local/ (filesystem) + convex/
    ├── config/                 # brigade.json schema, io, validators
    ├── providers/              # model provider catalog + auth detection

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spinabot/brigade](https://github.com/spinabot/brigade) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-21 -->
