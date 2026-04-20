---
trigger: always_on
description: Rust Agent Development Kit — a modular workspace of publishable crates for building AI agents with tool calling, multi-model support, real-time voice, graph workflows, and more.
---

# ADK-Rust

Rust Agent Development Kit — a modular workspace of publishable crates for building AI agents with tool calling, multi-model support, real-time voice, graph workflows, and more.

## Dev environment

- Rust 1.85.0+, edition 2024. Use `make setup` or `devenv shell` to bootstrap.
- `sccache` is the compilation cache. Set `RUSTC_WRAPPER=sccache` in your shell profile.
- On Linux, `wild` is the linker (configured in `.cargo/config.toml`). macOS uses the default linker.
- Copy `.env.example` to `.env` for API keys. Never commit `.env` files or secrets.
- `adk-mistralrs` is excluded from the workspace (GPU deps). Build it explicitly: `cargo build --manifest-path adk-mistralrs/Cargo.toml`.
- `CMAKE_POLICY_VERSION_MINIMUM=3.5` is needed for cmake 4.x compatibility (audiopus).
- **Performance**: `.cargo/config.toml` sets `incremental = false` globally for `sccache` compatibility, but `Cargo.toml` `[profile.dev]` overrides this with `incremental = true` for local dev builds. CI uses the `ci` profile where the config.toml setting applies.

## Quality gates

Run all three before every commit. CI enforces them — save yourself the round-trip:

```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo nextest run --workspace
```

### AI Agent Workflow (`devenv`)

Use these shorthand scripts instead of raw `cargo` to ensure `sccache` wrap and workspace coverage:

| Action | Command | Description |
| :--- | :--- | :--- |
| **Check** | `devenv shell check` | Fast workspace compilation check. |
| **Test** | `devenv shell test` | Run all non-ignored tests (nextest). |
| **Lint** | `devenv shell clippy` | Clippy with `-D warnings` (zero tolerance). |
| **Format** | `devenv shell fmt` | Enforce Rust Edition 2024 style. |

Run `cargo fmt --all` automatically after finishing Rust code changes; do not ask for approval.

Clippy runs with `-D warnings` — every warning is a compile error. Fix warnings before pushing. Don't suppress with `#[allow(...)]` unless there's a documented reason.

## Workspace structure

Crate names are prefixed with `adk-`. The Rust module name uses underscores (`adk_core`, `adk_agent`, etc.).

### Core crates (publishable to crates.io)

```
adk-core/        Core traits and types: Agent, Tool, Llm, Session, Event, Content, State
adk-agent/       Agent implementations: LlmAgent, CustomAgent, SequentialAgent, ParallelAgent,
                 LoopAgent, ConditionalAgent, LlmConditionalAgent
adk-model/       LLM provider facade: Gemini, OpenAI, Anthropic, DeepSeek, Groq, Ollama,
                 Fireworks, Together, Mistral, Perplexity, Cerebras, SambaNova, Bedrock, Azure AI
                 (feature-gated)
adk-gemini/      Dedicated Gemini client with GeminiBackend trait (Studio + Vertex AI)
adk-tool/        Tool system: FunctionTool, MCP integration (rmcp 1.2), Google Search, MCP Resource API
adk-runner/      Agent execution runtime with event streaming
adk-server/      HTTP server (Axum) and A2A v1.0.0 (Agent-to-Agent) protocol
adk-session/     Session management and state persistence, encrypted sessions (AES-256-GCM)
adk-artifact/    Binary artifact storage for agents
adk-memory/      Semantic memory and RAG search
adk-graph/       Graph-based workflow orchestration with checkpoints, durable resume, and HITL
adk-realtime/    Real-time bidirectional audio/video streaming (OpenAI, Gemini Live, Vertex AI Live,
                 LiveKit, WebRTC), mid-session context mutation, interruption detection
adk-browser/     Browser automation tools via WebDriver
adk-eval/        Evaluation framework: trajectory, semantic, rubric, LLM-judge
adk-telemetry/   OpenTelemetry 0.31 integration for agent observability
adk-guardrail/   Input/output guardrails: validation, content filtering, PII redaction
adk-auth/        Authentication: API keys, JWT, OAuth2, OIDC, SSO
adk-plugin/      Plugin system for agent lifecycle hooks
adk-skill/       Skill discovery, parsing, and convention-based agent capabilities
adk-cli/         Command-line launcher for agents
adk-anthropic/   Dedicated Anthropic API client with streaming, thinking, caching, citations, pricing
adk-doc-audit/   Documentation audit: rustdoc coverage, link checking, crate validation
adk-rust-macros/ Procedural macros (#[tool] attribute)
adk-code/        Code execution (experimental)
adk-sandbox/     Sandboxed execution environments — process/WASM backends, OS-level sandbox profiles
                 (Seatbelt on macOS, bubblewrap on Linux, AppContainer on Windows)
adk-audio/       Audio processing, STT/TTS providers, Deepgram streaming (experimental)
adk-rag/         Retrieval-augmented generation pipelines
adk-action/      Action node execution for deterministic workflow operations
adk-deploy/      Deployment utilities
adk-payments/    Payment integration for agent services
cargo-adk/       Cargo subcommand for project scaffolding
adk-rust/        Umbrella crate re-exporting all of the above
```

### Excluded from workspace

```
adk-mistralrs/   Local LLM inference via mistral.rs v0.8.0 (Gemma 4, Qwen 3.5, Voxtral — GPU deps, build explicitly)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zavora-ai/adk-rust](https://github.com/zavora-ai/adk-rust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
