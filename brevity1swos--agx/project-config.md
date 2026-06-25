---
trigger: always_on
description: Step-through debugger for your agent. Rust TUI app using ratatui + crossterm + serde. Narrow scope, deep engineering, terminal-native. Consumes Claude Code, Codex CLI, Gemini CLI, OpenTelemetry GenAI JSON, and generic OpenAI-compatible conversation files and renders them as a navigable color-coded timeline with bidirectional tool call ↔ tool result pairing regardless of source format. Per-step token usage and USD cost estimates; exports to Markdown / HTML / JSON.
---

# agx

Step-through debugger for your agent. Rust TUI app using ratatui + crossterm + serde. Narrow scope, deep engineering, terminal-native. Consumes Claude Code, Codex CLI, Gemini CLI, OpenTelemetry GenAI JSON, and generic OpenAI-compatible conversation files and renders them as a navigable color-coded timeline with bidirectional tool call ↔ tool result pairing regardless of source format. Per-step token usage and USD cost estimates; exports to Markdown / HTML / JSON.

## Quick Reference

```bash
cargo build --release                                # Build (release, default features)
cargo build --release --features otel-proto          # Release build with binary OTLP support
cargo build --release --features embedding-search    # Release build with semantic search (fastembed / ONNX)
cargo build --release --features notifications       # Release build with --notify-on-error / --notify-on-idle support
cargo test                                           # Run all tests (feature-off path)
cargo test --features otel-proto                     # Run all tests (feature-on path — prost included)
cargo test --features embedding-search               # Run all tests (feature-on path — fastembed included)
cargo test --features notifications                  # Run all tests (feature-on path — notify-rust included)
cargo clippy --all-targets -- -D warnings            # Lint, default features
cargo clippy --all-targets --features otel-proto -- -D warnings  # Lint with feature on
cargo clippy --all-targets --features embedding-search -- -D warnings  # Lint with semantic feature on
cargo clippy --all-targets --features notifications -- -D warnings  # Lint with notifications feature on
cargo bench --bench agx_bench                        # Run criterion bench suite (parsers + aggregate + corpus)
cargo bench --bench agx_bench -- --save-baseline main  # Capture baseline before a perf-targeted refactor
cargo bench --bench agx_bench -- --baseline main     # Compare after-change run against saved baseline
cargo fmt --check                                    # Format check
cargo fmt                                            # Format apply
cargo audit                                          # Supply chain audit
./target/release/agx assets/sample_session.jsonl             # Claude Code fixture
./target/release/agx assets/sample_codex_session.jsonl       # Codex fixture
./target/release/agx assets/sample_gemini_session.json       # Gemini fixture
./target/release/agx assets/sample_generic_session.json      # OpenAI-compatible fixture
./target/release/agx assets/sample_otel_json_traces.json     # OTel GenAI JSON fixture
./target/release/agx --summary        assets/sample_session.jsonl  # Non-interactive summary with tokens + cost
./target/release/agx --export md      assets/sample_session.jsonl  # Transcript → stdout (md | html | json)
./target/release/agx --debug-unknowns assets/sample_session.jsonl  # Format-drift diagnostics to stderr
./target/release/agx --no-cost        assets/sample_session.jsonl  # Suppress cost estimates
```

## Architecture

```
crates/
└── agx-core/           # Pure parsers / timeline / corpus / pricing / annotations / pii / semantic / notify / export.
                        # Zero TUI deps. Publishable standalone to crates.io for Python / WASM / eval-harness consumers.
src/
├── lib.rs              # Thin library re-exporting everything from agx-core plus the local TUI modules
├── main.rs             # CLI entry point: clap + format dispatch + --summary / --export / --diff branches
├── format.rs           # Format detection — returns ClaudeCode | Codex | Gemini | Generic | OtelJson
├── browser.rs          # Multi-session discovery + picker (scans ~/.claude, ~/.codex, ~/.gemini)
├── session.rs          # Claude Code JSONL parser (Entry enum + serde Deserialize + ClaudeUsage)
├── codex.rs            # Codex CLI JSONL parser (response_item + function_call pairing)
├── gemini.rs           # Gemini CLI single-JSON parser (toolCall splitting + usageMetadata)
├── generic.rs          # OpenAI-compatible conversation parser ({messages: [{role, content, tool_calls}]})
├── langchain.rs        # LangChain / LangSmith run-tree export parser (chain/chat_model/tool)
├── otel_json.rs        # OpenTelemetry GenAI JSON parser (OTLP-JSON envelope + gen_ai.* semconv)
├── otel_proto.rs       # Binary OTLP parser (.pb/.otlp) — feature-gated behind `otel-proto`
├── vercel_ai.rs        # Vercel AI SDK `generateText` / `streamText` result parser (camelCase tool fields, steps[])
├── loader.rs           # `load_session(path)` — format dispatch front door shared by single-session and corpus flows
├── corpus.rs           # `agx corpus <dir>` subcommand: parallel rayon parse, aggregate, filter, text/json output

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brevity1swos/agx](https://github.com/brevity1swos/agx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
