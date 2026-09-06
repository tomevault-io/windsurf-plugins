---
trigger: always_on
description: This repository is a Rust skeleton for `relay-knowledge`, a graph-database-based knowledge graph project. The root contains Cargo metadata, contributor docs, pre-commit configuration, and GitHub Actions workflow files.
---

# Repository Guidelines

## Project Structure & Module Organization

This repository is a Rust skeleton for `relay-knowledge`, a graph-database-based knowledge graph project. The root contains Cargo metadata, contributor docs, pre-commit configuration, and GitHub Actions workflow files.

Use the existing Rust layout:

- `Cargo.toml`: package manifest and Rust lint configuration.
- `src/relay_knowledge/lib.rs`: reusable knowledge graph primitives and the Cargo library entry point.
- `src/relay_knowledge/main.rs`: default CLI entry point.
- `tests/`: integration and smoke tests.
- `docs/zh/03-architecture-specs/02-engineering-hard-constraints.md`: hard constraints for shallow functions, dead code, documentation completeness, foundational modules, acyclic dependencies, max file length, unit-test coverage, event-driven HTTP, QoS, and Playwright Chromium browser integration-test readiness.
- `docs/zh/03-architecture-specs/19-installation-release-and-upgrade.md`: installation, packaging, publishing, service deployment, upgrade, and uninstall requirements.
- `.github/workflows/pr-checks.yml`: CI quality gates.

Keep generated output, build products, and large temporary data out of version control.

CodeSpec map: codespec/codespec-map.yaml
Knowledge map: knowledge/knowledge-map.yaml

## Build, Test, and Development Commands

- `cargo build`: compile the project.
- `cargo check --all-targets --all-features`: run the fast type/build gate without final code generation.
- `cargo test --all-targets --all-features`: run unit and integration tests.
- `cargo fmt --all -- --check`: verify formatting without rewriting files.
- `cargo clippy --all-targets --all-features -- -D warnings`: run lint checks and fail on warnings.
- `cargo run`: run the default binary.
- `cargo package`: verify the crate contents that would be published to crates.io.
- `cargo publish --dry-run`: validate the center-repository publishing path without publishing.
- `python3 tools/docs/check_docs.py`: validate documentation structure, local links and anchors, code fences, chapter numbering, indexes, and English-edition translation hygiene.
- `./setup.sh` or `setup.bat`: install/check the Rust toolchain, set up hooks, and run quality gates.
- `pre-commit run --all-files`: run the local quality hooks.
- `./check.sh --deep`: after installing nightly `miri` and `rust-src`, run the deterministic benchmark, FFI-free Miri core-domain tests, and AddressSanitizer in addition to the standard local gates.

Document required services, such as graph databases or local containers, in `README.md` and commit example configuration files.

## Architecture Constraints

- Build the project as event-driven and async-first from the beginning. New I/O, graph database access, indexing, ingestion, and service orchestration should expose async APIs.
- Do not add blocking work to async execution paths. If blocking CPU or filesystem work is unavoidable, isolate it behind explicit worker boundaries.
- Use bounded queues, backpressure, timeouts, and cancellation for event pipelines so ingestion or query spikes cannot grow without control.
- Keep graph storage, event transport, and domain logic separated behind small interfaces. Tests should be able to exercise domain behavior without a live database.
- Prefer observable workflows: important events should carry enough structured context for logging, tracing, retries, and debugging.
- Provide both CLI and Web usage modes. They must share the same core services and domain APIs so behavior does not diverge between interfaces.
- Provide three-layer retrieval from the start: keyword BM25, semantic retrieval, and vector retrieval. Retrieval indexes and answers must stay tied to the latest graph state, with explicit refresh, versioning, or invalidation when graph data changes.
- Treat installed background operation as a first-class runtime. Long-running graph refresh, indexing, maintenance, and diagnostics should be hosted by the platform service manager (systemd, Windows Service, or launchd) rather than an unmanaged CLI loop.
- Silent background updates must be user-configurable, observable, and reversible. They may refresh graph data and derived indexes only within authorized scopes, and must expose freshness, stale, paused, degraded, and failure states.
- Background pipelines must use bounded queues, resource budgets, backpressure, timeouts, cancellation, retry backoff, persistent cursors or leases, and dead-letter handling so spikes cannot consume unbounded CPU, memory, or disk.
- CPU-heavy or disk-heavy work such as embedding, OCR, large-file parsing, full index rebuilds, WAL checkpointing, and compaction must run behind explicit worker or maintenance boundaries and must not block query hot paths or async runtime executors.
- Design ingestion, indexing, and maintenance for crash recovery and hung-task recovery. Startup reconcilers should replay missed index refresh work, recover expired task leases, report index lag, and keep graph facts and derived indexes consistent by version.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coolplayagent/relay-knowledge](https://github.com/coolplayagent/relay-knowledge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
