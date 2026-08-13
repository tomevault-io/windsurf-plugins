---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-container appliance for exercising the Ignite UI AI toolchain (the Ignite UI CLI MCP server, the Theming MCP server, and the Agent Skills) against **opencode**. A small Express web wizard collects the user's choices, scaffolds an Ignite UI project, wires the AI config, and hands off to the opencode web UI. Each session runs in a fresh, ephemeral rootless Podman container.

This repo's logic (the MCP translation and server classification) is unit-/syntax-tested, but the README notes it has **not** been run end-to-end against real `igniteui-cli` / `opencode` / Podman — treat first builds as a shakedown, and expect the "adjust to your packages" spots (see below) to need tuning.

## Commands

```bash
./run.sh build           # podman build -t localhost/igniteui-testbed:latest .
./run.sh build --prune   # build, then `podman image prune -f` to drop dangling <none> images
./run.sh                 # run a fresh ephemeral container; publishes ports 8080 / 4096 / 5000
./run.sh --matrix-config <file>  # run with a matrix JSON config: bind-mounts it to
                         # /matrix-config.json, sets MATRIX_CONFIG, and (by default)
                         # auto-runs the matrix headlessly; the UI prefills from it
./stop.sh [<sess>]       # stop running testbed container(s) (all, or one by session id)
# all four scripts take -h/--help/help (PowerShell: -Help, `help`, -? or Get-Help) and print usage

npm start                # run the wizard backend directly (node src/server.ts), for host-side dev
npm run typecheck        # tsc --noEmit gate over backend (tsconfig.json) + frontend (web/tsconfig.json)
npm run build:web        # esbuild web/main.ts -> public/vendor/app.js (run alongside `npm start` for host dev)
npm run dev:web          # build:web in --watch mode
```

On Windows, `run.ps1` / `stop.ps1` are PowerShell ports of `run.sh` / `stop.sh` with the
same arguments (`.\run.ps1 build`, `.\run.ps1 build -Prune`, `.\run.ps1`,
`.\run.ps1 -MatrixConfig <file> [-Validate]`, `.\stop.ps1 [<sess>]`). The prune step (`--prune` /
`-Prune`) only runs after a successful build and removes only untagged images. Keep the
two implementations in sync — a change to one platform's run/stop logic should be
mirrored in the other.

There is no test runner or linter. The wizard backend is **ESM TypeScript** under `src/` (entry `src/server.ts` → `src/app.ts` wiring → `src/routes/*`), run natively by Node (≥24) which strips types at load — **no build/emit step for the backend**; `tsc --noEmit` (the `typecheck` script) is the separate type gate. The frontend is TypeScript under `web/`, the one piece that *is* compiled (esbuild → `public/vendor/app.js`, since browsers can't run `.ts`). Runtime deps: `express` and `playwright` (the latter only for matrix screenshots); the TypeScript toolchain (`typescript`, `@types/*`, `esbuild`) is dev-only and never ships in the runtime image.

Ports: wizard `8080`, opencode web `4096`, generated app dev server `5000`. These are fixed at container-create time (Podman can't add published ports later), so the app dev server is forced onto `0.0.0.0:5000`.

## Architecture

The backend lives under `src/`: `src/server.ts` is the entry (ensureDirs → provider-pack loadAll → the `MATRIX_VALIDATE` early-exit → history.reapStale → matrix-config load → console-mirror attach → listen), `src/app.ts` is the thin express wiring (static mounts + `register*Routes(app)` imported from each `./routes/*` module), and the HTTP handlers are split across `src/routes/{run,matrix,history,stats,skills,providers,tests,prompt-images}.ts`. The shared pipeline engine is `src/pipeline/pipeline.ts`, the per-run capture modules live under `src/capture/` (`usage.ts` tokens/cost, `tool-usage.ts` MCP-tool/skill invocations, `route-discovery.ts` + `screenshots.ts`), the matrix engine `src/matrix/matrix.ts`, and cross-cutting session state (`lastConfig`, run progress, the live `StatsCollector`) lives in `src/session.ts`. The three SSE feeds (run progress, stats, matrix) are one `createSSE()` registry helper in `src/stream/sse.ts`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IgniteUI/igniteui-mcp-testbed](https://github.com/IgniteUI/igniteui-mcp-testbed) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
