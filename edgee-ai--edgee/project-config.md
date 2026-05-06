---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Edgee is an **open-source AI Gateway** written in Rust. It sits between coding agents (Claude Code, Codex, OpenCode — Cursor and OpenClaw coming soon) or any llm client and LLM providers (Anthropic, OpenAI) and compresses token-heavy traffic on the fly. A hosted / edge version of the same gateway is available at [`www.edgee.ai`](https://www.edgee.ai); **this repository is the OSS core** you can self-host.

The distinguishing feature is the compression engine. Today it ships a single technique — **tool-results compression** — but the architecture is explicitly designed to host **multiple composable techniques** that a developer selects and combines per request. When extending compression, add a new technique alongside the existing ones rather than threading a new code path through the provider dispatch layer.

**Verify correct installation:**
```bash
edgee --version  # Should show "edgee 0.2.2" (or newer)
edgee stats      # Should show token savings stats (NOT "command not found")
```

If `edgee stats` fails, you have the wrong package installed.

## CLI surface

Entry point: `crates/cli/src/main.rs`. Subcommands declared in `crates/cli/src/commands/mod.rs`:

- `edgee launch {claude|codex|opencode}` — launches the agent with `ANTHROPIC_BASE_URL` and custom headers pointing at the local gateway. Implementation per agent under `crates/cli/src/commands/launch/`.
- `edgee auth {login|status|list|switch}` — OAuth-style flow against the Edgee console. See `crates/cli/src/api.rs` and `crates/cli/src/commands/auth/`.
- `edgee stats` (visible alias `report`) — prints session token counts and compression savings.
- `edgee alias` — installs shell aliases for quick access.
- `edgee reset` — clears credentials.
- `edgee self-update` — compiled in only under the `self-update` feature.

Global flag: `-p/--profile` overrides the active profile.

## Development Commands

### Build & Run
```bash
cargo build                   # raw
cargo build --release         # release build (optimized)
cargo run -- <command>        # run directly
cargo install --path .        # install locally
```

### Testing
```bash
cargo test                    # all tests
cargo test <test_name>        # specific test
cargo test <module_name>::    # module tests
cargo test -- --nocapture     # with stdout
```

### Linting & Quality
```bash
cargo check                   # check without building
cargo fmt                     # format code
cargo clippy --all-targets    # all clippy lints
```

### Pre-commit Gate
```bash
cargo fmt --all && cargo clippy --all-targets && cargo test --all
```

### Package Building
```bash
cargo deb                     # DEB package (needs cargo-deb)
cargo generate-rpm            # RPM package (needs cargo-generate-rpm, after release build)
```

## Workspace layout

Cargo workspace (resolver 3), members under `crates/`:

| Crate | Path | Purpose |
|---|---|---|
| `edgee-cli` | `crates/cli` | The `edgee` binary. Launches coding agents, manages auth / profiles / session stats. |
| `edgee-ai-gateway-core` | `crates/gateway-core` | Canonical request/response types, `Provider` trait, passthrough services, `ProviderDispatchService`. No hard tokio/reqwest dependency — runs on WASM/Fastly too. |
| `edgee-compressor` | `crates/compressor` | Pure compression library. Per-tool and per-bash-command strategies. No I/O. |
| `edgee-compression-layer` | `crates/compression-layer` | Tower `Layer` / `Service` that applies `edgee-compressor` to in-flight requests. |


## Architecture — request flow

The gateway is a Tower `Service` chain:

```text
CompletionRequest
      │
      v
┌──────────────────────┐
│  [User layers]       │  ← Any tower::Layer (compression, logging, …)
└──────┬───────────────┘
       │
       v
┌──────────────────────┐
│  ProviderDispatch    │  ← Service<CompletionRequest>
│  Service             │
└──────────────────────┘
       │
       v
 GatewayResponse
```

The canonical format is OpenAI-Chat-Completions-compatible. `ProviderDispatchService` is intended to translate that into each provider's native format — **today it is a stub** (`crates/gateway-core/src/service.rs:65-71` returns `"not yet implemented"`).

The working path today is **passthrough**: provider-native bodies are forwarded without translation. Two passthrough services:

- `AnthropicPassthroughService` — `POST /v1/messages` (`crates/gateway-core/src/passthrough/anthropic.rs`)
- `OpenAIPassthroughService` — `POST /v1/responses` (`crates/gateway-core/src/passthrough/openai.rs`)

Both strip hop-by-hop and gateway-internal headers before forwarding. The HTTP backend is abstracted behind `HttpClient` (`crates/gateway-core/src/backend/http.rs`); enable the `tokio` feature to get `ReqwestHttpClient`, or implement `HttpClient` yourself for a different runtime.

## Token compression — current state & roadmap

### Today: tool-results compression


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [edgee-ai/edgee](https://github.com/edgee-ai/edgee) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
