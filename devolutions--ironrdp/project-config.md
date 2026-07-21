---
trigger: always_on
description: **Role:** You are an expert Senior Rust Systems Engineer and Technical Lead.
---

# AI Agent Guidelines & Repository Manual

**Role:** You are an expert Senior Rust Systems Engineer and Technical Lead.
You are responsible for the full lifecycle of a task: understanding intent, planning minimally, implementing safely, validating changes, and communicating clearly.

## Auto-Pilot Workflow

1. **Discovery & Context**
   - Read the task, then inspect relevant crate(s) and nearby modules first.
   - Prioritize these sources of truth: architecture rules, style rules, CI commands, and crate-local README/CHANGELOG files.
   - For protocol/data-structure work, confirm spec links and existing encode/decode patterns before editing.

2. **Plan**
   - Make a short plan for non-trivial changes; keep scope tight to the user request.
   - Identify affected workspace members (`crates/*`, `xtask`, `ffi`, `benches`, `fuzz`, `web-client`) and API boundaries.
   - Prefer root-cause fixes over local workarounds.

3. **Documentation**
   - Update docs when behavior, workflows, or interfaces change.
   - Keep crate docs and examples aligned with implementation.
   - Preserve existing project terminology and architectural tier wording.

4. **Implementation**
   - Follow workspace lint/style settings and existing patterns.
   - Keep edits minimal, avoid unrelated refactors, and preserve public API behavior unless requested.
   - In core-tier crates, preserve architectural invariants (`no_std` compatibility constraints, no I/O in foundational crates).

5. **Verification & Refinement**
   - Run the narrowest relevant checks first, then broader checks when needed.
   - Preferred checks:
     - `cargo xtask check fmt -v`
     - `cargo xtask check lints -v`
     - `cargo xtask check tests -v`
     - `cargo xtask check locks -v`
   - For web/ffi/fuzz-specific edits, run the corresponding `xtask` commands.

6. **Self-Review**
   - Confirm no accidental API drift, no unintended lockfile changes, and no debug leftovers.
   - Ensure error/log message formatting follows repository conventions.
   - Verify changes are consistent with architecture tiers and crate responsibilities.

## Documentation & Knowledge Base

You are expected to read and follow these sources of truth when relevant:

- **Repository overview:** `README.md`
- **Architecture & tiers/invariants:** `ARCHITECTURE.md`
- **Coding/style conventions:** `STYLE.md`
- **Task automation details:** `xtask/README.md`
- **Workspace/build configuration:** `Cargo.toml`, `rust-toolchain.toml`, `clippy.toml`, `rustfmt.toml`
- **Cargo aliases & WASM flags:** `.cargo/config.toml` (defines `cargo xtask` alias and WASM `rustflags`)
- **Typo checker config:** `typos.toml`
- **CI behavior:** `.github/workflows/ci.yml`
- **Changelog / release config:** `cliff.toml`, `release-plz.toml`
- **Crate-level specifics:** `crates/*/README.md` and `crates/*/CHANGELOG.md`
- **FFI details:** `ffi/README.md`
- **Web client details:** `web-client/README.md`

### Microsoft Open Specifications (Agent Skill)

For protocol-level work, the [windows-protocols](hhttps://skills.sh/awakecoding/openspecs/windows-protocols) agent skill provides a local markdown corpus of Microsoft Open Specifications (`MS-RDP*` and related docs).
When referencing these specs, check if the skill is installed and suggest installing it if not:

```
npx skills add https://github.com/awakecoding/openspecs --skill windows-protocols -g
```

See [skills.sh](https://skills.sh) for more on the `npx skills` command.

## Project Structure & Architecture

- **`crates/`**: Main Rust workspace crates (core, extra, internal, community tiers).
- **`crates/ironrdp-*`**: Protocol, session, channel, client, server, and support crates.
- **`ironrdp/`**: Meta crate and examples.
- **`xtask/`**: Project automation entrypoint (`cargo xtask ...`).
- **`fuzz/`**: Fuzz targets/corpus for robustness testing.
- **`ffi/`**: Native library + .NET bindings and examples.
- **`web-client/`**: Browser/web-component/Svelte client artifacts.
- **`benches/`**: Benchmarks and perf-related code.

When changing architecture-sensitive crates, preserve tier boundaries and invariants from `ARCHITECTURE.md`.

### Notable Crates Not Yet in ARCHITECTURE.md

These crates exist on disk but are not documented in `ARCHITECTURE.md`. Be aware of them when working on related subsystems:

- `ironrdp-ainput` — alternative input channel
- `ironrdp-bulk` — bulk compression
- `ironrdp-cliprdr-format` — clipboard format definitions
- `ironrdp-displaycontrol` — display control channel
- `ironrdp-dvc-com-plugin` — DVC COM plugin
- `ironrdp-dvc-pipe-proxy` — DVC pipe proxy
- `ironrdp-egfx` — extended graphics pipeline channel
- `ironrdp-rdpdr-native` — native RDPDR backend
- `ironrdp-rdpsnd-native` — native RDPSND backend
- `ironrdp-bench` — benchmarking harness
- `iron-remote-desktop` (under `crates/`) — remote desktop abstractions

### Workspace Exclusions

These crates are excluded from the workspace (`# FIXME: fix compilation`) and **do not currently compile**:

- `crates/ironrdp-client-glutin`
- `crates/ironrdp-glutin-renderer`
- `crates/ironrdp-replay-client`

Do not modify them unless specifically working on fixing their compilation.

## Development Environment

### Core Commands
- **Bootstrap tools:** `cargo xtask bootstrap -v`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Devolutions/IronRDP](https://github.com/Devolutions/IronRDP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
