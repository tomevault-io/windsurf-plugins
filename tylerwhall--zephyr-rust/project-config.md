---
trigger: always_on
description: - This repository is a **Zephyr module** that enables Rust applications to be linked into Zephyr images. Applications add this module through `ZEPHYR_EXTRA_MODULES` (see sample/test `CMakeLists.txt`).
---

# Agent instructions for `zephyr-rust`

## High-level architecture

- This repository is a **Zephyr module** that enables Rust applications to be linked into Zephyr images. Applications add this module through `ZEPHYR_EXTRA_MODULES` (see sample/test `CMakeLists.txt`).
- Build flow is split between Zephyr CMake and Rust Cargo:
  1. Top-level `CMakeLists.txt` derives `rust_target`/`clang_target` from Zephyr `ARCH`/Kconfig.
  2. `scripts/gen_syscalls.py` generates syscall thunk C/header files from Zephyr syscall metadata.
  3. `zephyr-bindgen` is built and invoked by `rust/zephyr-sys/build.rs` to generate Rust FFI bindings (`bindings.rs`, `syscalls.rs`).
  4. `rust/genproject.sh` creates a generated Cargo project that depends on the app crate (from the sample/test directory).
  5. `rust/build.sh` builds a custom sysroot (`rust/sysroot-stage1`) and then builds the app staticlib (`librust_app.a`), which CMake imports and links into the Zephyr app.
- Crate layering is intentional:
  - `zephyr-sys`: generated/raw FFI and syscall bindings
  - `zephyr-core`: core/no_std-safe wrappers and context-aware syscall traits
  - `zephyr`: std-facing API layer built on `zephyr-core`
  - helper crates: `zephyr-macros`, `zephyr-futures`, `zephyr-logger`, `zephyr-uart-buffered`
- C shims (`src/main.c`) are the ABI bridge: Zephyr C entrypoints call exported Rust symbols (`extern "C"`, `#[no_mangle]`).

## Build, test, and run commands

Builds can be done using natively with local versions of rust toolchain,
zephyr-sdk, west, and the zephyr source. A container-based build environment
for CI exists in ./ci. Using the containers is preferred for performing
development/maintenance on this repo, where modifications to Zephyr source is
not required.

The default application and machine target if not specified is samples/rust-app
on qemu_x86.

### Prerequisites used by this repository
- Make sure submodules are in sync: `git submodule status --recursive`, `git diff --submodule`
  - There may be local commits above the submodule version, but the base should be the submodule rev
- Rust toolchain is pinned to the version in rust-toolchain.toml (enforced by `rust/build.sh`).
- Use a Zephyr version this repo targets (documented in `README.md`).

### Build natively
- General form: `west build -p auto -b <board> <sample-or-test-path>`
- ./samples/rust-app is the catch-all example/integration test. When run, it exits non-zero by design: it intentionally triggers a page fault at the end ("Next call will crash if userspace is working") to prove user-mode isolation. Success is the full "Hello from Rust userspace..." console output before the fatal error.
- Example for different machines:
  - `west build -p auto -b qemu_x86 samples/rust-app/`
  - `west build -p auto -b native_posix samples/rust-app/`

### Run a built QEMU/native image
- Native, from build dir: `ninja run`
- CI container, single step (build + run in one ephemeral container; `-d /tmp/build` is required because the repo is mounted read-only):
  - `cd ci && ./build-cmd.sh bash -c "west build -d /tmp/build -p auto -b qemu_x86 samples/rust-app -t run"`
- CI container, multiple steps (persist the build dir across invocations with a host volume via `DOCKER_ARGS`):
  - `cd ci && DOCKER_ARGS="-v /tmp/zr-build:/tmp/build" ./build-cmd.sh west build -d /tmp/build -p auto -b qemu_x86 samples/rust-app`
  - `cd ci && DOCKER_ARGS="-v /tmp/zr-build:/tmp/build" ./build-cmd.sh ninja -C /tmp/build run`

### Clippy
- `ci/clippy.sh` runs `cargo clippy` on all Rust crates: the host crates
  (`zephyr-bindgen`, `zephyr-macros`), every sample/test app crate, and the
  app-layer library crates. The sysroot-layer crates (`zephyr-sys`,
  `zephyr-core`, `time-convert`) are linted via `-p` from the sysroot-stage1
  workspace, so only rustc lints surface there, never clippy (mechanism and
  tracked debt in `docs/CLIPPY_SYSROOT_DEBT.md`). Each app is
  `west build`-ed in its own build dir first, because the cross-compiled
  sysroot (and the
  `zephyr-sys` bindings generated from the app's headers/devicetree/Kconfig)
  is app-specific; clippy then reuses that build's sysroot and environment.
- CI container: `cd ci && ./build-cmd.sh ci/clippy.sh`
- Natively (west, Zephyr, Zephyr SDK, and the clippy component must be
  available): `./ci/clippy.sh`
- Select what to lint with positional arguments: `ci/clippy.sh` (everything,
  the CI default), `ci/clippy.sh lib` (only the common library crates),
  `ci/clippy.sh serial` (one app, by `samples/`/`tests/` dir name or path),
  or any combination like `ci/clippy.sh lib serial`.
- Apps that cannot be *built* on the selected board are reported as skipped
  (some tests only build on certain Zephyr versions); set `CLIPPY_STRICT=1`
  to treat that as a failure. Warnings are not fatal by default; set
  `CLIPPY_ARGS="-D warnings"` to make them so. Other knobs: `CLIPPY_BOARD`
  (default `qemu_x86`), `CLIPPY_BUILD_DIR`, `CLIPPY_JOBS`.
- A `clippy` job in `.github/workflows/main.yml` runs this on Zephyr 3.7.0
  with `CLIPPY_ARGS="-D warnings"`, so new warnings fail CI.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tylerwhall/zephyr-rust](https://github.com/tylerwhall/zephyr-rust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
