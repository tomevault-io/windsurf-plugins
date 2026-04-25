---
trigger: always_on
description: **AI-powered assistant for communities — React + Tauri v2 desktop app with a Rust core (JSON-RPC / CLI) and sandboxed QuickJS skills.**
---

# OpenHuman

**AI-powered assistant for communities — React + Tauri v2 desktop app with a Rust core (JSON-RPC / CLI) and sandboxed QuickJS skills.**

This file orients contributors and coding agents. Authoritative narrative architecture: [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md). Frontend layout: [`docs/src/README.md`](docs/src/README.md). Tauri shell: [`docs/src-tauri/README.md`](docs/src-tauri/README.md).

---

## Repository layout

| Path                    | Role                                                                                                                                                                                                        |
| ----------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`app/`**              | Yarn workspace **`openhuman-app`**: Vite + React (`app/src/`), Tauri desktop host (`app/src-tauri/`), Vitest tests                                                                                          |
| **Repo root `src/`**    | Rust library **`openhuman_core`** and **`openhuman`** CLI binary entrypoint (`src/main.rs`) — `core_server`, `openhuman::*` domains, skills runtime (QuickJS / `rquickjs`), MCP routing in the core process |
| **Skills registry**     | **[`tinyhumansai/openhuman-skills`](https://github.com/tinyhumansai/openhuman-skills)** on GitHub — canonical skill packages and TS build; not vendored in this tree (see blurb below).                     |
| **`Cargo.toml`** (root) | Core crate; `cargo build --bin openhuman` produces the sidecar the UI stages via `app`’s `core:stage`                                                                                                       |
| **`docs/`**             | Architecture and module guides (numbered pages under `docs/src/`, `docs/src-tauri/`)                                                                                                                        |

Commands in documentation assume the **repo root** unless noted: `yarn dev` runs the `app` workspace.

**Skills registry:** Skill sources and the bundler live in **[github.com/tinyhumansai/openhuman-skills](https://github.com/tinyhumansai/openhuman-skills)**. Clone that repository to author or change skills (`yarn install`, `yarn build`). The desktop app’s skills catalog defaults to that GitHub slug; override with `VITE_SKILLS_GITHUB_REPO` (see [`app/src/utils/config.ts`](app/src/utils/config.ts)).

---

## Runtime scope

- **Shipped product**: desktop — Windows, macOS, Linux (see [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) “Platform reach”).
- **Tauri host** (`app/src-tauri`): **desktop-only** (`compile_error!` for non-desktop targets). Do not add Android/iOS branches inside `app/src-tauri`.
- **Core binary** (`openhuman`): spawned/staged as a **sidecar**; the Web UI talks to it over HTTP (`core_rpc_relay` + `core_rpc` client), not by re-implementing domain logic in the shell.

**Where logic lives**

- **Rust (`openhuman` / repo root `src/`)**: **Business logic and execution**—domains, skills runtime, RPC, persistence, and CLI behavior. This is the authoritative place for rules and side effects.
- **Tauri + React (`app/`)**: **Interaction and UX**—screens, navigation, input, accessibility, windowing, and bridging to the core. The shell presents and orchestrates; it does not duplicate core business rules.

---

## Commands (from repository root)

```bash
# Frontend + Tauri dev (workspace delegates to app/)
yarn dev

# Desktop with Tauri (loads env via scripts/load-dotenv.sh)
yarn tauri dev

# Production UI build (app workspace)
yarn build

# Typecheck / lint / format (app workspace)
yarn typecheck
yarn lint
yarn format
yarn format:check

# Stage openhuman core binary next to Tauri resources (required for core RPC)
cd app && yarn core:stage

# Skills — develop in the GitHub registry repo, then build (see tinyhumansai/openhuman-skills).
# If you keep a local clone path wired in app scripts, you can also run:
yarn workspace openhuman-app skills:build
yarn workspace openhuman-app skills:watch

# Rust — core library + CLI (repo root)
cargo check --manifest-path Cargo.toml
cargo build --manifest-path Cargo.toml --bin openhuman

# Rust — Tauri shell only
cargo check --manifest-path app/src-tauri/Cargo.toml
```

**Tests**: Vitest in `app/` (`yarn test`, `yarn test:coverage`). Rust tests via `cargo test` at repo root as wired in `app/package.json`.

**Quality**: ESLint + Prettier + Husky in the `app` workspace.

---

## Configuration

Environment variables are documented in two `.env.example` files:

- **[`.env.example`](.env.example)** (repo root) — Rust core, Tauri shell, backend URL, logging, proxy, storage, web search, local AI binary overrides. Loaded via `source scripts/load-dotenv.sh`.
- **[`app/.env.example`](app/.env.example)** — Frontend `VITE_*` vars (core RPC URL, backend URL, Sentry DSN, skills repo, dev helpers). Copy to `app/.env.local` for local overrides.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tinyhumansai/openhuman](https://github.com/tinyhumansai/openhuman) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
