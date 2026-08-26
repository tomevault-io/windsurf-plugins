---
trigger: always_on
description: Orientation for agents working in this repo. Read before editing. README.md is the user-facing doc; this file is the engineering map: commands, layout, wire contract, invariants, conventions, verification.
---

# AGENTS.md

Orientation for agents working in this repo. Read before editing. README.md is the user-facing doc; this file is the engineering map: commands, layout, wire contract, invariants, conventions, verification.

## What this repo is

Two products in one tree, sharing one Solid.js web UI and one wire contract:

- **omp-session** (`server/`): single-session agent daemon. One process, one project dir (bound at spawn, immutable), one live agent session via the `@oh-my-pi/pi-coding-agent` SDK **in-process** (`createAgentSession`: no child process, no JSON-RPC hop). Serves the full standalone UI over SSE + POST.
- **omp-fleet** (`fleet/`): registry + supervisor + connector for N daemons (local children, external/remote). Re-exposes them to the same UI in **roster mode** and to CLI fan-out. Holds **zero agent state**. All truth lives in the omp-session processes and their `.jsonl` session logs.
- **Web UI** (`src/`): one Solid.js bundle serves both modes; mode is decided by the wire (`roster` frame ⇒ roster mode, sticky; a bare omp-session never sends it ⇒ standalone). No router.

Runtime is **Bun** (`type: module`, `bun.lock` committed). No CI. Lint runs through **oxlint** (`.oxlintrc.json`; Solid rules come from `eslint-plugin-solid` loaded as an oxlint JS plugin) and formatting through **oxfmt** (`.oxfmtrc.json`: tabs, print width 100, TS/TSX only: markdown/CSS/HTML/JSON stay hand-maintained). Comments reference audit findings as `finding #N` (numbering kept from the 2026-08 audit).

## Commands

```sh
bun install                    # once
bun run dev                    # roster mode: vite (HMR, /events+/command proxied to fleet edge) + omp-fleet, ports chosen per-run; fleet state/lock scoped per worktree under the data home (dev-fleets/<slug>-<hash8>/), so N worktrees + the user's real fleet coexist; config and managed workspaces stay shared
bun run dev:single             # standalone: omp-session (--watch) + vite (proxies /events+/command+/download), ports chosen per-run
bun run dev:server             # just the daemon, --watch
bun run dev:web                # just vite
bun run check:types            # tsgo -p tsconfig.json --noEmit  (tsgo = @typescript/native-preview, NOT tsc)
bun run lint                   # oxlint (.oxlintrc.json); warnings only don't fail the run
bun run format                 # oxfmt, writes in place (TS/TSX only)
bun run format:check           # oxfmt --check, exits nonzero on unformatted files
bun run test                   # scripts/test.ts wrapper → bun test (see Testing)
bun scripts/test.ts --bail 1   # extra args forwarded to bun test (file filters work: `bun scripts/test.ts server/omp-session.test.ts`)
bun run bench                  # scripts/bench-tests.ts: run [--runs N]|report [--last N]|flakes [--last N]|baseline; per-file stats (mean/sd/p50/p95/CV%, Welch t vs baseline), flake/broken classification, JSONL history in .bench/
bun run build                  # → dist-bundle/cli.js installable bundle (vite build → regenerate server/embedded-dist.ts → bun build, dist/ + @oh-my-pi/* external; shebang verified)
bun run build:web              # vite build → dist/ (gitignored; the UI half of `build`)
bun scripts/test-onboard.ts    # OFFLINE distribution+onboarding E2E: pack → poisoned-store pinned install → first-run config → bare serve → spawn → update round-trip (exit 0 = green)
bun run fleet -- serve|sessions|projects|spawn|add-repo|add|provision|stop|remove|rm-project|add-worktree|rm-worktree|prompt
bun run collab [-- --join|--stop]   # collab room CLI (TUI/CLI-only surface)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nibblebot/omp-web](https://github.com/nibblebot/omp-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-24 -->
