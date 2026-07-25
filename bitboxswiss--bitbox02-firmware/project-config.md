---
trigger: always_on
description: Core firmware and bootloader code sits in `src/`, grouped by subsystem (`bootloader`, `usb`, `ui`,
---

# Repository Guidelines for bitbox02-firmware

## Project Structure & Module Organization
Core firmware and bootloader code sits in `src/`, grouped by subsystem (`bootloader`, `usb`, `ui`,
`securechip`, plus a `rust/` workspace). Tests live in `test/`: `unit-test/` for cmocka suites,
`hardware-fakes/` for device shims, and `simulator/` assets. The BitB02 Python client library is in
`py/bitbox02`. Supporting tooling is in `scripts/` (CI, J-Link macros), and `doc/` for
manuals. Vendored dependencies are tracked in `external/`.

The firmware has C and Rust code. Rust code lives in src/rust. The most important rust crates are:
- bitbox02-rust: the main app logic. It can expose functions to C using extern "C". If it needs
  access to C functions, it has to go through the bitbox-hal crate. Never add bitbox02 or
  bitbox02-sys dep to bitbox02-rust.
- bitbox02-sys: generated bindings to bitbox02 specific C code. build.rs contains the functions etc
  that are exposed. See also `wrapper.h`, it needs to include any C headers/declarations that are
  added to build.rs.
- bitbox-hal: provides an interface to device specific functionality
- bitbox02: wraps bitbox02-sys as idiomatic safe Rust and implements the bitbox-hal interface.

bitbox02-rust is pure Rust and device agnostic. To access device specific functionality it must
always go through bitbox-hal. The migration is a work in progress, only migrate what is necessary
for the current scope.

## Build, Test, and Development Commands
- `make dockerpull` / `make dockerdev`: fetch and enter the maintained development container.

Run regular Unix commands such as `git`, `rg`, `grep`, `ls`, `find`, `sed`, and `cat` directly on
the host.

Before running Rust workspace commands directly, run `./scripts/bootstrap-cargo-config` once in the
environment where Cargo will run. It writes `.cargo/config.local.toml` with the bindgen and `cc`
crate target settings for the local ARM sysroot. The generated file is included by Cargo config
files. Re-run it after changing ARM toolchains or sysroots.

Cargo commands for the Rust workspace, such as `cargo test`, `cargo check`, and `cargo clippy`, may
also be run directly on the host by passing `--manifest-path src/rust/Cargo.toml`.

Use `./scripts/dev_exec.sh <command>` only for project-specific commands that depend on the project
toolchain or compiler environment.

In practice, the repository `make` targets in this file are project-specific toolchain commands.
When running from the host, invoke them via `./scripts/dev_exec.sh make <target>`.

Do not wrap `./scripts/dev_exec.sh` itself in `bash -lc`. Prefer changing CWD
with CLI args like `tar -C <PATH>`. If a command genuinely needs shell features such as pipes, pass
an explicit shell as the command, e.g. `./scripts/dev_exec.sh bash -lc 'cat versions.json | jq'`.

- `make firmware` / `make bootloader`: compile firmware or bootloader ELFs into `build/`.
- `make simulator`: build the Linux simulator under `build-build-noasan/bin/`.
- `make unit-test && make run-unit-tests`:  build and run the C cmocka/CTest suite with ASan/UBSan.
- `make run-rust-clippy`: lint Rust code with the workspace configuration.
- When invoking the above `make` targets from the host, prefer
  `./scripts/dev_exec.sh make <target>`.
- Rust workspace commands may also be run directly with `cargo`, without `./scripts/dev_exec.sh`:
  -  From the repository root on the host, use
     `cargo test --manifest-path src/rust/Cargo.toml [ -p <crate> ] --all-features -- --test-threads 1`.
  -  For checks, use
     `cargo check --manifest-path src/rust/Cargo.toml [ -p <crate> ] --all-features`.
  -  If you modify `messages/*.proto`, run `make generate-protobufs` before direct Rust `cargo`
     commands. Plain `cargo test`/`cargo check` does not regenerate the protobuf outputs.

You may use `make -j$(nproc)` to speed up compilation. Do not use `make -j` without specfiying the
number of processing units.

Never run multiple `make` commands in parallel. For example, instead of
`make -j$(nproc) firmware bootloader factory-setup`, run them one after the other: `make -j$(nproc) firmware; make -j$(nproc) bootloader; make -j$(nproc) factory-setup`.


## Coding Style & Naming Conventions
`.clang-format` (Chromium base, 4-space indent, Linux braces) and `.clang-tidy` govern C/C++. Use
`snake_case` for symbols, `PascalCase` for types, and `ALL_CAPS` for macros. Python utilities follow
`.pylintrc` rules (100-column limit, explicit imports). Rust crates rely on `rustfmt.toml` and the
pinned toolchain in `rust-toolchain.toml`; keep module paths aligned with `src/rust` and regenerate
bindings (`cbindgen`, protobuf) when interfaces change. When changing protobuf interfaces, run
`make generate-protobufs`.

* For C code changes, run `./scripts/dev_exec.sh ./scripts/format` to format the code.
* For Python changes, run `./scripts/dev_exec.sh ./scripts/format-python` to format the code.
* For Rust code changes, run
  `./scripts/dev_exec.sh cargo fmt --manifest-path src/rust/Cargo.toml --all` to format the code.

## Testing Guidelines
Place new C specs in `test/unit-test` and add doubles to `test/hardware-fakes` when hardware

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BitBoxSwiss/bitbox02-firmware](https://github.com/BitBoxSwiss/bitbox02-firmware) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
