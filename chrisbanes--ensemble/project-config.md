---
trigger: always_on
description: Ensemble is a long-running Rust service that orchestrates multi-agent pipelines against an issue tracker. It reads work from trackers (GitHub Projects, todo files), creates isolated per-issue workspaces, runs named agents through a step DAG (build, review, etc.), collects strict `StepOutput` results, and drives tracker state transitions. Configuration lives in a configuration directory containing `config.yaml`.
---

# Ensemble

Ensemble is a long-running Rust service that orchestrates multi-agent pipelines against an issue tracker. It reads work from trackers (GitHub Projects, todo files), creates isolated per-issue workspaces, runs named agents through a step DAG (build, review, etc.), collects strict `StepOutput` results, and drives tracker state transitions. Configuration lives in a configuration directory containing `config.yaml`.

See `docs/SPEC.md` for the full specification, `CONTEXT.md` for canonical domain
language, and `docs/adr/` for architectural decisions.

## Project structure

```
ensemble/
├── Cargo.toml                    # workspace root
├── crates/
│   ├── ensemble-core/            # core library (domain model, config, workspace)
│   │   ├── src/
│   │   │   ├── lib.rs
│   │   │   ├── error.rs          # EnsembleError, ConfigError, WorkspaceError, WorktreeError, PipelineError
│   │   │   ├── tracker/
│   │   │   │   ├── mod.rs        # IssueTracker trait (read + write), TrackerError
│   │   │   │   └── model.rs      # Issue, RunningEntry, RetryEntry, AgentTotals
│   │   │   ├── config/
│   │   │   │   ├── ensemble.rs   # config.yaml loader (EnsembleConfig)
│   │   │   │   ├── location.rs   # config directory resolution
│   │   │   │   └── template.rs   # Liquid prompt template renderer
│   │   │   ├── pipeline/
│   │   │   │   ├── mod.rs        # re-exports
│   │   │   │   ├── dag.rs        # DAG construction + validation
│   │   │   │   ├── engine.rs     # PipelineRun per-issue execution
│   │   │   │   └── verdict.rs    # StepOutput parsing and validation
│   │   │   └── workspace/
│   │   │       ├── manager.rs    # WorkspaceManager (create/reuse/cleanup directories + worktrees)
│   │   │       ├── coordinator.rs # WorktreeCoordinator (multi-repo worktree lifecycle)
│   │   │       ├── worktree.rs   # Core git worktree operations (create/remove/exists/pull)
│   │   │       ├── push_strategy.rs # PushStrategy enum (ask/auto_push/manual/pr_only)
│   │   │       └── hooks.rs      # Async hook runner with timeouts
│   │   └── tests/
│   │       └── workflow_to_workspace.rs  # integration test
│   ├── ensemble-cli/             # CLI binary
│   │   ├── build.rs              # optional SPA build + embed script (`web-ui` feature)
│   │   ├── src/
│   │   │   ├── main.rs           # CLI entry point, subcommand dispatch
│   │   │   ├── embedded_ui.rs    # rust-embed SPA serving (`web-ui` feature)
│   │   │   └── commands/
│   │   │       ├── mod.rs        # re-exports
│   │   │       ├── init.rs       # `ensemble init` interactive config wizard
│   │   │       ├── run.rs        # `ensemble run` headless orchestrator
│   │   │       └── web.rs        # `ensemble web` orchestrator + SPA + API (`web-ui` feature)
│   │   └── tests/
│   └── ensemble-desktop/         # Tauri desktop app
│       ├── build.rs              # tauri-build + SPA embed script
│       └── src/
│           ├── main.rs           # Tauri entry point, runtime management
│           ├── embedded_ui.rs    # rust-embed SPA serving for Tauri
│           └── server.rs         # Desktop HTTP server using the shared ensemble-core bootstrap
└── .github/workflows/ci.yml     # CI: check, test, clippy, fmt
```

Future crates (not yet implemented): `ensemble-agent`, `ensemble-server`.

## Build and test

```sh
cargo build --workspace
cargo test --workspace
cargo clippy --workspace -- -D warnings
cargo fmt --all -- --check
```

Default `ensemble-cli` builds are headless. Compile the web dashboard command with
`--features web-ui`; for Rust-only checks of that feature, use `SKIP_UI_BUILD=1`.

## Pre-push checklist

Before pushing commits, ensure all checks pass locally:

```sh
# Rust code
cargo test --workspace --exclude ensemble-desktop
SKIP_UI_BUILD=1 cargo test -p ensemble-cli --features web-ui --test product_e2e -- --nocapture
SKIP_UI_BUILD=1 cargo check -p ensemble-cli --features web-ui
cargo clippy --workspace --exclude ensemble-desktop -- -D warnings
cargo fmt --all -- --check

# Frontend code (if you modified UI files)
cd crates/ensemble-ui/src-ui
pnpm test
pnpm run build
```

CI will run these checks on your PR; failures block merge.

## CI

GitHub Actions runs on push to `main` and all PRs. A dedicated MSRV job uses Rust 1.95.0 to
check all workspace targets. Main, normal, frontend, and desktop jobs use the pinned Rust 1.97.0
toolchain from `rust-toolchain.toml`. The main CI job runs format, clippy, default non-desktop
Rust tests, the feature-enabled product E2E test, and a CLI `web-ui` feature check. Frontend and
desktop jobs run separately. All must pass. Primary jobs use `RUSTFLAGS=-Dwarnings` — treat
warnings as errors; the MSRV job is a compile-compatibility check.

## Release

**One-time setup:**
```sh
cargo install cargo-release
```

**Cutting a release:**
```sh
cargo release <version> --execute   # e.g. cargo release 0.2.0 --execute
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chrisbanes/ensemble](https://github.com/chrisbanes/ensemble) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
