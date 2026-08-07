---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## ⚠️ This is a public repository

**Argus is public open source.** Never commit or push private, sensitive, or personal data —
not in code, tests, fixtures, comments, docs, or commit messages. This includes:

- **Secrets**: API keys, tokens, certificates, credentials, `.env` values.
- **Data stores**: `argus.db`, `argus.json`, the contents of `$ARGUS_CONFIG_DIR`/`$ARGUS_CACHE_DIR`,
  or any indexed/cached output produced by running Argus locally.
- **Agent session data**: real Claude/Codex/Gemini transcripts, prompts, or any captured
  conversation content from `~/.claude`, `~/.codex`, etc.
- **Local file information**: real home/user paths, machine names, directory listings, or anything
  that identifies a specific user or machine.
- **PII**: names, emails, org/customer identifiers, or any personal information.

When you need example data, synthesize it. Use redacted, obviously-fake fixtures (e.g. `/Users/you`,
`user@example.com`). When in doubt, leave it out and ask.

## What this is

**Who it's for.** Argus is built for **business users, not developers** — people who use agents to
do their *business* work, not to build software. The audience spans a range, from knowledge workers
who never open a terminal to technical non-developers who run Claude Code and write the occasional
script. When you design features, write copy, or choose examples/taxonomies, assume that range and
never assume a developer.

**`docs/contributing/audience.md` is the canonical audience definition** — read it before writing
anything user-facing or designing anything, and don't restate it elsewhere. Never call them
"non-coders" or talk down.

**How we describe Argus is canonical in `docs/contributing/positioning.md`** — the promise, the
approved descriptions, which register each surface uses, which agents the descriptions name, and
the inventory of every place a description lives. Anything outward-facing (the README, the repo
description, package metadata, social cards, release notes) comes from there.

Argus audits local agent usage — Claude Code, Claude Cowork, Claude Chat, Codex, and Gemini — by
reading local session transcripts (`~/.claude/projects/**/*.jsonl`, `~/.codex/sessions/**/*.jsonl`,
…). All parsing is local. **Most users run the desktop app**, and it's the preferred way to interact
with Argus: a Tauri **tray app** (`desktop/`) that wraps the CLI — it bundles the compiled `argus`
binary plus the web assets, supervises `argus run` in the background, opens the dashboard in the
user's default browser through a stable local front-door port (default `4242`, proxied so an open
tab survives background restarts), and auto-updates. The tray app is the front door; the CLI is the
engine underneath.

That engine is a Bun + TypeScript CLI, and the more technical end of the audience can run it
directly. `serve` runs the local **web app** — a React SPA (see `docs/internals/web-app.md`) that is
the dashboard the desktop app opens. `index` reads transcripts into the local store (`argus.db`).
`sync` (formerly `push`) uploads per-(org, user) usage data to an **Argus Hub** — a hosted backend a
company runs to pool its users' usage.
`run` ties the long-running pieces together (`index --watch` + `serve`, plus `sync --watch` when a
Hub is configured) in one supervised process — this is what the desktop app runs. Nothing is
uploaded during `serve`/`index`; the only data that ever leaves the machine is what `sync` sends.

This repo is the public CLI, its web app (`web/`), and the desktop tray shell (`desktop/`). The
Hub's own code (the backend `sync` uploads to) lives in a **separate public repo**,
`agentdeploymentco/argus-hub`.

## Commands

```bash
bun run src/cli.ts serve --open          # run the interactive web app (needs build:web once)
bun run dev:web                           # Vite dev server for web/ (live reload; proxies /api → 4242)
bun test                                  # run all tests (uses bun:test, zero extra deps)
bun test test/parse.test.ts               # run a single test file
bun test -t "dedup"                       # run tests matching a name
bun run typecheck                         # tsc --noEmit for root src + web/ (also run in CI)
bun run build:web                         # build web/ → dist/web
bun run build:compile                     # compile a self-contained CLI binary → dist/argus (bun:sqlite, no Node)
bun run build:npm                         # build the publishable npm package set → dist/npm/* (all OS/arch)
bun run desktop:build                     # build the Tauri tray app (per-OS bundle) → desktop/src-tauri/target/**
```

CI (`.github/workflows/ci.yml`) typechecks the root and `web/`, runs `bun test`, and verifies
`build:web` on every PR and on pushes to `main`. Development runs `src/cli.ts`
directly with Bun. There is no Node-targeted bundle: the CLI compiles to a self-contained binary
with `bun build --compile` (it uses `bun:sqlite`, so it needs no Node/node-gyp). `bun run build:npm`
emits per-OS packages under `dist/npm/` — a launcher package (`@agentdeploymentco/argus`, the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Agent-Deployment-Co/argus](https://github.com/Agent-Deployment-Co/argus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
