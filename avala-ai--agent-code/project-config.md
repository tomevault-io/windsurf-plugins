---
trigger: always_on
description: Instructions for AI coding agents working in this repository. Read this file in full before making changes. It captures the non-obvious rules — conventions you cannot derive from just grepping the code.
---

# AGENTS.md

Instructions for AI coding agents working in this repository. Read this file in full before making changes. It captures the non-obvious rules — conventions you cannot derive from just grepping the code.

This repo is **agent-code** (Avala AI): a Rust terminal coding agent, shipped as both a CLI and an embeddable library. Because this project *is itself* an agent, changes here affect the safety posture of every user who runs it. Move carefully.

---

## 1. Repo layout

```
crates/
  lib/        # agent-code-lib — LLM providers, tools, query loop, memory,
              #   permissions, MCP client, skills, compaction. The engine.
  cli/        # agent-code    — binary: REPL, TUI, slash commands, streaming output.
  eval/       # agent-code-eval — behavioral evaluation harness (not published).
client/       # Flutter desktop/web client that talks to `agent --serve`.
packages/     # TypeScript client library (`agent_code_client`).
npm/          # npm installer wrapper.
docs/         # Mintlify + mdBook docs (architecture, guides, reference).
evals/        # Eval fixtures and scenarios consumed by `crates/eval`.
scripts/      # E2E shell harnesses.
.github/workflows/  # CI — `ci.yml` is the canonical gate.
```

Supporting docs you should read before a non-trivial change:
`README.md`, `ARCHITECTURE.md`, `SECURITY.md`, `CONTRIBUTING.md`, `ROADMAP.md`, `RELEASING.md`, `CHANGELOG.md`.

---

## 2. Build, test, lint — the CI gate

Every PR must pass `ci.yml`. Run these locally before pushing; they are the exact commands CI runs.

```bash
cargo check   --all-targets                   # compile
cargo test    --all-targets                   # unit + integration + doc tests
cargo clippy  --all-targets -- -D warnings    # lint — warnings are errors
cargo fmt     --all -- --check                # format gate
```

Release builds (matrix of targets) use:

```bash
cargo build --release --target <triple>
```

Benchmarks live in `crates/lib/benches/`:

```bash
cargo bench --bench compaction
cargo bench --bench token_estimation
cargo bench --bench startup
```

### Running a single test

```bash
cargo test -p agent-code-lib <module>::<test_name>
cargo test -p agent-code     <test_name>          # CLI crate
```

### What tests need

- Unit and integration tests in `crates/lib/tests/` and `crates/cli/tests/` are **hermetic** — no network, no API keys. Keep them that way.
- Eval runs in `crates/eval/` and `evals-nightly.yml` may require `ANTHROPIC_API_KEY`. Do not add key requirements to the default `cargo test` path.
- If you add a test that needs network, gate it with `#[ignore]` and document how to run it.

### Format and lint config

There is **no `rustfmt.toml` and no `clippy.toml`** — defaults only. Do not add one without discussion; style changes cascade through the whole tree. The `-D warnings` flag means any new clippy lint must be fixed, not suppressed. Prefer fixing the code; use `#[allow(...)]` only with a one-line comment explaining why.

---

## 3. Security rules — non-negotiable

This project's security posture is load-bearing. Breaking any of the following is a hard blocker for merge.

### Protected directories

`crates/lib/src/permissions/mod.rs` defines `PROTECTED_DIRS`:

```rust
const PROTECTED_DIRS: &[&str] = &[
    ".git/", ".git\\",
    ".husky/", ".husky\\",
    "node_modules/", "node_modules\\",
];
```

Write tools (`FileWrite`, `FileEdit`, `MultiEdit`, `NotebookEdit`) are **unconditionally** blocked from these paths — the check runs before any permission rule. If you extend the write-tool set, add the new tool to `is_write_tool()` in the same file. If you add a new protected path, add both forward- and back-slash variants.

### Permission system invariants

- Every tool call goes through the permission check. No exceptions. Do not add a "trusted" bypass.
- Read-only tools skip the ask prompt; write/exec tools do not. When adding a new tool, classify it correctly.
- Plan mode must remain read-only. If your change adds a tool, explicitly decide whether plan mode can call it and test that branch.
- The `--dangerously-skip-permissions` flag exists for automation use cases. It can be globally disabled via `disable_bypass_permissions = true` in settings. **Never weaken or remove that setting's effect.**

### Bash tool destructive-command detection

The bash tool warns on patterns like `rm -rf`, `git reset --hard`, `DROP TABLE`, etc. and blocks writes to system paths (`/etc`, `/usr`, `/bin`, `/sbin`, `/boot`, `/sys`, `/proc`). Do not narrow these checks. If you add a new one, add a test in the same file.

### Skills

Skills can embed shell blocks. The `disable_skill_shell_execution` setting strips them. Any new skill-loading code path must respect that setting.

### Secrets and telemetry

- **Never** write API keys to config files. Env vars only. The config schema should not grow a `*_api_key` field.
- **Never** add telemetry that leaves the machine without explicit opt-in. The only outbound network calls allowed by default are to the user-configured LLM provider and MCP servers.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [avala-ai/agent-code](https://github.com/avala-ai/agent-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
