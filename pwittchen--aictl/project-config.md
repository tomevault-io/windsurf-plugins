---
trigger: always_on
description: Guidance for Claude Code when working in this repository.
---

# CLAUDE.md

Guidance for Claude Code when working in this repository.

See [README.md](README.md) for user-facing docs, [docs/ARCH.md](docs/ARCH.md) for architecture detail, and [docs/SERVER.md](docs/SERVER.md) for the HTTP server (`aictl-server`). This file is the compact reference for code changes.

## Build & Run

```bash
cargo build                    # debug build (workspace, all members)
cargo build --release          # release build
cargo run --bin aictl -- <args># run the CLI binary with arguments
cargo lint                     # clippy pedantic (alias in .cargo/config.toml; lints default-members — desktop excluded)
cargo fmt                      # format
cargo test                     # run tests across the workspace
```

## Workspace layout

Four-crate Cargo workspace; `aictl-desktop` is excluded from `default-members` so a bare `cargo build` / `cargo lint` / `cargo test` keeps working without Tauri's deps. CI builds the desktop separately on macOS only.

- `crates/aictl-core/` — package `aictl-core`, library crate (lib name `aictl_core`). Hosts the agent loop, providers, tools, security, sessions, audit log, MCP/plugin/hook systems, and the `aictl_core::ui::AgentUI` trait that frontends implement. Does not link any terminal library; every side effect routes through `AgentUI` (or `aictl_core::ui::warn_global` for runtime warnings).
- `crates/aictl-cli/` — package `aictl-cli`, binary crate (`[[bin]] name = "aictl"`). Hosts the REPL, slash-command UI, status banner, and the `PlainUI` / `InteractiveUI` impls of `AgentUI` (crossterm + indicatif + termimad + rustyline live here). Re-exports `aictl-core`'s modules under `crate::*` for legacy import paths.
- `crates/aictl-server/` — package `aictl-server`, binary crate (`[[bin]] name = "aictl-server"`). OpenAI-compatible HTTP LLM proxy: `POST /v1/chat/completions`, `POST /v1/completions`, `POST /v1/messages`, `GET /v1/models`, `GET /v1/stats`, `GET /healthz`. Pure proxy — no agent loop, no tool dispatch, no agents/skills/sessions. The OpenAI-shaped routes translate to the engine's `Vec<Message>` and dispatch via `aictl_core::llm::call_<provider>`. `POST /v1/messages` is dual-mode (see [`messages.rs`](crates/aictl-server/src/messages/mod.rs)): Anthropic models always pass through `messages::passthrough` verbatim to `api.anthropic.com/v1/messages` (tool_use / tool_result blocks untouched, prompt caching / extended thinking / `anthropic-beta` features intact); non-Anthropic models are rejected with `400 model_not_found` unless `AICTL_SERVER_MESSAGES_CROSS_PROVIDER=true`, in which case they route through `messages::translator` — a per-provider HTTP round-trip (not `aictl_core::llm::call_*` — those use the engine's XML tool format) that translates the Anthropic request shape into OpenAI / Gemini / Ollama native shapes and back. The translator owns its own dispatch, native `tools[]` / `tool_calls[]` survive the round-trip, and provider streams bridge to Anthropic's structured SSE event sequence (`message_start` / `content_block_start` / `content_block_delta` / `content_block_stop` / `message_delta` / `message_stop`) via state machines under `messages/translator/stream/`. Unsupported Anthropic features (`cache_control`, `thinking`, PDF blocks, URL images on Gemini/Ollama) flow through `messages::translator::feature_gate` with `strip` / `warn` / `reject` modes (`AICTL_SERVER_MESSAGES_FEATURE_GATE`); GGUF / MLX are rejected on the cross-provider path. Reuses `aictl_core::run::redact_outbound` and `aictl_core::security::detect_prompt_injection` on every path; passthrough logs `gateway:anthropic`, translation logs `gateway:messages:<provider>`, plus `feature_dropped:<provider>` entries when fields are stripped. Master-key gate on every authenticated route; `master_key::resolve` reads `AICTL_SERVER_MASTER_KEY` through `keys::get_secret` (keyring-first, plain config fallback) and persists a generated key via `keys::set_secret` on first launch — so CLI-side `/keys` lock/unlock cycles round-trip the entry without restarting the server. axum 0.8 + tower-http live here only — they never enter `aictl-core`. See [docs/SERVER.md](docs/SERVER.md) for the full reference.
- `crates/aictl-desktop/` — package `aictl-desktop`, Tauri-based macOS desktop app (work-in-progress, unreleased). Excluded from `default-members`; only built explicitly with `cargo build -p aictl-desktop`. Reuses the `aictl-core` engine; no functional drift from the CLI.

Cargo features (`gguf`, `mlx`, `redaction-ner`) live on the `aictl-core` crate; the CLI and server declare them as `aictl-core/<feature>` passthroughs.

## Module map

Submodule trees: `llm/` (providers) and `tools/` (tool impls) live under `crates/aictl-core/src/`. `commands/` (slash-command handlers) lives under `crates/aictl-cli/src/`.

- `crates/aictl-cli/src/main.rs` — CLI (clap), config + security + session init, agent loop driver, single-shot vs REPL
- `crates/aictl-core/src/run.rs` — `run_agent_turn` loop, tool-call dispatch, outbound redaction, stream suspend wiring; also exposes `Provider`, `with_esc_cancel`, `Interrupted`, `build_system_prompt`, `run_agent_single`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pwittchen/aictl](https://github.com/pwittchen/aictl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
