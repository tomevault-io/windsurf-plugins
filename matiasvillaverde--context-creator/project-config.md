---
trigger: always_on
description: Repository-level guidance for coding agents working on `context-creator`.
---

# AGENTS.md

Repository-level guidance for coding agents working on `context-creator`.

## Scope

- This file applies to the entire repository unless a more specific nested `AGENTS.md` is added later.
- Prefer current source code over older docs when they disagree. Some docs still show older API names, while the Rust modules are authoritative.
- Keep changes narrowly scoped. This project has broad CLI, semantic-analysis, MCP, and security coverage, so unrelated rewrites are expensive to validate.

## Project Overview

`context-creator` is a Rust CLI and library for turning codebases into LLM-friendly context. It walks files, applies ignore/include rules, optionally performs semantic dependency analysis with tree-sitter, prioritizes files under token budgets, formats output, and can run as an MCP server.

Main user-facing surfaces:

- CLI context generation from local paths or GitHub repositories.
- Subcommands: `search`, `diff`, `telemetry`, and `examples`.
- Output styles: markdown, XML, plain text, and paths-only.
- MCP server implementations: legacy jsonrpsee and RMCP.
- LLM tool integration for Gemini, Codex, Claude, and Ollama.

## Toolchain

- Rust edition: 2021.
- MSRV: 1.74.0, as configured in `clippy.toml`.
- CI uses stable Rust with `rustfmt` and `clippy`.
- Optional tools: `make`, `cargo-audit`, `cargo-tarpaulin`, `cargo-watch`, Node.js for the generated npm package workflow.

## High-Value Commands

Run targeted checks while developing, then broaden before finishing risky changes.

```bash
cargo check --all-targets
cargo fmt -- --check
cargo clippy --all-targets --all-features -- -D warnings
cargo test --test lib
```

Make targets wrap the common flows:

```bash
make check
make fmt
make fmt-check
make lint
make test
make test-fast
make test-acceptance
make validate
```

Useful focused test forms:

```bash
cargo test --test lib cli_test::
cargo test --test lib semantic_include_types_test::
cargo test --test lib acceptance:: --no-fail-fast
cargo test --test rmcp_basic_test
cargo test --test mcp_server_test
```

Run the CLI locally:

```bash
cargo run -- --help
cargo run -- examples/sample-rust-project
cargo run -- search "AuthenticationService" src/
cargo run -- diff HEAD~1 HEAD
cargo run -- telemetry -t examples/telemetry/traces.json
cargo run -- --rmcp --rmcp-transport stdio
cargo run -- --rmcp --rmcp-transport http --mcp-port 9090
```

Notes:

- `make test` runs format, lint, `cargo build`, and the consolidated `tests/lib.rs` integration target.
- Use `cargo test` when changing standalone integration tests, MCP tests, or shared behavior not registered in `tests/lib.rs`.
- `cargo audit` may require installing `cargo-audit`; `.cargo/audit.toml` intentionally ignores `RUSTSEC-2025-0009` for the documented jsonrpsee/rustls/ring transitive dependency.

## Source Map

- `src/main.rs`: binary entry point, config loading, logging setup, stdin prompt handling, MCP server startup.
- `src/lib.rs`: library entry point and main processing pipeline.
- `src/cli.rs`: clap definitions, subcommands, validation, LLM tool behavior, token-limit precedence.
- `src/config.rs`: TOML config loading and application to CLI config.
- `src/commands/`: implementations for `search`, `diff`, and `telemetry`.
- `src/core/walker.rs`: path walking, ignore/include pattern handling, file metadata, priority calculation, binary filtering.
- `src/core/context_builder.rs`: context options and output generation.
- `src/core/prioritizer.rs`: token-aware priority selection.
- `src/core/token.rs`: token counting.
- `src/core/project_analyzer.rs`: single-pass project analysis reused by semantic features.
- `src/core/file_expander.rs`: semantic expansion for imports, callers, and types.
- `src/core/search.rs`: text search support.
- `src/core/telemetry/`: OpenTelemetry parsing, correlation, and enrichment.
- `src/core/semantic/`: tree-sitter semantic analysis, graph traversal, language analyzers, parser/cache infrastructure.
- `src/formatters/`: markdown, XML, plain text, and paths renderers behind `DigestFormatter`.
- `src/mcp_server/`: MCP request/response types, jsonrpsee handlers, RMCP server, and MCP cache.
- `src/remote.rs`: remote repository fetch support.
- `src/utils/`: file extension mapping, git utilities, shared error types.
- `tests/lib.rs`: consolidated integration test runner that includes most tests under `tests/modules/`.
- `tests/modules/acceptance/` and `tests/acceptance/`: acceptance coverage and telemetry enrichment fixtures.
- `examples/`: sample projects, configs, and telemetry data for manual testing.
- `.github/workflows/`: CI, release, npm publish, and crates.io publish workflows.

## Architecture Rules

- Preserve the CLI flow: parse `Config`, load config, validate, dispatch subcommands, then run the normal directory-processing pipeline.
- Use `Config` helper methods instead of reading raw fields when behavior depends on precedence or normalization:
  - `get_directories()`
  - `get_prompt()`
  - `get_include_patterns()`
  - `get_ignore_patterns()`
  - `get_effective_max_tokens()`
  - `get_effective_context_tokens()`
- Keep validation in `Config::validate()` for user-facing CLI combinations. Existing flexible combinations are intentional; do not re-add broad mutual exclusions without tests.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [matiasvillaverde/context-creator](https://github.com/matiasvillaverde/context-creator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
