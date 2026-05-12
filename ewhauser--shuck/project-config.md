---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is this project?

Shuck is a shell script linter/checker CLI tool, built on top of **shuck-parser** (an in-process virtual bash interpreter written in Rust). The repo is a Cargo workspace containing both shuck (the linter) and shuck-parser (the underlying library).

## Clean-Room Policy

This project is a clean-room reimplementation of ShellCheck. To preserve the integrity of the independent authoring process, the following rules **must** be followed at all times.

### Prohibited Inputs

- **Do not** read, reference, or import ShellCheck source code.
- **Do not** read, reference, or import ShellCheck wiki pages or documentation examples.
- **Do not** reuse diagnostic wording from ShellCheck materials.
- **Do not** copy raw ShellCheck output into committed repository files.
- **Do not** search the web for ShellCheck source, wiki content, or diagnostic text.

### Approved Inputs

- Shell language manuals, specifications (POSIX, Bash reference manual), and semantic notes.
- Files already authored inside this repository.
- The ShellCheck binary used only as a **black-box oracle** (run it, observe numeric exit/code, but do not copy its output text into committed files). You can reverse engineer behavior by running code through the ShellCheck binary.
- The companion `shell-checks` repository (`../shell-checks`) which contains independently authored rule specs, examples, and compatibility mappings.

### Authoring Rules

- Write all summaries, rationales, comments, and diagnostics from scratch in your own words.
- Compatibility codes may be referenced as bare numbers or `SC1234`-style identifiers.
- Do not copy ShellCheck text or third-party snippets verbatim into committed code or comments.

## Build and test commands

```bash
# Build just the shuck crates (fast iteration)
make build                    # cargo build -p shuck-cli -p shuck-cache

# Test just the shuck crates
make test                     # cargo test -p shuck-cli -p shuck-cache

# Run the shuck CLI
make run ARGS="check ."       # cargo run -p shuck-cli -- check .

# Build/test everything (including shuck-parser)
cargo build
cargo test --features http_client

# Run a single test
cargo test -p shuck-cli -- test_name
cargo test -p shuck-linter -- test_name
cargo test -p shuck-parser -- test_name

# Format and lint
cargo fmt
cargo clippy --all-targets -- -D warnings
```

## Large corpus tests

Always run the large corpus comparisons with the nix-provided `shellcheck`, not a globally installed one. The supported path is `make test-large-corpus`, which enters the repo's nix dev environment before running the ignored large-corpus test.

```bash
# Download/populate the corpus if needed
make setup-large-corpus

# Run the full compatibility comparison
make test-large-corpus

# Run a targeted comparison for one rule or a CSV list of selectors
make test-large-corpus SHUCK_LARGE_CORPUS_RULES=C001
make test-large-corpus SHUCK_LARGE_CORPUS_RULES=C001,C006

# Run a smaller sample while iterating
make test-large-corpus SHUCK_LARGE_CORPUS_SAMPLE_PERCENT=10

# Print only the 25 slowest Shuck fixtures and always exit successfully
make test-large-corpus SHUCK_LARGE_CORPUS_TIMING=1
```

Relevant environment variables for the large-corpus harness:

- `SHUCK_TEST_LARGE_CORPUS=1` — enables the ignored large-corpus test. `make test-large-corpus` sets this for you.
- `SHUCK_LARGE_CORPUS_ROOT=/path/to/corpus` — overrides corpus discovery. By default the test looks under `.cache/large-corpus` and then `../shell-checks`.
- `SHUCK_LARGE_CORPUS_TIMEOUT_SECS=300` — ShellCheck timeout budget per fixture.
- `SHUCK_LARGE_CORPUS_SHUCK_TIMEOUT_SECS=30` — Shuck timeout budget per fixture.
- `TEST_SHARD_INDEX=0` and `TEST_TOTAL_SHARDS=1` — split the corpus run across shards.
- `SHUCK_LARGE_CORPUS_RULES=C001,C006` — comma-separated rule selectors to compare. Accepts exact rule codes and the existing selector syntax such as category or prefix selectors.
- `SHUCK_LARGE_CORPUS_SAMPLE_PERCENT=100` — deterministic fixture sampling percentage in `[1,100]`.
- `SHUCK_LARGE_CORPUS_MAPPED_ONLY=1` — limits ShellCheck diagnostics to codes that Shuck maps today.
- `SHUCK_LARGE_CORPUS_KEEP_GOING=1` — collects all fixture failures instead of stopping at the first one.
- `SHUCK_LARGE_CORPUS_TIMING=1` — runs a Shuck-only timing pass, prints the 25 slowest fixtures, and always exits successfully. This mode does not produce a compatibility log, so `make large-corpus-report` rejects it.

If you need to call `shellcheck` directly while debugging a large-corpus issue, do it through nix so the version matches the test harness. For example:

```bash
nix --extra-experimental-features 'nix-command flakes' develop --command shellcheck --version
```

## Architecture

### Workspace crates

- **`crates/shuck-cli`** — CLI binary. Discovers files, resolves config, coordinates caching, parses shell sources, runs lint/format commands, applies fixes, and renders reports. Project roots are resolved by walking up to find `.shuck.toml` or `shuck.toml`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ewhauser/shuck](https://github.com/ewhauser/shuck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
