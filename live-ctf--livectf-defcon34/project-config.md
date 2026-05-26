---
trigger: always_on
description: Infrastructure for the LiveCTF bot-game competition at DEFCON 34. Teams submit bots (compiled to a custom architecture) that compete in an automated game. The system handles bot submission, queueing games, running them in sandboxed containers, and publishing results and replays.
---

# LiveCTF DEFCON34 — Codebase Guide

## What this is

Infrastructure for the LiveCTF bot-game competition at DEFCON 34. Teams submit bots (compiled to a custom architecture) that compete in an automated game. The system handles bot submission, queueing games, running them in sandboxed containers, and publishing results and replays.

## Repo structure

```
/
├── bot-game/           # All Rust code, Docker images, deployment config, and frontend
├── ansible/            # Server provisioning (run once after Terraform)
├── infrastructure/     # Hetzner Cloud Terraform (servers, network, firewalls, DNS)
├── infrastructure-bootstrap/  # Terraform for the TF state bucket (run once ever)
├── docs/               # cicd.md — deployment design decisions
├── Makefile            # Top-level shortcuts (infra, ansible, compose)
└── README.md           # Deployment runbook
```

## Frontend (`bot-game/frontend/`)

Vue 3 + Pinia SPA built with Vite and pnpm. Serves the team-facing UI (leaderboard, games, bots, profile, watch mode). The WASM game viewer is embedded at `/game/` as a separate static asset served by the same nginx container.

- Package manager: **pnpm** (not npm). Use `pnpm install`, `pnpm run dev`, `pnpm run build`.
- Dev server proxies `/api` to `http://localhost:3000` (configured in `vite.config.ts`).
- `.env.development` sets `VITE_USE_GAME_SHIM=true` and `VITE_USE_MOCKS=true` — MSW intercepts all API calls in dev so no backend is needed.
- `GameViewer.vue` dynamically imports either `GameViewerShim.vue` (dev) or `GameViewerWasm.vue` (prod) based on `VITE_USE_GAME_SHIM`. With no `.env.production`, the shim branch is constant-folded out of prod builds, so `GameViewerShim.vue` and its `lz4js` dependency are fully tree-shaken (zero prod bundle cost). Both viewers auto-detect the LZ4-frame magic and handle compressed or plain replays.
- The WASM viewer calls `import('/game/game.js')` → `init('/game/game_bg.wasm')` → `start(canvasSelector, replayUrl)`. These are typed via `src/wasm.d.ts` with `paths` in `tsconfig.json`.
- `Dockerfile.frontend` has a `vue-builder` stage (node:22-alpine + pnpm) that runs before the WASM build stages; the Vue dist is copied to nginx root, WASM to `/game/`.

## Cargo workspaces

The code is split into **five independent Cargo workspaces** under `bot-game/`. There is no workspace at the `bot-game/` level itself.

| Workspace | Path | Crates |
|---|---|---|
| backend | `bot-game/backend/` | `api`, `entities`, `migration` |
| cli | `bot-game/cli/` | `cli` (binary: `livectf`) |
| coordinator | `bot-game/coordinator/` | `coordinator` (single-crate workspace) |
| game | `bot-game/game/` | `game` (single-crate workspace) |
| runner | `bot-game/runner/` | `compiler`, `compiler-bin`, `driver`, `emulator`, `game-common`, `game-engine` |

Always `cd` into the relevant workspace before running `cargo` commands. There is no top-level workspace to run them from.

## Crate overview

| Crate | Workspace | Description |
|---|---|---|
| `api` | backend | Axum REST API for teams and admins. SeaORM + PostgreSQL, object storage via `object_store`. Checks for pending migrations at startup and refuses to start if any exist. |
| `entities` | backend | SeaORM database entity models, shared by `api` and `migration`. Includes `Phase` (phase1–phase6) and `RunStatus` (pending/queued/running/finished/failed) as PostgreSQL native enums. **Auto-generated** — do not hand-edit the files in `src/entities/`. Manual trait impls (Display, FromStr, JsonSchema) live in `src/impls.rs`. |
| `migration` | backend | SeaORM schema migrations. Run with `sea-orm-cli migrate` or the `migration` binary. |
| `cli` | cli | Command-line client for the API (`livectf`). Covers every endpoint. Config via `Config.toml`, overridable with `--api-url` and `--token`. |
| `coordinator` | coordinator | Polls the API for queued games, auto-detects its phase by running `driver --phase`, downloads bots, runs `driver` under nsjail, reads back replay/scores files, uploads artifacts and final scores. No internal path dependencies. |
| `game` | game | Bevy-based game visualizer, compiles to WASM for browser playback. Path-depends on `game-common` from the runner workspace. Exports `start(canvas_selector: &str, replay_url: &str)` via `wasm_bindgen`; uses `#[wasm_bindgen(main)]` as the wasm32 entry point. |
| `compiler` | runner | Compiles the custom bot language to emulator bytecode. Depends on `emulator`. |
| `compiler-bin` | runner | CLI wrapper around `compiler`: `--source-path <dir> --binary-path <dir>` compiles every `.bot` in the source dir to `<stem>.bin`. Used by the runner Dockerfile to compile NPC bots. |
| `driver` | runner | Runs a single game: executes bots on the `emulator`, drives `game-engine`, writes replay/log/score **files** into `--output-path`. Feature-gated per competition phase. Supports `--phase` / `--version`. See "Game execution pipeline". |
| `emulator` | runner | Custom CPU architecture emulator. No internal dependencies. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Live-CTF/LiveCTF-DEFCON34](https://github.com/Live-CTF/LiveCTF-DEFCON34) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-26 -->
