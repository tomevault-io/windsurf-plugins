---
trigger: always_on
description: This file defines repository-wide instructions for coding agents working in this project.
---

# AGENTS.md — SAF Static Analyzer Factory

This file defines repository-wide instructions for coding agents working in this project.

## Project Overview

SAF is a Rust + Python (PyO3) static analysis framework that provides:
- Whole-program pointer analysis and value-flow reasoning
- A schema-driven Python API for AI agents to author new analyzers
- A frontend-agnostic core operating on AIR (Analysis Intermediate Representation)

Full requirements: `docs/static_analyzer_factory_srs.md`

## Development Commands (Docker-only)

All builds and tests run inside Docker (Ubuntu 24.04 LTS):

```bash
make help        # Show all available commands
make test        # Run all tests (Rust + Python) in Docker
make lint        # Run clippy + rustfmt check in Docker (runs both targets below)
make lint-clippy # Run clippy only in Docker
make lint-fmt    # Run rustfmt check only in Docker
make fmt         # Auto-format all Rust code in Docker
make shell       # Open interactive dev shell in Docker
make build       # Build minimal runtime Docker image
make clean       # Remove Docker volumes and local target/
```

**IMPORTANT: Never run `cargo test` or `cargo build` locally** for crates that depend on LLVM (`saf-frontends`, `saf-analysis`, `saf-python`, `saf-cli`). LLVM 18 is only available inside the Docker container. Running `cargo test -p saf-analysis` locally will fail with "No suitable version of LLVM was found." Always use `make test` or `make shell` to run tests inside Docker. The only crate safe to build/test locally is `saf-core` (no LLVM dependency).

**IMPORTANT: Subagents must NEVER call `make` commands.** All `make` commands (`make test`, `make shell`, `make lint`, etc.) must be called by the main agent only. Subagents should focus on code reading, analysis, and editing — the main agent coordinates Docker/build operations.

**IMPORTANT: Always `make fmt` before `make lint`.** `make lint` includes a formatting check (`lint-fmt`) that will fail if code isn't formatted. Run `make fmt && make lint` in one command to avoid the wasted round-trip of lint-fail → fmt → lint-again.

**IMPORTANT: `make shell CMD='...'` is unreliable for passing commands.** The CMD variable may not be forwarded through Docker correctly, producing empty output or silently dropping the command. Always use `docker compose run --rm dev sh -c '...'` directly when you need to run a specific command inside Docker. Reserve `make shell` for interactive use only.

**IMPORTANT: PTABen benchmarks take 30-120s.** Always run them in the background (`run_in_background: true`) and check output later, rather than blocking and retrying when the timeout hits.

**IMPORTANT: Never re-run expensive Docker commands just to grep differently.** `make test`, `make lint`, etc. take minutes. If you need to inspect the output, capture it once (e.g., `make test 2>&1 | tee /tmp/test-output.txt`) and search the file afterward. Do not re-run the command with a different `grep` pattern.

**IMPORTANT: SAF uses `cargo-nextest`, not `cargo test`.** The test summary format is `Summary [...] N tests run: N passed, N skipped` — not the standard `test result: ok. N passed` from `cargo test`. Grep for `Summary` or `passed`, not `test result`.

## Architecture

### Crate Layout

```
crates/
  saf-core/       # AIR, config, IDs, deterministic utils (no internal deps)
  saf-frontends/  # Frontend trait + LLVM/AIR-JSON implementations (depends on saf-core)
  saf-analysis/   # CFG, callgraph, PTA, valueflow (depends on saf-core)
  saf-cli/        # CLI binary (depends on saf-core, saf-frontends, saf-analysis)
  saf-python/     # PyO3 bindings (depends on saf-core, saf-frontends, saf-analysis)
```

### Data Flow

```
Input (.ll / .bc / .air.json)
  → Frontend (implements api::Frontend trait)
    → AirBundle (canonical IR)
      → Graph builders (CFG, ICFG, CallGraph, DefUse)
        → PTA solver (Andersen CI)
          → ValueFlow graph
            → Queries (flows, taint_flow, points_to)
              → Export (JSON, SARIF)
```

### Key Types

- `saf_core::id` — BLAKE3-based deterministic ID generation (`make_id`, `id_to_hex`)
- `saf_core::config::Config` — Full configuration contract (SRS Section 6)
- `saf_core::air::AirBundle` — Canonical IR bundle from frontend ingestion
- `saf_frontends::api::Frontend` — Trait all frontends implement
- `saf_core::error::CoreError` — Error types using `thiserror`

## Coding Conventions

### Rust
- Use `thiserror` for library error types, `anyhow` only in binaries
- Use `BTreeMap`/`BTreeSet` (not `HashMap`/`HashSet`) for deterministic iteration (NFR-DET)
- No `.unwrap()` in library code — return `Result` or use `expect()` with a message
- All public items should have doc comments
- Pedantic clippy lints enabled (see `.cargo/config.toml`)
- **Clippy `doc_markdown`**: In doc comments, wrap type names, function names, and code identifiers in backticks (e.g., `` `ValueId` ``, `` `MemPhi` ``, `` `GEP` ``, `` `LiveOnEntry` ``). Without backticks, clippy treats CamelCase/PascalCase words as missing markdown links.
- **Clippy `iter_over_hash_type` / map iteration**: Use `.values()` or `.keys()` instead of destructuring `for (_, val) in map`. Clippy flags `for (_, v) in btreemap` as "you seem to want to iterate on values".

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Static-Analyzer-Factory/static-analyzer-factory](https://github.com/Static-Analyzer-Factory/static-analyzer-factory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
