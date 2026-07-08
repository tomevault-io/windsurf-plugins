---
trigger: always_on
description: Libra is a Rust implementation of an AI agent-native version control system with
---

# Libra - Repository Custom Instructions for GitHub Copilot

## What this repo is

Libra is a Rust implementation of an AI agent-native version control system with
Git on-disk compatibility, SQLite-backed repository metadata, durable AI runtime
state, tiered local/S3/R2 object storage, and a Cloudflare-backed publish flow.

This is a single Rust package named `libra` plus two TypeScript/Next.js surfaces:

- `web/`: the static Code UI exported by Next.js and embedded into the Rust binary.
- `worker/`: the Libra publish Cloudflare Worker, backed by D1 and R2.

Do not assume the older multi-crate `engine/`, `delta/`, `transport/`, or
`storage/` layout exists. The current implementation lives primarily under
`src/`, with command handlers in `src/command/` and shared/runtime internals in
`src/internal/` and `src/utils/`.

## Repository layout

- `src/main.rs`: binary entry point.
- `src/lib.rs`: embedding API (`exec`, `exec_async`) and public re-exports.
- `src/cli.rs`: clap root grammar, global output flags, repository preflight, and
  command dispatch.
- `src/command/`: one module per `libra <subcommand>`, including Git-compatible
  commands (`init`, `clone`, `add`, `commit`, `push`, `pull`, `status`, `log`,
  `show`, `diff`, `branch`, `switch`, `checkout`, `merge`, `rebase`, `stash`,
  `worktree`, etc.) and Libra-only commands (`code`, `code-control`, `agent`,
  `automation`, `usage`, `graph`, `sandbox`, `cloud`, `publish`, `db`).
- `src/internal/ai/`: AI runtime, providers, tools, MCP, session storage,
  permissions, sandboxing, context budget, goal mode, orchestration, skills, and
  web projections.
- `src/internal/tui/`: terminal UI for `libra code`.
- `src/internal/model/`: Sea-ORM models.
- `src/internal/protocol/`: Git, HTTPS, SSH, LFS, and local protocol clients.
- `src/internal/publish/`: publish snapshot/export pipeline.
- `src/utils/`: object/path/storage/output/test helpers, tiered storage,
  worktree utilities, pager support, and stable CLI error types.
- `sql/`: SQLite bootstrap schemas and migrations; `sql/publish/` is the publish
  Worker schema.
- `tests/`: integration tests. `tests/INDEX.md` is the authoritative index of
  every cargo `--test` target.
- `tests/command/`: per-command integration suites and shared command helpers.
- `tests/compat/`: cross-command compatibility guards that must also be
  registered as `[[test]]` entries in `Cargo.toml`.
- `docs/commands/`: user-facing command docs, kept in sync with the CLI surface.
- `COMPATIBILITY.md`: compatibility matrix guarded by tests.
- `.github/workflows/`: CI gates. `base.yml` is the main PR gate.

## Languages and defaults

- Rust edition: 2024.
- Primary runtime: Tokio.
- CLI parsing: clap derive in `src/cli.rs` and `src/command/*`.
- Database: SQLite through Sea-ORM.
- Serialization: serde and serde_json.
- Logging/diagnostics: tracing and user-facing `CliError`/`CliResult`.
- Frontend: Next.js 16, React 19, TypeScript, Tailwind CSS, pnpm.
- Prefer existing helpers and local patterns over new abstractions. In particular,
  use `src/utils/`, `src/internal/db.rs`, command test helpers, and AI runtime
  helper APIs where they already model the behavior being changed.

## Rust coding rules

- Run `cargo +nightly fmt --all` formatting. `rustfmt.toml` groups imports as
  standard, external, then crate imports.
- CI treats clippy warnings as failures:
  `cargo clippy --all-targets --all-features -- -D warnings`.
- Avoid wildcard imports except in tests.
- Prefer `anyhow::Result`/`anyhow::Context` for CLI flows and `thiserror` for
  domain/library errors. User-facing errors must say what failed, which resource
  was affected, and what the user can do next.
- Production code must not use `unwrap()`, `expect()`, or `panic!()` unless the
  case is obviously infallible and has a short `// INVARIANT:` comment. This
  applies to startup and initialization paths too. Tests may use them.
- When adding public enum contracts under `src/internal/ai/agent_run/`, preserve
  additive compatibility with `#[non_exhaustive]` where the existing guard expects
  it.
- Respect the repository object format. Libra supports `sha1` and `sha256` via
  `core.objectformat`; do not hard-code 20-byte object IDs.
- Keep hot paths streaming and bounded. Avoid unbounded directory walks, retries,
  buffers, allocations, or network calls in command paths unless clearly justified.
- Do not log secrets, tokens, provider keys, vault material, full authorization
  headers, or sensitive AI transcript details. Use existing redaction utilities.

## Build and test commands

Use `LIBRA_SKIP_WEB_BUILD=1` for Rust-only iteration when the embedded Code UI is
not the subject of the change.

```bash
cargo +nightly fmt --all --check
LIBRA_SKIP_WEB_BUILD=1 cargo clippy --all-targets --all-features -- -D warnings
LIBRA_SKIP_WEB_BUILD=1 cargo test --all
cargo run -- <cmd>
```

Feature-gated Rust tests:

```bash
cargo test --features test-network --test network_remotes_test
cargo test --features test-live-ai --test ai_agent_test --test ai_chat_agent_test -- --test-threads=1
cargo test --features test-live-cloud --test cloud_storage_backup_test --test publish_live_test --test storage_r2_test -- --test-threads=1

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [libra-tools/libra](https://github.com/libra-tools/libra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
