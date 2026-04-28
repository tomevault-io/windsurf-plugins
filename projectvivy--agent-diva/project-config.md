---
trigger: always_on
description: This repository is a Rust workspace. Crates are organized by responsibility:
---

# Repository Guidelines

## Project Structure & Module Organization

This repository is a Rust workspace. Crates are organized by responsibility:

- `agent-diva-core`: shared config, memory/session, cron, heartbeat, and event bus foundations.
- `agent-diva-agent`: agent loop, context assembly, skill/subagent flow.
- `agent-diva-providers`: LLM/transcription provider abstractions and implementations.
- `agent-diva-channels`: channel adapters (Slack, Discord, Telegram, Email, QQ, etc.).
- `agent-diva-tools`: built-in tools (filesystem, shell, web, cron, spawn).
- `agent-diva-neuron`: supporting types/helpers used heavily by the desktop GUI.
- `agent-diva-manager`: default local gateway and HTTP control plane for **`agent-diva-cli`** (hard dependency; no `nano` feature in CLI).
- `agent-diva-nano`: **template-line** local gateway stack in **`external/agent-diva-nano/`** (nested workspace; `cd external && cargo build -p agent-diva-nano`); not a root workspace member.
- `agent-diva-cli`: user-facing CLI entrypoint (`agent-diva` binary).
- `agent-diva-service`: Windows service wrapper.
- `agent-diva-gui`: optional Tauri desktop app (separate from default CLI `cargo` closure).
- `agent-diva-migration`: migration utility from earlier versions.

Use each crate's `src/` for code; add crate-level integration tests under `tests/` when needed.

**Common workspace conventions:**

- Keep cross-cutting domain types in `agent-diva-core`.
- Keep provider/channel-specific code isolated in their dedicated crates.
- Prefer adding small modules over large monolithic files.
- Place examples and fixtures under the owning crate when practical.

## Getting Started

- Install Rust stable (via `rustup`) and ensure `cargo`, `rustfmt`, and `clippy` are available.
- Install `just` and run commands from the workspace root.
- Copy and configure local environment files if required by a crate or channel.
- Verify toolchain and project health with `just fmt-check && just check && just test`.

## Development Guide
If users request references to projects such as openclaw, nanobot, or shannon, prioritize reviewing the contents under the .workspace directory. Analyze the architectures of these sibling projects and propose a development approach suitable for the agent-diva architecture.

## Build, Test, and Development Commands

Prefer `just` recipes from the workspace root:

- `just build` / `just build-release`: build all crates (debug/release).
- `just test`: run `cargo test --all`.
- `just check`: run clippy with warnings denied.
- `just fmt` and `just fmt-check`: format or verify formatting.
- `just ci`: run formatting, lint, and tests (CI-equivalent gate).
- `just run -- <args>`: run `agent-diva-cli`.
- `just migrate -- <args>`: run migration CLI.

**Useful direct cargo invocations:**

- `cargo test -p <crate>`: run tests for a single crate.
- `cargo test <test_name>`: run a specific test by name.
- `cargo run -p agent-diva-cli -- <args>`: run CLI without `just`.

## Coding Style & Naming Conventions

Use Rust 2021 conventions and keep `rustfmt` output authoritative (`rustfmt.toml` is checked in). Use:

- `snake_case` for modules/functions/files,
- `PascalCase` for structs/enums/traits,
- `SCREAMING_SNAKE_CASE` for constants.

Keep public APIs documented with `///`; use `//!` for module overviews when helpful. Run `cargo clippy --all -- -D warnings` before opening a PR.

**Additional style guidance:**

- Favor small, composable functions and explicit types at API boundaries.
- Avoid `unwrap`/`expect` in non-test code; propagate or map errors with context.
- Keep async boundaries clear; avoid blocking calls inside async paths.
- Preserve backward compatibility for public interfaces unless a breaking change is intentional and documented.

## Provider Model-ID Safety Rule (Critical)

When calling a provider's native OpenAI-compatible endpoint (e.g., DeepSeek `https://api.deepseek.com/v1`), always send the provider's raw model ID (e.g., `deepseek-chat`) — **do not auto-add LiteLLM prefixes** (e.g., do *not* rewrite to `deepseek/deepseek-chat`).  
Only apply `provider/model` prefix rewriting when routing through a true LiteLLM-style gateway or aggregator.

**Implementation checklist for provider routing changes:**

- Verify whether the endpoint is native-provider or LiteLLM-compatible gateway.
- Add/adjust tests that assert final outbound `model` value.
- Confirm config migration paths do not silently rewrite model IDs incorrectly.
- Document behavior in crate-level docs or README when introducing new providers.

## Testing Guidelines

Write focused unit tests near the code with `#[cfg(test)]`. Add integration tests in crate `tests/` folders for cross-module behavior. Run `cargo test --all` locally before pushing. Use workspace test utilities (`tokio-test`, `tempfile`, `wiremock`, `mockito`) where appropriate.

**Test quality expectations:**

- Cover both success paths and representative failure paths.
- Validate serialization/config parsing for new config fields.
- Prefer deterministic tests; avoid external network calls in unit/integration tests.
- For async behavior, include timeout-aware assertions to avoid hanging CI.

## Configuration, Secrets, and Safety


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ProjectViVy/agent-diva](https://github.com/ProjectViVy/agent-diva) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
