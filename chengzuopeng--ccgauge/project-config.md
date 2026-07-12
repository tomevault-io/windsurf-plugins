---
trigger: always_on
description: Working agreement for AI coding agents (Claude Code, Codex CLI, Cursor agents,
---

# AGENTS.md

Working agreement for AI coding agents (Claude Code, Codex CLI, Cursor agents,
etc.) editing this repo. Humans should skim it too — it captures the non-obvious
parts of the codebase that aren't in the README.

## What this project is

`ccgauge` is a **local web dashboard + CLI + MCP server** for inspecting
**Claude Code** and **OpenAI Codex CLI** token usage and cost. Everything runs
on the user's machine; there are zero outbound network calls at runtime.

Three user-facing surfaces share one data layer:

| Surface | Entry | Talks to |
| --- | --- | --- |
| Web dashboard | `app/` (Next.js 15 RSC) | the indexer singleton |
| CLI (`ccgauge`) | `bin/cli.mjs` | the indexer via a bundled `dist/report/index.mjs` |
| MCP server (`ccgauge mcp`) | `lib/mcp/entry.ts` → `dist/mcp/server.mjs` | the indexer via a separate `'mcp'`-named instance |

Published to npm as `ccgauge`. End-users typically run `npx ccgauge`.

## Repo layout

```
app/                       Next.js routes (RSC pages + /api routes)
  api/                     Server-side JSON endpoints (scan, usage, sessions, …)
  page.tsx                 Overview
  usage/, sessions/, …     Drill-down pages

bin/cli.mjs                Single-file CLI (commander). Imports the bundled
                           dist/report/index.mjs and dist/mcp/server.mjs lazily
                           when those subcommands run.

components/                React. All client UI atoms (KpiCard, Section,
                           usage-table, hover-card, activity-stats, …).

lib/
  providers/               Per-CLI adapters (claude, codex). Add a new provider
                           by dropping a folder + one line in lib/providers/index.ts.
  data-loader/             scan.ts (entry), indexer.ts (singleton + watchers +
                           persist), parse-jsonl.ts (claude parser).
  aggregator/              Pure aggregations: totals, time-buckets, by-model,
                           by-project, by-session, activity heatmap.
  pricing/                 cost-from-usage.ts (math) + per-provider rate tables.
  serialize.ts             Shape AssistantRecord[] → UsageTableRow[] /
                           UsageTurnRow[]; turn grouping lives here too.
  turns.ts                 Parent-chain walking that decides which assistant
                           records collapse into the same "turn".
  cli-report/              The pretty terminal report. Bundled into dist/report.
  mcp/                     MCP server (stdio JSON-RPC). Bundled into dist/mcp.
  i18n/, theme/            Cookie-driven SSR + localStorage mirror + no-flash.

scripts/                   Build & test helpers (esbuild bundlers, postbuild,
                           parser fixtures).

dist/                      esbuild output. Not source-controlled — generated
                           by `pnpm build`. Listed in package.json#files so it
                           ships in the npm tarball.

site/                      Astro 4 marketing site (ccgauge.dev). Source-only
                           subdirectory: commands and dependencies are managed
                           by the root `package.json` / `pnpm-lock.yaml`, while
                           site builds remain explicit via `pnpm site:*`.
                           NOT published to npm — excluded by the main
                           `package.json#files` allowlist and a
                           belt-and-suspenders `site/` line in `.npmignore`.
                           Develop with `pnpm site:dev` (runs on :4321 so it
                           doesn't clash with the dashboard on :3738).
```

## Commands you'll actually run

```bash
pnpm dev            # Next dev on :3738, hot-reload
pnpm typecheck      # tsc --noEmit — run before any commit
pnpm lint           # eslint flat config — run before any commit
pnpm test           # codex parser smoke test (node --experimental-strip-types)
pnpm build:report   # rebuild just dist/report (lib/cli-report/index.ts → bundle)
pnpm build:mcp      # rebuild just dist/mcp
pnpm build          # full: next build + mcp + report + postbuild
pnpm site:dev       # Astro marketing site on :4321
pnpm site:build     # build only site/ into site/dist

# After source changes, exercise the CLI without reinstalling:
node bin/cli.mjs report --no-color -r 7d
node bin/cli.mjs report --json | jq .totals
```

`pnpm dev` starts on port **3738** (not 3737). Standalone `npx ccgauge` starts
on 3737 by default.

## Architecture invariants (the non-obvious stuff)

1. **The indexer is a module-level singleton with file watchers.**
   `lib/data-loader/indexer.ts` exports `export const indexer = getIndexer()`
   which is a side effect at import time. Anything that imports this module —
   even just to call `getCachedScan()` — spins up a long-lived indexer that
   watches `~/.claude/projects` / `~/.codex/sessions` and writes
   `~/.ccgauge/cache/index-v2.json`. Implications:
   - **In `pnpm dev`, the singleton survives HMR.** If you change parser code,
     the module is reloaded but the existing indexer's in-memory `files` map
     still holds records parsed by the **old** code. Bump the provider's
     `parserVersion` in `lib/providers/<name>/index.ts` so the indexer detects

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chengzuopeng/ccgauge](https://github.com/chengzuopeng/ccgauge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
