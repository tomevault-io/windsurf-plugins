---
trigger: always_on
description: - Rust workspace crates live under `crates/`:
---

# Repository Guidelines

## Workspace Layout
- Rust workspace crates live under `crates/`:
  - `crates/etl/`: core replication library.
  - `crates/etl-api/`: HTTP API service.
  - `crates/etl-replicator/`: standalone replicator binary.
  - `crates/etl-postgres/`: Postgres integration.
  - `crates/etl-destinations/`: destination implementations.
  - `crates/etl-config/`: configuration types and loading.
  - `crates/etl-telemetry/`: tracing and Prometheus setup.
  - `crates/etl-examples/`: examples.
  - `crates/etl-benchmarks/`: benchmarks.
  - `crates/xtask/`: workspace automation commands.
- Docs live in `docs/`.
- Local development and ops tooling live in `crates/xtask/` (run via `cargo x`) and `DEVELOPMENT.md`.
- Tests live next to code in `src/` or `tests/`.

## Commands
- Build everything:
  - `cargo build --workspace --all-targets --all-features`
- Format:
  - `cargo x fmt`
- Check formatting:
  - `cargo x fmt --check`
- Lint:
  - `cargo clippy --workspace --all-targets --all-features -- -D warnings`
- Run unit tests (no Postgres required):
  - `cargo nextest run --workspace --all-features --lib`
- Run tests for one crate:
  - `cargo nextest run -p etl-config --all-features`
- Run doctests (nextest does not support doctests):
  - `cargo test --doc --workspace --all-features`
- List tests:
  - `cargo nextest list --workspace --all-features`
- Run full test suite (requires Postgres clusters via `cargo xtask postgres start`):
  - `cargo xtask nextest run`

## Agent Workflow
- Keep changes focused on the issue being solved.
- Prefer small diffs unless a broader refactor is clearly justified.
- Before adding new patterns, inspect nearby code and follow the local style first.
- Do not add dependencies unless they are justified by the task.
- If you change workflow assumptions, build or test the smallest relevant target and report what actually ran.
- Never create commits, push branches, open pull requests, or perform other git write actions unless the user explicitly instructs you to do so.
- Never modify migration files unless the user explicitly asks for a migration
  change. This includes changing comments or other non-executable text inside
  migration files.
- Keep the workspace on the stable toolchain from `rust-toolchain.toml` for build, lint, and test commands; use the pinned nightly formatter only through `cargo x fmt` and `cargo x fmt --check`.
- Treat `Cargo.toml` workspace lints, `rustfmt.toml`, and compiler diagnostics as the source of truth for enforceable style and correctness rules. Prefer adding or tightening static checks over adding prose rules here.
- Run Clippy, builds, and tests intentionally when they are relevant: for example
  after changing Rust code, when compiler/lint diagnostics indicate a problem,
  when workflow assumptions changed, or when the user asks for verification. Do
  not run expensive checks reflexively for unrelated documentation, YAML-only, or
  similarly low-risk edits; in those cases, run the smallest relevant validation
  instead and report what actually ran.
- Add local destination value validation only when it prevents silent data
  corruption, such as rounding, truncation, clamping, coercion, or another
  semantic change that the destination would accept. If the destination rejects
  an unsupported value with an error, prefer delegating that check to the
  destination instead of duplicating expensive validation in the write path.

## Public Repo Secret Safety
- Treat this repository, every branch, every commit, every PR, and every review
  comment as public by default. Assume anything written to tracked files, commit
  metadata, terminal output quoted in a PR, screenshots, logs, comments, and
  generated artifacts can become permanently visible.
- Never include real secrets, credentials, tokens, API keys, service keys,
  passwords, private URLs, private hostnames, customer data, personal data, raw
  production payloads, internal incident details, or proprietary configuration
  values in code, tests, docs, comments, examples, commit messages, branch
  names, PR titles, PR descriptions, review comments, issue comments, logs, or
  generated files.
- Before writing or modifying files, scan the relevant context for suspicious
  values. Before any git write action that the user explicitly requested, review
  the staged diff, commit message, branch name, PR title, PR body, and any
  comments for possible sensitive information.
- If a value looks real, came from the user's environment, appeared in local
  config, came from command output, or was copied from any non-public source, do
  not commit it or repeat it in comments or PR text. Stop and ask the user
  before proceeding, explaining the specific risk without quoting the sensitive
  value back.
- Use invented, clearly fake placeholders for examples and tests, such as
  `example.com`, `127.0.0.1`, `placeholder-token`, `fake-api-key`, or
  documented test credentials already present in the repository. Prefer
  redacted forms such as `<redacted>` when discussing an existing sensitive
  value.
- Keep secrets out of source control entirely. Use environment variables,
  ignored local files, secret managers, CI secret storage, or documented local
  setup steps instead of tracked files.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [supabase/etl](https://github.com/supabase/etl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
