---
trigger: always_on
description: Conventions for agents (Claude Code, Codex CLI, etc.) contributing to this repository. Humans should also follow them. For deeper context see `docs/ARCHITECTURE.md` and `docs/CONTRIBUTING.md`.
---

# Agent Guidelines

Conventions for agents (Claude Code, Codex CLI, etc.) contributing to this repository. Humans should also follow them. For deeper context see `docs/ARCHITECTURE.md` and `docs/CONTRIBUTING.md`.

## Project Overview

`gittype` is a Rust CLI typing game that turns source code from real repositories into typing challenges. It parses code with `tree-sitter`, renders a terminal UI with `ratatui` / `crossterm`, persists session history in SQLite, and is wired together via the `shaku` DI container.

- Entry points: `src/main.rs` (binary), `src/lib.rs` (library root).
- Default binary: `cargo run -- ...`.
- Rust edition: 2021.

## Architecture

The codebase follows a DDD-style three-layer split. Respect the dependency direction: `presentation` → `domain` ← `infrastructure`. Never have `domain` import from `presentation` or `infrastructure`.

```
src/
├── domain/           # Pure business logic — no I/O, no UI, no DB
│   ├── models/       # Value objects, entities (Challenge, Stage, Session, ...)
│   ├── repositories/ # Repository traits + their default implementations
│   ├── services/     # Domain services (scoring, parsing, version checks, ...)
│   ├── events/       # Domain & presentation events (EventBus)
│   ├── stores/       # In-memory shared state
│   └── error.rs      # GitTypeError + Result
├── infrastructure/   # External adapters: filesystem, SQLite, git2, HTTP, terminal
│   ├── database/     # rusqlite + DAOs + migrations
│   ├── storage/      # FileStorage / CompressedFileStorage (with test-mocks)
│   ├── git/          # git2 wrappers
│   ├── http/         # reqwest-based clients (GitHub, OSS Insight)
│   ├── logging.rs    # log4rs setup + error/panic file logging
│   └── terminal.rs   # ratatui terminal factory (real TTY only)
└── presentation/
    ├── cli/          # clap-based CLI (Cli, Commands, run_cli)
    ├── tui/          # ratatui screens, ScreenManager, transitions
    ├── ui/           # Reusable rendering primitives (colors, gradients, ...)
    ├── sharing.rs    # SharingPlatform enum + share URL building
    ├── di.rs         # AppModule (shaku) wiring all components
    └── signal_handler.rs
```

### Dependency injection

All wiring lives in `presentation/di.rs` (`AppModule`). When you add a new repository / service / screen:

1. Define a `pub trait FooInterface: shaku::Interface` in the appropriate layer.
2. Implement it as a `#[derive(shaku::Component)] pub struct FooImpl` with `#[shaku(inject)]` for its dependencies.
3. Register it in `AppModule` (`components` list).
4. Resolve via `module.resolve::<dyn FooInterface>()` or `#[shaku(inject)]`, never construct it ad-hoc.

## Build, test, lint

CI runs these exact commands — all four must pass:

```bash
cargo fmt --all -- --check
cargo clippy --all-targets --all-features -- -D warnings
cargo test
cargo llvm-cov --all-features --workspace --lcov --output-path lcov.info
```

Useful local variants:

- `cargo test <name>` — run a single test.
- `cargo run -- --help` — exercise the CLI.
- `cargo bench` — benchmarks under `benches/`.

## Test conventions

**Never write tests inside `src/`.** No inline `#[cfg(test)] mod tests { ... }` blocks in source files. All tests live under the top-level `tests/` directory, mirroring `src/`:

- `tests/unit/<layer>/<module>_tests.rs` — unit tests against `gittype::...`.
- `tests/integration/...` — multi-component / CLI-level tests, language fixtures, screen snapshots.
- `tests/helpers/`, `tests/fixtures/` — shared utilities and inputs.

When a test needs to construct or inspect something whose fields/methods are private, add a feature-gated test helper on the `src/` side rather than tests in `src/` or making the real API public:

```rust
impl Foo {
    #[cfg(feature = "test-mocks")]
    pub fn new_for_test(deps: ...) -> Self { ... }

    #[cfg(feature = "test-mocks")]
    pub fn internal_thing_for_test(&self) -> &Bar { &self.internal_thing }
}
```

The dev-dependency declared in `Cargo.toml` enables `test-mocks` for the tests crate:

```toml
[dev-dependencies]
gittype = { path = ".", default-features = false, features = ["test-mocks"] }
```

so `_for_test` helpers are visible only to tests, never to production builds.

### CI has no TTY

The Linux GitHub Actions runners have no TTY. Tests that construct a real `ratatui` terminal — directly via `CrosstermBackend::new(stdout())` or transitively through `TerminalComponent::get()` / `ScreenManagerFactory::create()` — panic with `Os { code: 11, kind: WouldBlock }`. Either:

- Guard with `if !atty::is(atty::Stream::Stdout) { return; }` (acceptable for "real-terminal" smoke tests), or
- Construct `ratatui::backend::TestBackend::new(w, h)` and wrap it in `Terminal::new(...)` directly.

Prefer `TestBackend` for anything testing rendering or screen transitions.

### Snapshots

Screen rendering uses `insta` for snapshot tests. Review and commit `.snap` files alongside the source change; never auto-accept blindly.

## Coding style

- **Place public items at the top of files**: pub structs / traits / functions first, private helpers below.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [unhappychoice/gittype](https://github.com/unhappychoice/gittype) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
