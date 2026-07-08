---
trigger: always_on
description: nano-ros is a Rust workspace for a `no_std` ROS 2 client stack with C/C++ integration. Core crates live under `packages/core/`; RMW backends under `packages/zpico/`, `packages/xrce/`, and `packages/dds/`; board/platform support under `packages/boards/` and `packages/platforms/`; drivers under `packages/drivers/`; and reusable integration tests under `packages/testing/nros-tests/`. Shell and smoke fixtures live in `tests/`. Examples are standalone copy-out projects under `examples/`, with the can
---

# Repository Guidelines

## Project Structure & Module Organization

nano-ros is a Rust workspace for a `no_std` ROS 2 client stack with C/C++ integration. Core crates live under `packages/core/`; RMW backends under `packages/zpico/`, `packages/xrce/`, and `packages/dds/`; board/platform support under `packages/boards/` and `packages/platforms/`; drivers under `packages/drivers/`; and reusable integration tests under `packages/testing/nros-tests/`. Shell and smoke fixtures live in `tests/`. Examples are standalone copy-out projects under `examples/`, with the canonical shape `examples/<platform>/<language>/<example>/`; the RMW is selected at build time.

**Supported hosts: Linux (primary) and *BSD (POSIX path). macOS is NOT supported** (dropped 2026-06-18, phase-260): no macOS CI runner means macOS-specific link/section paths ship un-run, so the project does not carry them. Do not add `APPLE`/`target_os = "macos"`/`*-apple-darwin` branches to nano-ros source, CMake, or CI; embedded RTOS targets + the Linux host are the supported surface.

Reference and contributor docs live in `docs/`; user-facing mdBook docs live in `book/src/`; build orchestration lives in `justfile` and `just/*.just`.

## Design Documents (RFCs)

`docs/design/` is the design source of truth: numbered, living RFCs (`NNNN-slug.md`) with a
`status` of `Draft` / `Stable` / `Superseded`. [docs/design/ARCHITECTURE.md](docs/design/ARCHITECTURE.md)
is the finalized whole-system view; [docs/design/README.md](docs/design/README.md) is the index.
New RFC: copy `docs/design/0000-template.md`.

Two rules:

- **Design rationale goes in an RFC, never only in a phase doc.** Phase docs in `docs/roadmap/`
  are work breakdowns; they name the RFC they implement in an `Implements: RFC-NNNN` header.
- **Drift rule:** flipping an RFC to `status: Stable` requires updating the matching section of
  `ARCHITECTURE.md` in the same commit.

## Build, Test, and Development Commands

- `just --list`: show public recipes.
- `scripts/bootstrap.sh`: first-time entrypoint; installs/checks `just`, then prints setup choices.
- `scripts/bootstrap.sh base`: first-time native/ROS/zenoh quick-start setup.
- `scripts/bootstrap.sh all`: contributor/full-matrix setup; pulls and installs every supported SDK tier.
- `scripts/bootstrap.sh platform <platform>`: first-time focused setup for one platform.
- `just setup`: print setup choices; does not fetch/install.
- `just setup base`: install the base quick-start SDK/tooling tier.
- `just setup all` or `just setup tier=all`: install the full contributor/test-all tier.
- `just <platform> setup`: install a focused platform SDK/tooling tier.
- `just doctor` and `just doctor tier=all`: diagnose base or full setup readiness.
- `just build`: build the workspace plus generated bindings and transport artifacts.
- `just build-examples`: build the workspace and example matrix.
- `just build-test-fixtures`: prebuild binaries required by the full test matrix.
- `just build-all`: run the broad build tier; it auto-routes through the GNU make jobserver path when the pinned make/ninja tools are available.
- `just <platform> build`, `just <platform> build-examples`, `just <platform> build-fixtures`, `just <platform> build-all`: run platform-scoped tiers first when a platform-specific failure appears.
- `just test-unit`: fast workspace unit tests.
- `just test`: standard dev tier; skips heavy platform/ROS 2 groups.
- `just test-all`: full matrix, doctests, Miri, and C codegen tests. Run `just build-test-fixtures` first.
- `just check`: formatting and clippy checks across Rust, C, C++, and Python surfaces.
- `just ci`: `check` plus `test-all`.

Treat `<platform>` as target families such as `qemu`, `zephyr`, `freertos`, `nuttx`, `threadx_linux`, `threadx_riscv64`, `esp32`, or board groups. Support services such as `zenohd`, `cyclonedds`, and `xrce` are not platform scopes.

Codex sandbox notes:

- `just` may fail before running a recipe if the default runtime
  directory is read-only, with an error about creating a temporary
  directory under `/run/user/.../just`. In that environment, rerun with
  `XDG_RUNTIME_DIR=/tmp`.
- Cargo commands inside `just` may need to update the user's registry
  cache under `$HOME/.cargo`. If a recipe fails with read-only
  filesystem errors in `.cargo/registry`, rerun the same command with
  sandbox escalation rather than treating it as a project failure.
- A failed pre-nextest Cargo setup can leave an old
  `target/nextest/default/junit.xml` in place. If a recipe prints
  slow-test output after such a setup failure, verify whether nextest
  actually ran before trusting the timing report.

## Coding Style & Naming Conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NEWSLabNTU/nano-ros](https://github.com/NEWSLabNTU/nano-ros) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
