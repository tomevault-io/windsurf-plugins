---
trigger: always_on
description: This document describes how to work in this repo using idiomatic Rust patterns and the current module layout.
---

# Repository Guidelines

This document describes how to work in this repo using idiomatic Rust patterns and the current module layout.

## Project Structure

- src/
  - lib.rs — library crate root; exposes `agent`, `fs`, `logging`; re-exports `CodexAgent`, `SessionManager`, `FsBridge`, and `init_from_env` for embedders.
  - main.rs — binary entrypoint; initializes tracing, loads config + profiles, boots the filesystem bridge, and wires the ACP runtime. Pass `--acp-fs-mcp` to run the standalone filesystem MCP server.
  - logging.rs — tracing init helpers driven by env variables (`init_from_env`), including optional file logging and daily rotation.
  - agent/
    - mod.rs — Agent trait façade; wires ACP trait methods into `CodexAgent` and re-exports `ClientOp`.
    - core.rs — defines `CodexAgent` and `ClientOp`, handles initialize/auth/new/load session, and applies session mode/model mutations while coordinating the auth/conversation managers.
    - prompt.rs — streaming prompt pipeline, slash-command detection, tool + reasoning updates, cancel handling, and extension method stubs.
    - commands.rs — slash command registry (`AVAILABLE_COMMANDS`) plus helpers like `handle_slash_command` and `/status` rendering.
    - events.rs — Codex Event → ACP update formatting (`EventHandler`), approval option helpers, and `ReasoningAggregator` for thought text.
    - config_builder.rs — builds per-session `Config` instances, injects filesystem guidance, and produces MCP server configs (`prepare_fs_mcp_server_config`, `build_mcp_server`).
    - session_manager.rs — session state store (`SessionState`, `SessionManager`), conversation caching, capability tracking, notification helpers, and `apply_context_override`.
    - utils.rs — shared helpers for session modes/models, provider detection, tool formatting (`format_command_call`, `describe_mcp_tool`, `is_custom_provider`, etc.).
  - fs/
    - mod.rs, bridge.rs, mcp_server.rs — filesystem bridge runtime (`FsBridge::start`) and MCP server entry (`run_mcp_server`) communicating via `ACP_FS_BRIDGE_ADDR`/`ACP_FS_SESSION_ID`.
- Cargo.toml, rust-toolchain.toml
- README.md, AGENTS.md
- Makefile, scripts/stdio-smoke.sh

## Build, Test, Run

- `cargo check` — fast type pass.
- `cargo build` — compile.
- `cargo fmt --all` — format with rustfmt.
- `cargo clippy -- -D warnings` — lint and deny warnings.
- `cargo test` — run unit tests.
- `RUST_LOG=info cargo run --quiet` — run the agent over stdio.
- `make smoke` — run a simple stdio JSON-RPC smoke test.

## Coding Style & Conventions

- Rust 2024 edition; 4-space indentation; rustfmt enforced.
- Unsafe: forbidden at crate root (`#![forbid(unsafe_code)]`).
- Naming: snake_case (fns/vars), CamelCase (types/traits), SCREAMING_SNAKE_CASE (consts).
- Imports: group `std`, external crates, then local modules; avoid unused imports.
- Visibility: default to private; prefer `pub(crate)` over `pub` unless part of the public API.
- Errors: convert external errors early; map to ACP `Error` at boundaries. Use `anyhow` internally where appropriate.
- Logging: `tracing`; control via `RUST_LOG`.

## Testing Guidelines

- Keep tests deterministic (avoid timing races); prefer current-thread executors (`LocalSet`) for async tests when needed.
- Name tests by behavior in snake_case (e.g., `is_read_only_detection`).
- Place small unit tests inline with `#[cfg(test)]` in the same module, or create a dedicated tests module under `src/agent/` if grouping makes sense.

## Pull Requests & Commits

- Conventional Commits: `feat:`, `fix:`, `refactor:`, `docs:`, `test:`, etc.
- PRs include: problem statement, approach, linked issues, and a test plan (commands run, expected output). Include brief `RUST_LOG` snippets when relevant.

## Security & Configuration

- Auth: use `codex login` or `OPENAI_API_KEY`.
- Do not commit secrets (API keys, auth.json); rely on env/OS keychain.
- First build fetches git dependencies; subsequent builds are cached.

## Agent-Specific Notes

- Add/extend slash commands in `src/agent/commands.rs` (advertised via `AVAILABLE_COMMANDS`).
- Use `events::EventHandler` to construct ACP updates; aggregate reasoning with `ReasoningAggregator`.
- Session mode/model helpers now live in `agent::utils` (`session_modes_for_config`, `available_modes`, `find_preset_by_mode_id`, `is_custom_provider`, etc.).
- `SessionManager` is available via `codex_acp::SessionManager` and can be accessed from `CodexAgent` using the `session_manager()` method; it exposes `current_mode()`, `is_read_only()`, `resolve_acp_session_id()`, `support_terminal()`, and `apply_context_override()`.
- Queue client-side interactions through `ClientOp` (`RequestPermission`, `ReadTextFile`, `WriteTextFile`) rather than calling transport code directly.
- Prefer the session manager and filesystem bridge abstractions for capability checks and FS requests.

## Custom Provider Support

### Authentication
The agent supports custom (non-builtin) model providers through a dedicated authentication flow:

- Builtin providers: `openai` (uses existing ChatGPT or API key auth)
- Custom providers: any other provider configured in `model_providers`

When a custom provider is configured:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cola-io/codex-acp](https://github.com/cola-io/codex-acp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
