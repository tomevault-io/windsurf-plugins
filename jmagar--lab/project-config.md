---
trigger: always_on
description: `lab` is a pluggable homelab CLI + MCP server SDK in Rust. One binary, **23 services** (22 feature-gated + always-on `extract`), runtime MCP tool selection via a single tool per service with an `action` + `params` dispatch shape (~23 MCP tools max, not hundreds).
---

# Lab — Development Instructions

## What is this?

`lab` is a pluggable homelab CLI + MCP server SDK in Rust. One binary, **23 services** (22 feature-gated + always-on `extract`), runtime MCP tool selection via a single tool per service with an `action` + `params` dispatch shape (~23 MCP tools max, not hundreds).

Start with `docs/README.md` for the docs index. The topic docs in `docs/` are the source of truth; if this file disagrees with them, this file is stale.

Observability is governed by `docs/OBSERVABILITY.md`. When adding or changing request paths, treat that file as the source of truth for logging boundaries, required fields, correlation, redaction, and verification.
Errors are governed by `docs/ERRORS.md`. Serialization and output-boundary rules are governed by `docs/SERIALIZATION.md`.
Shared dispatch ownership and adapter direction are governed by `docs/DISPATCH.md`.

**Build assumption.** This repo is developed and verified as an **all-features** binary. Treat `cargo build --all-features`, `cargo test --all-features` / `cargo test --tests --no-fail-fast`, and the equivalent `just` commands as the default truth. Do not delete or rewrite shared helpers just because they appear unused in a narrow feature slice; first verify whether they are used by other feature-gated services in the normal all-features build.

**Service onboarding rule.** When bringing a service online, prefer scaffold first, audit second, and all-features verification last. New onboarding work should be generated with `lab scaffold service`, checked with `lab audit onboarding`, and only then validated with the all-features test/build path.

**Nested guides.** Subdirectories carry their own `CLAUDE.md` with rules that don't belong at the root. Read the nearest one when working in:
- `crates/lab-apis/src/core/` — trait contracts, error taxonomy, HttpClient invariants
- `crates/lab-apis/src/servarr/` — shared *arr primitives
- `crates/lab-apis/src/extract/` — synthetic-service rules, `.env` merge algorithm
- `crates/lab/src/dispatch/` — shared dispatch layer, required service layout, canonical templates
- `crates/lab/src/mcp/` — dispatch, envelopes, elicitation, catalog
- `crates/lab/src/cli/` — thin-shim pattern, destructive flags, batch commands
- `crates/lab/src/tui/` — plugin manager UX, `.mcp.json` patching
- `crates/lab/src/api/` — axum HTTP surface, status code mapping, middleware stack

## Repository Structure

Two crates. Pure API clients live in `lab-apis`. Everything else (CLI, MCP, TUI, binary) lives in `lab`.

```
lab/
├── crates/
│   ├── lab-apis/                     # PURE Rust SDK — reusable in any binary
│   │   ├── Cargo.toml                # deps: reqwest, serde, thiserror, tokio
│   │   └── src/
│   │       ├── lib.rs                # re-exports, feature gates
│   │       ├── core/                 # HttpClient, Auth, errors, traits
│   │       ├── servarr/              # shared *arr primitives
│   │       ├── radarr/               # { client.rs, types.rs, error.rs }
│   │       ├── sonarr/
│   │       ├── prowlarr/
│   │       ├── plex/
│   │       ├── tautulli/
│   │       ├── sabnzbd/
│   │       ├── qbittorrent/
│   │       ├── tailscale/
│   │       ├── linkding/
│   │       ├── memos/
│   │       ├── bytestash/
│   │       ├── paperless/
│   │       ├── arcane/                # Docker management UI
│   │       ├── unraid/                # Unraid GraphQL API
│   │       ├── unifi/                 # UniFi Network Application local API
│   │       ├── overseerr/              # Media request manager
│   │       ├── gotify/                # Push notifications
│   │       ├── openai/                # OpenAI API (+ OpenAI-compatible)
│   │       ├── qdrant/                # Vector database
│   │       ├── tei/                   # HF Text Embeddings Inference
│   │       ├── apprise/               # Universal notification dispatcher
│   │       └── extract/                # ALWAYS-ON synthetic service: scan local/SSH hosts for service creds
│   │
│   └── lab/                          # BINARY: cli + mcp + tui + main
│       ├── Cargo.toml                # deps: lab-apis, clap, rmcp, ratatui, anyhow, tabled
│       └── src/
│           ├── main.rs
│           ├── api.rs                # axum surface module declaration
│           ├── catalog.rs            # build_catalog() — single source for help/resource/CLI
│           ├── cli/                  # clap subcommands per service (thin shims)
│           ├── cli.rs
│           ├── mcp/
│           │   ├── registry.rs       # runtime tool registration
│           │   ├── resources.rs      # action catalog as MCP resources
│           │   ├── error.rs          # structured JSON errors
│           │   └── services/         # one dispatch module per service
│           ├── mcp.rs
│           ├── api/                  # axum HTTP API (mirrors MCP action dispatch)
│           │   ├── state.rs          # AppState — Catalog + ToolRegistry (Arc-wrapped)
│           │   ├── error.rs          # ApiError + IntoResponse mapping

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jmagar/lab](https://github.com/jmagar/lab) — distributed by [TomeVault](https://tomevault.io/claim/jmagar).
<!-- tomevault:4.0:windsurf_rules:2026-04-17 -->
