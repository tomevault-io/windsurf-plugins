---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

NewAgent is a **ground-up TypeScript rewrite** of the **VERDICT** (formerly AMRAAM, originally VERITAS) autonomous web/API pentest agent. It shares only *principles* with the older `CrowSong/` + `HungrySong/` pipeline documented in the parent `/home/sophos/BugBounty/CLAUDE.md` — **no code is reused**. When working under `NewAgent/`, this file governs; the parent file describes a different (Python) codebase.

- **Design source of truth:** `DESIGN.md` (Japanese). `README.md` is the operator guide (commands, pilot modes, WebUI, Burp/header-audit workflows). The richest running log of decisions is in memory (`newagent_claude_pilot_2026_06_09`).
- **Picking up work / cross-machine handoff:** read `docs/NEXT_STEPS.md` first — it carries the current TODOs (attended manual-auth mode, diagnosis parallelization), the agreed design defaults, and new-box setup. (CLI sessions/memory are machine-local; that doc is the portable channel.)
- **Authorized targets only.** Every network action passes the scope gate (`isInScope`); out-of-scope is denied. `--url` derives scope as same-origin + path-prefix; a manifest gives explicit scope.

## Setup & requirements

- **Node.js >= 24** is mandatory — the state store uses the builtin `node:sqlite` (no native dependency). `nvm use 24` if needed.
- **pnpm** via corepack: `corepack enable pnpm` (pinned `pnpm@9.15.4`).
- Playwright uses **`playwright-core`** and needs a chromium binary you supply: `npx playwright install chromium`, or pass `--browser-path <bin>` / set `VERITAS_BROWSER_PATH`. Add `--no-sandbox` in containers.
- LLM calls shell out to the **`claude` CLI** (subscription auth) — see invariants. No `ANTHROPIC_API_KEY` is used.

```bash
pnpm install
pnpm -r build        # tsc per package (+ Vite build for webui)
```

## Commands

```bash
pnpm -r build           # build all packages (respects workspace dep order)
pnpm -r typecheck       # tsc --noEmit across packages
pnpm -r test            # node:test via tsx across packages
pnpm -r clean

pnpm --filter @veritas/core build      # one package
pnpm --filter @veritas/crawler test    # one package's whole suite
```

**Run a single test file** — must be run *from inside the package directory* (the test script globs `src/**/*.test.ts` relative to the package; invoking with a root-relative path fails):

```bash
cd packages/core
node --import tsx --disable-warning=ExperimentalWarning --test src/tree.test.ts
```

**Build before testing dependent packages.** Workspace deps resolve through `main: ./dist/index.js`, so a package that imports `@veritas/core` (crawler, scanner, agent, server, cli, pilot) needs core (and its other deps: llm, scanner) **built** first. Run `pnpm -r build` before a fresh `test`/`typecheck` of dependents. Tests have no external network/LLM needs — they use `FakeDriver` / `FakeHttpClient` / `FakeLlmClient`.

### CLI

Two invocation forms — built JS, or `tsx` dev (resolves `src` directly, no build):

```bash
node packages/cli/dist/main.js <command> [opts]            # after pnpm -r build
pnpm --filter @veritas/cli dev <command> [opts]            # tsx, src-resolved
```

| Command | Purpose |
|---|---|
| `manifest` (alias `init`) | Interactive scope-manifest generator (`node:readline` line-queue, no deps): prompts for target / in·out-of-scope hosts+paths / rate / crawl / model / auth roles → writes the `AssessManifest` JSON that `pilot`/`assess` read. `--out <file>` / `--force`; passwords are read with echo masked. Default name `scope_manifest_<host>.json` is gitignored. |
| `pilot` | **Claude-led, staged** assessment (survey → methodology → diagnosis). `--manifest`/`--url`, `--model` (deep) + `--fast-model` (model tiering), `--max-turns`, `--headed`, `--burp-proxy <url>` (route all traffic through Burp; off by default = unchanged), `--keepalive-min <n>` (touch the top page between screens to keep an authed session warm; default 4, `0` = off). |
| `pilot --survey-only` | Map only (screens + screenshots + APIs); no methodology/diagnosis/findings. Cheap recon → resume later. |
| `pilot --resume --id <id>` | Continue an existing run: skip survey/methodology, diagnose only non-terminal (queued) screens. Reuses the run's browser-profile for auth. |
| `assess` | **Deterministic** one-shot: crawl → label → scan → logic → report (`--login-url`/`--login-wait` for headed login). |
| `serve` | Observability WebUI + state API/WS (default `127.0.0.1:4317`; `--host 0.0.0.0` to expose). |
| `shots --id <id>` | Backfill per-screen screenshots into an existing run (reuses its browser-profile; navigate-only). |
| `header-audit --id <id>` | Deterministic Info-level header checks (`--headers csp,hsts,…`). Toggle = run it or not. |
| `burp-import --id <id> --report <xml>` | Parse a Burp Pro XML report, merge **net-new** issues (deduped vs agent findings). |
| `run` / `crawl` / `label` / `scan` / `logic` | Deterministic pipeline steps (the internals of `assess`). |
| `report` / `status` / `list` | report.md; phase/coverage/stop-condition; list `runs/`. |

## Architecture (big picture)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vvts-alpha/VERDICT](https://github.com/vvts-alpha/VERDICT) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-01 -->
