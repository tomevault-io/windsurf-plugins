---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Rust API proxy exposing free DeepSeek model endpoints. Translates standard OpenAI-compatible requests to DeepSeek's internal protocol with account pool rotation, PoW challenge handling, and streaming response support.

Requires Rust **1.94.1** (pinned in `rust-toolchain.toml`) with **edition 2024**.

## Principles

### 1. Single Responsibility
- `config.rs`: Configuration loading only, no client creation or business logic
- `client.rs`: Raw HTTP calls only, no token caching, retry, or SSE parsing
- `accounts.rs`: Account pool management only, no network requests
- `pow.rs`: WASM computation only, no account management or request sending
- `server/handlers.rs`: Route handling only, delegates to OpenAIAdapter
- `server/stream.rs`: SSE response body only, no business logic
- `server/error.rs`: Error mapping only, no business logic

### 2. Minimal Viable
- No premature abstractions: Extract traits/structs when needed, not before
- No redundant code: Remove unused imports, avoid over-documenting, no pre-written tests
- Delay dependency introduction: only add deps when actually needed

### 3. Control Complexity
- Explicit over implicit: Dependencies injected via parameters, no global state
- Composition over inheritance: Small modules composed via functions, no deep inheritance
- Clear boundaries: Modules interact via explicit interfaces, no internal logic leakage

## Architecture

```
src/
├── main.rs                      # Entry point: boots axum HTTP server via server::run()
├── lib.rs                       # Library exports
├── config.rs                    # Config loader: -c flag, config.toml default
├── ds_core.rs                   # DeepSeek module facade (DeepSeekCore, CoreError)
├── ds_core/
│   ├── accounts.rs              # Account pool: init validation, round-robin selection
│   ├── pow.rs                   # PoW solver: WASM loading, DeepSeekHashV1 computation
│   ├── completions.rs           # Chat orchestration: SSE streaming, account guard
│   └── client.rs                # Raw HTTP client: API endpoints, zero business logic
├── openai_adapter.rs            # OpenAI adapter: OpenAIAdapter, OpenAIAdapterError, StreamResponse
├── openai_adapter/
│   ├── types.rs                 # OpenAI protocol types (request + response structs)
│   ├── models.rs                # Model list/get endpoints
│   ├── request.rs               # Request parsing facade + submodules
│   ├── request/
│   │   ├── normalize.rs         # Request normalization/validation
│   │   ├── prompt.rs            # ChatML prompt construction (<|im_start|>/<|im_end|>)
│   │   ├── resolver.rs          # Model name to internal type resolution
│   │   └── tools.rs             # Tool definition extraction and injection
│   ├── response.rs              # Response conversion facade + submodules
│   └── response/
│       ├── sse_parser.rs        # SSE byte stream to DsFrame event stream
│       ├── state.rs             # DeepSeek patch state machine
│       ├── converter.rs         # DsFrame to OpenAI chunk conversion
│       └── tool_parser.rs       # XML <tool_calls> detection/parse
├── server.rs                    # HTTP server: axum router, auth middleware, shutdown
├── server/
│   ├── handlers.rs              # Route handlers: chat_completions, list_models, get_model
│   ├── stream.rs                # SseBody: StreamResponse → axum Body
│   └── error.rs                 # ServerError: OpenAI-compatible error JSON responses
```

**Additional files not in src/**:
- `examples/openai_adapter_cli/` — JSON request samples (basic_chat, reasoning_search, stop_sequence, stream_options, tool_call)
- `examples/*-script.txt` — Scripted input for CLI examples
- `py-e2e-tests/` — Python end-to-end test suite (gitignored runtime artifacts)

## Key Architectural Patterns

### Account Pool Model
1 account = 1 session = 1 concurrency. Scale via more accounts in `config.toml`.

### Request Flow
`v0_chat()` → `get_account()` → `compute_pow()` → `edit_message(payload)` → `GuardedStream`

`completions.rs` hardcodes `message_id: 1` in `EditMessagePayload` because the health check during initialization already writes message 0 into the session.

### GuardedStream & Account Lifecycle
`AccountGuard` marks an account as `busy` and automatically releases it on `Drop`. `GuardedStream` wraps the SSE stream with an `AccountGuard`, so the account is held busy until the stream is fully consumed or dropped. This binds account concurrency to stream lifetime without explicit cleanup logic.

### Account Initialization Flow
`AccountPool::init()` spins up all accounts concurrently. Per-account initialization (`try_init_account`) follows:
1. `login` — obtain Bearer token
2. `create_session` — create chat session
3. `health_check` — send a test completion (with PoW) to verify the session is writable
4. `update_title` — rename session to "managed-by-ai-free-api"

Health check is required because an empty session will fail on `edit_message` with `invalid message id`.

### Request Pipeline
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NIyueeE/ds-free-api](https://github.com/NIyueeE/ds-free-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
