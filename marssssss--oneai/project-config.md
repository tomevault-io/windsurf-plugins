---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

OneAI is a cross-platform AI agent framework in Rust. It is a Cargo workspace of ~24 crates (`crates/*`) plus example binaries (`examples/*`). The README.md is the authoritative architectural reference — read it before making non-trivial changes; it documents the crate map, DomainPack layers, AgentLoop decisions, permission model, and paradigms in detail.

## Build, test, run

```bash
cargo build                      # build whole workspace
cargo test                      # run all tests across crates (see README badge for the current count)
cargo test -p oneai-agent      # tests for a single crate
cargo test -p oneai-agent agent_loop  # a single test/module within a crate
cargo test -p oneai-agent --test e2e_tests   # integration test file by name
cargo clippy --workspace --all-targets   # lints (keep clean — commits commonly fix warnings)
cargo run -p oneai-cli-demo     # launch the interactive TUI demo (bin name: oneai-cli)
```

The workspace uses `resolver = "2"`, `edition = "2021"`, shared version `0.2.0` from `[workspace.package]`, and all shared dependencies are pinned in `[workspace.dependencies]` — add new deps there and reference via `{ workspace = true }` in crate Cargo.tomls.

## Network proxy

All outbound HTTP in OneAI — LLM provider APIs (OpenAI/Anthropic/Ollama/Gemini), `web_search`/`web_fetch` tools, A2A client, embedding services, MCP HTTP transport — goes through `reqwest::Client`. Proxy support is therefore env-var based and works everywhere uniformly:

- `HTTPS_PROXY` / `HTTP_PROXY` / `ALL_PROXY` — proxy URL (auto-detected by reqwest on every client build; no code path opt-in needed).
- `NO_PROXY` — comma-separated exclusion list.
- SOCKS5: set `ALL_PROXY=socks5://host:port` (requires the reqwest `socks` feature, kept on in the workspace `Cargo.toml`).
- On macOS/Windows, reqwest's `system-proxy` feature also reads the OS GUI proxy settings; env vars always win.

Do not wire a custom `reqwest::Client` into individual providers/tools for proxy purposes — rely on the env vars. If a future subsystem needs a bespoke client, build it with `reqwest::Client::builder()` so it still picks up these env vars. The `crates/oneai-provider/tests/proxy_feature.rs` smoke test guards the `socks`/proxy feature flags.

`#[non_exhaustive]` is applied to public enums as part of the v0.2.0 API-stability commitment (P3-1). Preserve it on existing public enums and add it to new externally-facing enum APIs.

## Commit convention

Git commit messages must end with `Co-Authored-By: glm-5.2` (the model actually driving this repo), **not** the default Claude Opus co-author line. Commit messages in this repo are frequently written in Chinese.

## Supply-chain discipline

Reproducibility comes from the **committed `Cargo.lock`** (the workspace ships binaries — `oneai-cli` + examples — so the lockfile is tracked, not gitignored). On top of that, four gates enforce supply-chain integrity (evolution-plan §1.3):

- **`deny.toml`** — `cargo deny check` config: advisories (RustSec), license allowlist, ban on wildcards/unknown sources. Single source of truth; `cargo audit` is NOT used in CI to avoid drifting two parallel ignore lists. Accepted-risk advisories live in `[advisories].ignore` with a per-entry `reason` string — never ignore silently. Install locally: `cargo install cargo-deny && cargo deny check`.
- **`.github/workflows/audit.yml`** — daily cron + PR-triggered `cargo deny check`. PRs only gate on `Cargo.toml`/`Cargo.lock`/`deny.toml` changes.
- **lockfile drift gate** (`ci.yml` `lockfile-gate` job) — a PR that modifies `Cargo.lock` fails CI unless `ONEAI_ALLOW_LOCKFILE_CHANGE=1` is set. To intentionally upgrade a dependency, set that env on the PR (or note it in the commit message and a maintainer re-runs with the env). Push-to-main always passes.
- **`.github/workflows/publish.yml`** — tag `v*` triggered publish to crates.io via `scripts/publish_crates.sh` (Kahn topological order + idempotent skip + 429 backoff). `id-token: write` is set for crates.io Trusted Publishing (one-time crates.io UI binding required — see the workflow's setup header). Pre-publish smoke: `./scripts/release-local.sh` runs `cargo publish --dry-run` for every crate (packages + rewrites path deps + isolated build) before you tag.

When adding a new external dependency: add it to `[workspace.dependencies]` in the root `Cargo.toml` and reference via `{ workspace = true }` in the crate. Do NOT wire per-crate version pins — they're centralized for a reason. Run `cargo deny check` before committing; if it flags a new license or advisory, add an exception/ignore with a rationale rather than widening blindly.

## Architecture: how the pieces wire together


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Marssssss/OneAI](https://github.com/Marssssss/OneAI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
