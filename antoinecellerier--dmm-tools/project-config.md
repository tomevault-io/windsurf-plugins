---
trigger: always_on
description: Rust workspace for communicating with digital multimeters via USB (CP2110, CH9329, CH9325 and BU-86X HID cables) and Bluetooth LE. Supports UNI-T, Voltcraft, ZOTEK, EEVblog, Brymen and OWON meters.
---

# dmm-tools

Rust workspace for communicating with digital multimeters via USB (CP2110, CH9329, CH9325 and BU-86X HID cables) and Bluetooth LE. Supports UNI-T, Voltcraft, ZOTEK, EEVblog, Brymen and OWON meters.

## Project structure

- `crates/dmm-lib/` — library: CP2110/CH9329/CH9325/BU-86X and Bluetooth LE transports, protocol framing (AB CD and 0xAC extractors), measurement parsing, device tables. Protocol families: `ut61eplus`, `ut8802`, `ut8803`, `ut80x` (UT803/UT804, and the UT71 and Voltcraft VC920/940/960 on the UT804's packets), `ut171`, `ut181a`, `vc8x0` (VC-880/VC-890), `zotek` (ZOTEK's Bluetooth meters, sold as ZOYI, BSIDE and ANENG), `eevblog121gw` (EEVblog 121GW), `bm78xbt` (Brymen BM788BT/BM787BT), `bm86x` (Brymen's BM860s, BM820s and BM520s series on the BU-86X cable), `owon` (OWON's Bluetooth meters). Protocol internals (framing, per-family parsers) are `pub(crate)` — consumers use the `Dmm` API, not raw frame extraction; only `protocol::registry` is `pub` for the CLI/GUI, plus the fixtures their tests take behind the `test-support` feature (`MockTransport`, `Measurement::test_fixture`, `protocol::make_test_measurement`).
- `crates/dmm-shared/` — app-only things the CLI and GUI must agree on that `dmm-lib` has no business carrying, compile-enforced: the settings schema (`SharedSettings { device_family }`), `write_atomic`, `logging` (the default log levels), `help` (the help and version text both binaries print), `replay` (what `--replay` opens: a replay file, or a golden fixture as a one-frame recording), and `export` (the CSV columns, the JSON reading shape and the `measurements-<meter>-<mode>-<start>.<ext>` file name both binaries write). GUI-only settings fields (colors, panel visibility, theme) live in `dmm-gui` and merge via `#[serde(flatten)]`. Export formats stay identical in both binaries: a new format or field lands in both in one change.
- `crates/dmm-cli/` — CLI binary `dmm-cli`.
- `crates/dmm-gui/` — GUI binary `dmm-gui` (eframe/egui).

## Build & test

- `cargo build --workspace`
- `cargo test --workspace`
- `cargo clippy --workspace --all-targets -- -D warnings` (must pass clean)
- `cargo fmt --check`

A pre-commit hook (`git-hooks/pre-commit`) runs fmt, clippy, and the test suite on every commit; a commit-msg hook (`git-hooks/commit-msg`) rejects a `Claude-Session:` trailer. Fix failures; never bypass with `--no-verify`.

## Working with the user

### Physical device interaction
- When testing requires physical device interaction (dial position, lead placement, connections), describe the setup and **wait for user confirmation** before each step. Never assume the device is in the right state.
- What a check can answer, check yourself before asking: `dmm-cli list` shows any attached cable or meter. When a step needs a cable out and the check finds one (e.g. `scripts/doc-screenshots.sh` names skipped scenes), ask the user to unplug it, wait for confirmation, then rerun the skipped part.

### Specification data
- **Never fabricate specification data.** If a value cannot be directly read from the source document, mark it unknown rather than guessing.
- Verify PDF-sourced specs against the actual PDF rendering, not text-extracted versions — PDF-to-text conversion corrupts multi-column and merged-cell tables.

### Simplicity and scope
- Default to the simplest approach that meets the requirement. No speculative abstractions or "just in case" configurability.
- When told "keep it simple", revert anything that adds complexity without clear value.
- When a request is ambiguous about scope, ask before implementing.
- For multi-step work, persist progress in durable files (an untracked `<NAME>_PROGRESS.md`, excluded via `.git/info/exclude`) — sessions can run out of context.

### Plans and background agents
- Before presenting a plan, have a subagent critique it and fold in the fixes.
- While background agents run, end the turn; their notification wakes you. Don't poll.

### Tool usage
- Prefer Read/Grep/Glob/Write/Edit over Bash to reduce permission prompts.
- Run git commands directly — cwd is the repo. Don't use `git -C`.
- With Edit's `replace_all`: check the pattern doesn't match unintended locations (e.g. replacing `AcDcA` also catches `AcDcA2`). Prefer targeted single replacements.

## Engineering standards


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [antoinecellerier/dmm-tools](https://github.com/antoinecellerier/dmm-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
