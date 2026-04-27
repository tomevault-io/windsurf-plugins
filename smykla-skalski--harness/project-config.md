---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and test commands

```bash
mise run check                 # type-check without building
mise run test                  # unit + integration (fast, single-threaded)
mise run test:unit             # unit tests only
mise run test:slow             # slow tests (#[ignore])
mise run lint:fix              # format + clippy fixes
mise run install               # build release binary, install to ~/.local/bin
cargo test --lib cli::tests    # all tests in a module
cargo test --lib errors::tests::cli_err_basic_fields -- --exact  # single test
cargo fmt --check              # check formatting
cargo clippy --lib             # lint check only
```

Unit tests are in-crate `#[test]` blocks. Integration tests live in `tests/integration/` and cover hooks, commands, and workflows end-to-end. Integration tests run single-threaded (`--test-threads=1`) for env safety. Tests that read XDG paths must isolate state with `temp_env::with_vars` setting both `XDG_DATA_HOME` and `CLAUDE_SESSION_ID`. No mocks - tests use real filesystem state. Slow tests are marked `#[ignore]` and run via `mise run test:slow`.

Pre-commit: `cargo fmt --check && cargo clippy --lib && mise run test`

For the Harness Monitor macOS app (`apps/harness-monitor-macos`), see that directory's own `CLAUDE.md` - it covers XcodeGen, xcodebuild validation lanes, SwiftUI/UX rules, performance measurement, and daemon modes.

## Architecture

Harness is a test orchestration framework for Kubernetes/Kuma. It enforces tracked, user-story-first testing through state machines and hook-based guardrails.

### Two parallel workflow systems (`src/workflow/`)

**suite:run** (`workflow/runner.rs`): orchestrates test runs through phases `bootstrap` -> `ready` -> `approved` -> `running` -> `verdict`. State persisted as versioned JSON via `VersionedJsonRepository` (atomic tmp-file -> rename saves).

**suite:create** (`workflow/create.rs`): manages interactive suite creation with multi-step proposals and manifest validation.

### Hook system (`src/hooks/`, `src/cli.rs`)

Hooks intercept Claude Code tool usage. Classified in `cli.rs` as constants:

- **Pre-tool-use guards**: `guard-bash` (blocks direct cluster binary access), `guard-write` (blocks writes outside run surface), `guard-question`
- **Post-tool-use verifies**: `verify-bash`, `verify-write`, `verify-question`, `audit`
- **Blocking**: `guard-stop` (prevents session end if run incomplete)
- **Subagent gates**: `context-agent` (start), `validate-agent` (stop)
- **Failure enrichment**: `enrich-failure`

### Key modules

- `errors.rs` - unified error/hook message system with `{placeholder}` template substitution (fallback to `?`)
- `schema.rs` - custom frontmatter parser for suite/run YAML metadata
- `context.rs` - run lifecycle types: `RunLayout` (directory structure), `RunMetadata`, `CommandEnv`
- `prepared_suite.rs` - suite artifact types (manifests, groups, digests)
- `compact.rs` - file fingerprinting (SHA256 + mtime) for change tracking
- `core_defs.rs` - build info, timestamps, XDG paths, session scope (SHA256-hashed)
- `rules.rs` - declarative denied-binary lists, make targets, etc.
- `commands/` - 33 command handlers dispatched from CLI
- `session/` - multi-agent orchestration: `types.rs` (SessionState, AgentRegistration, WorkItem, SessionRole), `roles.rs` (permission matrix), `storage.rs` (VersionedJsonRepository + JSONL audit log), `service.rs` (12 orchestration functions), `transport.rs` (13 CLI commands), `observe.rs` (cross-agent observation with periodic sweep)
- `agents/runtime/` - AgentRuntime trait with 5 implementations (claude, codex, gemini, copilot, opencode), ConversationEvent types, signal protocol (write/read/acknowledge), liveness detection

### Data directories (XDG)

- `$XDG_DATA_HOME/harness/suites/` - suite library
- `$XDG_DATA_HOME/harness/runs/` - run directories (`{run_id}/{artifacts,commands,state,manifests,reports}`)
- `$XDG_DATA_HOME/harness/contexts/{session-hash}/` - session context
- `$XDG_DATA_HOME/harness/projects/project-{digest}/orchestration/` - multi-agent session state
- `$XDG_DATA_HOME/harness/projects/project-{digest}/agents/signals/` - file-based agent signaling

## Code conventions

- Rust 2024 edition, requires rustc 1.94+
- Clippy pedantic is set to `deny` - all new code must pass pedantic lints
- Errors use `CliErrorKind` enum variants with typed fields via thiserror
- Hook messages use `HookMessage` enum with `into_result()` conversion
- Commits: `{type}({scope}): {message}` — types: `feat`, `fix`, `refactor`, `chore`, `docs`, `test`, `perf`

## Versioning

Every feature change must evaluate semver and bump the version in the same change. Do not ship feature work without updating the version surfaces that track the release.

- `major` - any breaking change to CLI commands or flags, hook payload contracts, persisted state/schema/artifact formats, machine-consumed output, or behavior that user scripts or suites can reasonably rely on
- `minor` - backward-compatible new functionality such as a new command, flag, output field, hook capability, report surface, or materially expanded behavior

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/smykla-skalski) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-13 -->
