---
trigger: always_on
description: Orientation for AI coding agents working on this repo. The human-written map
---

# AGENTS.md

Orientation for AI coding agents working on this repo. The human-written map
is [docs/development/HANDOFF.md](docs/development/HANDOFF.md). Read it before any non-trivial
change; this page is the condensed agent view plus the gates that are easy
to trip.

Cyclops coordinates terminal coding agents already running in tmux: a Rust
daemon (`cyclopsd`) watches panes, fuses sensors into an agent state,
accepts messages into durable mailboxes, writes one doorbell line per
message with a recorded receipt, and appends every fact to an NDJSON
journal.

## Where things live

| Path | What | The rule that matters |
|---|---|---|
| `src/cyclops-proto` | Wire + journal types, the notification state machine, attention rule | Shared *rules* live here once; no IO, no tmux, no rendering |
| `src/cyclops-tmux` | The tmux adapter | **Every tmux invocation in the product is in this crate** (one exception: the daemon's boot-time `tmux -V`) |
| `src/cyclops-manifest` | Detection-manifest schema and evaluation | Vendor CLI behavior is TOML data in `resources/manifests/`, never Rust |
| `src/cyclops-ledger` | Append-only NDJSON writer/reader | Never rewritten; corrections are new lines |
| `src/cyclops-theme` | Semantic color tokens | Renderers use tokens, never raw colors |
| `src/cyclops-client` | Shared blocking and async daemon transport | Owns greeting, framing, correlation, timeout, and uncertainty; no presentation or domain policy |
| `src/cyclopsd` | The daemon: fusion, delivery, socket, identity | Library + thin binary so tests boot it in-process |
| `src/cyclops` | The CLI | Thin client; business rules stay in proto/daemon. User-facing sentences live in `src/cyclops/src/copy.rs` |
| `src/cyclops-ui` | The stream TUI (`cyclops watch`) | Its `grid` module is the CLI/stream rendering vocabulary |
| `src/cyclops-workspace` | The full-screen workspace (`cyclops`) | Ratatui/Crossterm chrome; pane VT runtimes; shares `cyclops-theme` tokens |
| `tests/testrig` | Test-only isolated tmux server | The only way tests may touch tmux |
| `resources/manifests/`, `resources/themes/`, `resources/layouts/`, `resources/hooks/` | Data, not code paths | `resources/manifests/` and `resources/layouts/` are compiled into the CLI with include_str and seeded to the user home on first run |
| `demos/` | Runnable end-to-end scripts on throwaway tmux servers | `tests/e2e/parity-check.sh` is a CI gate, not a demo |
| `website/` | SvelteKit landing page for usecyclops.dev | Outside the Cargo workspace; modify only on explicit request. CI checks it and requires its installer to match `scripts/install.sh` |
| `findings.md` | Measured facts (F13+), each with its probe | Docs and comments cite these F-numbers |

## The gates a change must pass

The complete local gate is documented in [CONTRIBUTING.md](CONTRIBUTING.md).
`./scripts/check.sh` runs it cheapest first; `--fast` stops after Rust
correctness and documentation compilation. Performance executables run in the
scheduled and release lanes, not as ordinary correctness tests.

The paired build below is for
`workspace_cli::start_starts_a_daemon_when_none_is_running`, which intentionally
starts and asserts a real daemon. `workspace_boot_sizing`'s sizing assertion
does not require a daemon and tolerates daemon-start failure.

```bash
# ./scripts/check.sh --quick runs only this one-to-two-minute subset:
#   cargo fmt --all --check
#   cargo clippy --workspace --all-targets -- -D warnings
#   cargo nextest run --workspace -E 'kind(lib) | kind(bin)' --no-fail-fast
./tests/e2e/messaging-docs-parity.sh
cargo fmt --all --check
cargo clippy --workspace --all-targets -- -D warnings
./scripts/check-headless.sh
python3 scripts/check-doc-paths.py
cargo build -p cyclops -p cyclopsd --bins
cargo nextest run --workspace -E 'not (package(cyclopsd) | binary_id(=cyclops-ui::perf) | binary_id(=cyclops-ui::queue_perf) | binary_id(=cyclops-workspace::perf_contract))' --no-fail-fast
cargo test -p cyclopsd --all-targets --no-fail-fast
cargo doc --workspace --no-deps
./tests/e2e/parity-check.sh
```

- `--no-fail-fast` is not optional: nextest must keep scheduling so one run
  reports every failing test instead of hiding the remaining failures.
- Touching either installer requires keeping `scripts/install.sh` and
  `website/static/install.sh` byte-for-byte identical, then running
  `./tests/e2e/parity-check.sh --with-installer`.
- CI runs focused root-selection, source-boundary, and tmux/daemon/socket
  evidence with `CYCLOPS_TEST_TMP` relocated. Website, installer, tmux HEAD,
  macOS, and other platform evidence run on pull requests only when their owned
  inputs change. Full matrix, tmux HEAD, reliability, performance, and release
  evidence have explicit workflows described in
  [CI.md](docs/development/CI.md).

## Rules that are unusual for this repo

1. **Docs are CI-verified against the binaries.** Never hand-write output
   into a doc: `tests/e2e/parity-check.sh` re-runs every command shape the
   README and `docs/` quote and fails when a line drifts. If you change
   output a doc quotes, copy the new output from the parity transcript into
   the page in the same commit.
2. **Every path a doc quotes is checked**, including in this file:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cyclops-team/cyclops](https://github.com/cyclops-team/cyclops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
