---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Orientation

`better-review` is a **local code-review tool**: a Node daemon + React SPA that drives one of several review-agent CLIs (currently `codex`, `claude`, and `pi`) plus the `gh` CLI. It reviews GitHub PRs, local git branches, and GitButler virtual branches — the per-source logic is behind the `SourceFlow` seam in `src/server/source/`. The agent layer is pluggable too — see `src/server/engine/agent/`. Distributed as an npm bin (`better-review`) that launches a detached daemon, opens a browser UI, and shells out to those CLIs for actual work. There is no cloud component, no auth layer, and no multi-user state — everything lives under `~/.better-review/`.

User-facing docs live in `README.md` (English) and `README.zh-CN.md` (Chinese). Read them for end-to-end semantics (PR input formats, submit flow, prompt overrides, config keys, rerun/round semantics). The notes below are about _building and changing_ the code, not using it.

## Common commands

```bash
pnpm run build          # tsc(server) + vite(web) + scripts/copy-assets.mjs
pnpm run dev:server     # tsx watch src/server/index.ts (daemon)
pnpm run dev:web        # Vite dev server on :5174, proxies /api → 127.0.0.1:7345
pnpm run test           # vitest: server + cli + shared (Node env, single fork)
pnpm run test:web       # vitest jsdom: tests/web + src/web component tests
pnpm run e2e            # Playwright happy path (run `pnpm exec playwright install chromium` once)
pnpm run lint           # oxlint
pnpm run format         # oxfmt (writes); `pnpm run format:check` to verify in CI
```

Single-test patterns:

```bash
pnpm exec vitest run tests/server/engine/findings-parser.test.ts
pnpm exec vitest run -t "fragment of test name"
pnpm exec vitest run --config vitest.web.config.ts tests/web/FindingRow.test.tsx
pnpm exec playwright test tests/e2e/<file>.spec.ts
```

The Node-env vitest run is pinned to `pool: "forks"` with `singleFork: true` (see `vitest.config.ts`) because tests share a `better-sqlite3` temp file pattern. Don't parallelize without checking that.

## Build pipeline gotchas

- Three tsconfigs: `tsconfig.server.json` (CLI + server + shared → `dist/`), `tsconfig.web.json` (Vite, `@/` and `@shared/` aliases), `tsconfig.test.json`. Keep server-only and web-only code separated; only `src/shared/` is consumed by both.
- `scripts/copy-assets.mjs` runs after `tsc` and does four things:
  1. Copies `src/server/db/migrations/*.sql` into `dist/server/db/migrations/`.
  2. Copies the language-paired prompt assets (`prompts/framework.{en,zh-CN}.md` + `prompts/builtin-rules.{en,zh-CN}.md`) to `dist/prompts/`.
  3. **Rewrites relative imports in compiled `.js` to add `.js` extensions** (and folds `./foo` → `./foo/index.js` where applicable). This is required because the package is ESM (`"type": "module"`) but the source uses extensionless imports. Don't manually add `.js` to `.ts` imports — rely on this step.
  4. Chmods `dist/cli/index.js` to 0755 so the bin works after `npm install -g .`.
- Vite builds the SPA to `dist/web/`. The daemon serves it via Hono static middleware (`webDir = dist/web`) — see `src/server/index.ts:67`.

## Architecture

### Process model

```
better-review (CLI)
  └─ checks ~/.better-review/server.json + /api/health
     └─ if dead: spawns detached `node dist/server/index.js`
        └─ daemon: Hono on 127.0.0.1:<port>
                   ├─ /api/* routes
                   ├─ /api/events SSE (per-session + global)
                   └─ static SPA from dist/web
        └─ on POST /api/sessions: orchestrates one review
```

The CLI is a thin launcher (`src/cli/index.ts`, `src/cli/daemon-launcher.ts`). All real logic is in `src/server/`. SPA talks to daemon via `/api/*` (REST + SSE). There is no websocket.

### Review session lifecycle

Source: `src/server/start-session.ts` plus `src/server/source/`, `src/server/engine/`, and `src/server/git/`. Sessions transition through `pending` (prep in progress) → `running` (agent producing findings) → `ready` (awaiting human submit) → `submitted` / `failed` / `cancelled` / `archived` (set by rerun).

1. **Resolve the session source** — `source/parse.ts` turns the user's input string into a `SessionSource` (zod union in `src/shared/source.ts`): canonical HTTPS GitHub PR URLs (`https://github.com/<owner>/<repo>/pull/<n>`, via `github/pr-target-parser.ts`) become `github-pr`; absolute / `~`-prefixed paths become `local-branch` (or `gitbutler-vbranch` when a vbranch name is supplied); anything else throws. `source/registry.ts` then picks the matching `SourceFlow` implementation (`github-pr-flow` / `local-branch-flow` / `gitbutler-vbranch-flow`) — the seam that the rest of prep drives. Sources are persisted as canonical JSON (`source_json`) keyed by `source/hash.ts` for dedup.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xieziyu/better-review](https://github.com/xieziyu/better-review) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
