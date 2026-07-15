---
trigger: always_on
description: fbuild is a PlatformIO-compatible embedded build tool, organized as a small fixed set of crates (kept as close to a monocrate as possible — see the "Never add a new crate" rule below). See @docs/CLAUDE.md for which architecture doc to read based on what you're working on.
---

# CLAUDE.md

fbuild is a PlatformIO-compatible embedded build tool, organized as a small fixed set of crates (kept as close to a monocrate as possible — see the "Never add a new crate" rule below). See @docs/CLAUDE.md for which architecture doc to read based on what you're working on.

## Agent docs routing (FastLED/fbuild#695)

When operating in this repo on a task that isn't covered by the architectural overview, jump to the right per-topic doc instead of grepping for it:

| Task | Read |
|---|---|
| "Which `fbuild` subcommand do I run?" | [`agents/docs/commands-reference.md`](agents/docs/commands-reference.md) |
| "How does deploy actually get firmware to the board?" | [`agents/docs/deploy-architecture.md`](agents/docs/deploy-architecture.md) |
| "What DTR/RTS state do I open this CDC port at?" | [`docs/usb-cdc-control-line-matrix.md`](docs/usb-cdc-control-line-matrix.md) |
| "How do I run the serial detection code against a real ESP32?" | [`agents/docs/serial-testing.md`](agents/docs/serial-testing.md) (FastLED/fbuild#899 — Docker/WSL real-device harness) |
| "Where does this path/cache/build dir live, and why won't my cache key hit?" | [`agents/docs/path-conventions.md`](agents/docs/path-conventions.md) |
| "Which crate owns this code?" | [`crates/CLAUDE.md`](crates/CLAUDE.md) |
| "Which architecture doc maps to my crate?" | [`docs/CLAUDE.md`](docs/CLAUDE.md) |
| "Is this serial port the right device?" | `fbuild serial probe list` (FastLED/fbuild#686) |

The four rules an agent must internalize before doing anything else (all listed in "Essential Rules" below): **never bypass `soldr`**, **never bypass `uv`**, **every directory needs a README.md**, **never invoke `pyocd` / `esptool` / `dfu-util` / `picotool` directly — go through `fbuild deploy`** (the last is FastLED/fbuild#694's scope; the hook ban lands separately).

## Essential Rules

- **USB VID/PID source of truth:** Never add or embed a board/device VID or PID
  in fbuild code, tests used by production, generated Rust tables, or release
  artifacts. All VID/PID records must come from the published
  [FastLED/boards](https://github.com/FastLED/boards) registry and be ingested
  through its normal build/publish pipeline. Test-only fixtures may use
  synthetic or copied IDs to exercise parsing and selection, but they must not
  become runtime defaults. If a VID/PID is missing, fix the boards registry;
  do not add an exception here.

- **Modules-first for new functionality; crate splits only for compile parallelism (backed by `--timings`).** New *functionality* is still folded into an existing crate as a *module*, never a drive-by new crate — that original rule (no scope-creep crates) stands. The one sanctioned reason to add a workspace member is **splitting an existing giant crate to compile in parallel**, backed by `cargo build --timings` data and maintainer sign-off (FastLED/fbuild#1008 is that sign-off for the `fbuild-build` / `fbuild-packages` splits). Such splits keep the original crate as a thin **facade** that re-exports the extracted crates at their old paths, so consumers are unchanged. If code is needed by two crates that can't depend on each other (e.g. the CLI and the daemon), put the shared, dependency-free pieces in a crate both already depend on (`fbuild-core` / `fbuild-paths`). Enforced by CI (`crate-gate.yml` → `ci/check_workspace_crates.py`): adding a workspace member fails the build unless you also add it to the approved allowlist (and `ci/hooks/crate_guard.py`) with a maintainer-reviewed rationale.
- **Always use a globally-installed `soldr` to execute Rust commands.** Bare cargo/rustc and legacy `uv run cargo` shims are blocked by hook. soldr uses `rustup which` to pick the rustup-managed toolchain from `rust-toolchain.toml`. The standard Cargo path is `soldr cargo ...`, so repo Rust builds get soldr's managed zccache path by default; do not add repo-specific `RUSTC_WRAPPER` wiring for normal builds. Install soldr globally via `uv tool install soldr` (or see https://github.com/zackees/soldr).
- **Always use `uv` for Python.** Bare `python`/`pip` are blocked by hook. Use `uv run ...` or `uv pip ...`.
- MSRV: 1.94.1 | Edition: 2021 | Toolchain: 1.94.1 pinned in `rust-toolchain.toml` (clippy + rustfmt)
- CI: Linux, macOS, Windows. All warnings denied (`RUSTFLAGS="-D warnings"`)
- Every directory with files must have a README.md (enforced by hook)

## Commands

```bash
uv run test                 # unit tests only
uv run test --full          # unit + stress + integration tests
uv run test -p <crate> -- <test_name>
soldr cargo check --workspace --all-targets
soldr cargo clippy --workspace --all-targets -- -D warnings
soldr cargo fmt --all
RUSTDOCFLAGS="-D warnings" soldr cargo doc --workspace --no-deps

# Public deploy API conventions
soldr cargo run -p fbuild-cli -- deploy tests/platform/uno -e uno --to emu
soldr cargo run -p fbuild-cli -- deploy tests/platform/uno -e uno --to emu --monitor
soldr cargo run -p fbuild-cli -- deploy tests/platform/esp32dev -e esp32dev-qemu --to emu --emulator qemu


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FastLED/fbuild](https://github.com/FastLED/fbuild) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
