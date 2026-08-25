---
trigger: always_on
description: All lints are configured in `Cargo.toml` under `[workspace.lints]`. The key rules:
---

# CLAUDE.md — Project Rules for AI Agents

## Lint Policy (STRICTLY ENFORCED — compiler will reject violations)

All lints are configured in `Cargo.toml` under `[workspace.lints]`. The key rules:

1. **NEVER add `#[allow(...)]`, `#[expect(...)]`, or `#![allow(...)]` attributes.** The `allow_attributes` lint is set to `warn` — the CI `-D warnings` flag treats this as an error, so hand-written lint suppressions are rejected in practice. Derive macros (serde, thiserror) are permitted to emit internal `#[allow]`.

2. **NEVER use `.unwrap()`, `.expect()`, `panic!()`, `todo!()`** — these are all `deny` (`unwrap_used`, `expect_used`, `panic`, `todo`). Use proper error handling with `Result`/`Option` combinators, `?` operator, or `if let`/`match`. Also avoid `unimplemented!()` and `unreachable!()` — they are not explicitly denied but expand to panics and should not appear in production code.

3. **NEVER use `println!()`, `eprintln!()`, `print!()`, `eprint!()`** — these are `deny`. Use `std::io::Write` with `write!()`/`writeln!()` for output, or a logging framework.

4. **NEVER use `dbg!()`** — `deny`. Remove all debug macros before committing.

5. **NEVER use `unsafe`** — `forbid`. No unsafe code anywhere.

6. **NEVER use `std::process::exit()`** — `deny`. Return from main instead.

7. **NEVER use `.unwrap()` inside functions returning `Result`** — `deny` via `unwrap_in_result`.

8. **Use `.get()` instead of `[]` indexing** where possible — `indexing_slicing` is `warn`.

9. **Use `create_dir_all` instead of `create_dir`** — `create_dir` is `warn`.

## If a lint blocks you

Do NOT suppress it. Instead:
- **Fix the code** to satisfy the lint
- If a lint is genuinely wrong for a specific case, raise it — the `Cargo.toml` config is the single source of truth, not inline attributes

## Build Commands

```bash
cargo build --workspace
cargo test --workspace
cargo clippy --workspace -- -D warnings
cargo fmt --check
```

All four must pass with zero warnings before any commit.

## Coverage Commands (MANDATORY before pushing)

89% branch coverage is required. Prereqs: `rustup toolchain install nightly && rustup component add llvm-tools-preview --toolchain nightly && cargo install cargo-llvm-cov`

```bash
cargo +nightly llvm-cov clean --workspace
cargo +nightly llvm-cov --no-report --workspace --branch
cargo +nightly llvm-cov --no-report --doc --branch
cargo +nightly llvm-cov report --doctests --branch \
  --ignore-filename-regex '(tests/|research/|specs/|wizard_tty\.rs|lsp\.rs|libaipm-engine-spec/build\.rs|libaipm-engine-spec/src/bin)'
```

Verify the TOTAL branch column shows ≥ 89%. For HTML or lcov output, append `--html --open` or `--lcov --output-path lcov.info` to the report command.

This command must stay in sync with the **Coverage (89% branch gate)** job in `.github/workflows/ci.yml`, which runs the same steps and reads the ignore pattern from its `COVERAGE_IGNORE_REGEX` env var (annotated there with "Must match the coverage command documented in CLAUDE.md exactly"). If you change the regex or the flags here, change `ci.yml` in the same commit — and vice versa.

## Copilot Coding Agent Setup

The file `.github/workflows/copilot-setup-steps.yml` defines the pre-build environment for the [GitHub Copilot coding agent](https://docs.github.com/en/copilot/using-github-copilot/using-claude-sonnet-in-github-copilot). It runs before every agent task and installs all toolchain prerequisites so the sandbox does not need network access during the actual build:

| Step | Purpose |
|---|---|
| `dtolnay/rust-toolchain@stable` | Installs `clippy` and `rustfmt` |
| `dtolnay/rust-toolchain@nightly` | Installs `llvm-tools-preview` for coverage |
| `apt-get install libgit2-dev libssl-dev pkg-config` | Native system libraries required by `git2` and OpenSSL crates |
| `Swatinem/rust-cache@v2` | Caches the Cargo registry and build artefacts across runs |
| `cargo fetch --locked` | **Pre-fetches all Cargo dependencies** so they are available in the offline sandbox environment |

> **Why `cargo fetch --locked`?** The Copilot agent sandbox has restricted network access after the setup phase. Without this step the build fails because crates cannot be downloaded during `cargo build`. Pre-fetching under `--locked` also guarantees the exact dependency graph recorded in `Cargo.lock` is used — no accidental updates. This step was added to fix the [#700](https://github.com/TheLarkInn/aipm/pull/700) sandbox build failures.

Do **not** remove or weaken `CARGO_NET_RETRY` (currently `10`) or the `--locked` flag — both are necessary for reliability in flaky network conditions.

## Agentic Workflows

The repository uses [GitHub Agentic Workflows](https://githubnext.com/projects/agentics) (`.github/workflows/*.md` compiled via `gh aw compile`) for automated maintenance tasks. One exception: `research-codebase.yml` is intentionally hand-written — see the note below the table.

| Workflow file | Timeout | Schedule | Purpose |
|---|---|---|---|
| `improve-coverage.md` | 45 min | Every 15 min | Finds uncovered branches, writes tests, opens PRs; guards open coverage PRs against conflicts/staleness and escalates repeated no-ops |
| `daily-qa.md` | 45 min | Every 3 h | Validates build, tests, and documentation health |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheLarkInn/aipm](https://github.com/TheLarkInn/aipm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
