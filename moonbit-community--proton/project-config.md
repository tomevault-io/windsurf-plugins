---
trigger: always_on
description: This document is for contributors and release maintainers working on the Proton
---

# Proton Maintainer Guide

This document is for contributors and release maintainers working on the Proton
repository itself. It defines the architecture boundaries, source layout,
validation expectations, generated-file policy, and release procedure.

Application developers should start with [README.md](README.md). Do not move
repository build steps, native ABI internals, prebuilt synchronization, or
package publication instructions into the root README unless an application
developer must perform them.

## Maintainer Workflow

- Read the nearest package README before changing a subsystem, but treat this
  file and `native/CMakeLists.txt` as the repository-wide maintenance rules.
- Preserve the single native DLL runtime route and the public root facade.
- Use the smallest relevant checks while iterating, then expand validation in
  proportion to the affected runtime, platform, generated code, or release
  surface.
- Keep generated sources and user-facing examples synchronized with their
  templates and implementation.
- Never publish from an unverified dependency chain or use repository-local
  overrides for the final release smoke test.

## Project Structure
- `native/`: standalone CMake project for the Proton native runtime. It builds
  `proton` as a dynamic library/import library, installs `proton_native.h`, and
  installs the helper executable when the engine build is enabled.
- `proton/`: root `justjavac/proton` MoonBit module. The public facade owns the
  app API (`html`, `url`, `file`, `asset`, `config`), command-extension bridge
  wiring, and selected low-level native re-exports.
- `proton/native/`: safe MoonBit binding over the `proton_*` C ABI. MoonBit code
  links only the native Proton library through `proton/native_link_config.mjs`.
- `proton/manifest/`, `proton/bootstrap/`, `proton/catalog/`,
  `proton/core/`, `proton/command/`, `proton/ipc/`: supporting packages for
  metadata, tooling, command bridge wiring, and transport-neutral IPC protocol
  helpers. Do not reintroduce the old app runtime route without an explicit
  design decision.
- `cli/`: `justjavac/proton_cli`; independent native developer CLI module plus
  `cli/codegen/` and `cli/doctor/` helpers.
- `extensions/`: `justjavac/proton_ext`; command extensions for examples and
  applications. Platform capability extensions are backed by the bindings
  under `sys/`.
- `sys/<pkg>/`: independently maintained native system capability binding
  modules (`justjavac/auto_launch`, `justjavac/clipboard`,
  `justjavac/global_hotkey`, `justjavac/keepawake`, `justjavac/microphone`,
  `justjavac/tray`). Each keeps its upstream module name, version lineage,
  and MIT license, and is published from this repository. `justjavac/ffi`
  remains an external dependency maintained upstream.
- `examples/`: runnable demos. Keep [examples/Readme.md](examples/Readme.md)
  aligned with the actual examples.
- `proton/prebuilt/<platform>/`: shipped Proton-only native artifacts. Do not
  put CEF runtime files here.
- `lib/`, `build/`, `_build/`, `target/`, `native/build*`, `native/dist/`:
  generated or vendored artifacts.
- `.proton/`: generated project runtime cache created by `proton_cli cef setup`.

## Build And Test
- Native engine build:
  `cmake -S native -B native\build-engine -DCMAKE_INSTALL_PREFIX=native\dist -DPROTON_WITH_ENGINE=ON -DPROTON_ENGINE_ROOT=.cef-cache`
- `cmake --build native\build-engine --config Debug`
- `cmake --install native\build-engine --config Debug`
- `ctest --test-dir native\build-engine -C Debug --output-on-failure`
- `node native\scripts\verify_link_config.mjs native\dist`
- Sync release artifacts into `proton/prebuilt/<platform>/`; only include the
  Proton DLL/shared library, import library if any, helper executable, public
  header, and manifest.
- Build release artifacts with the Release configuration and install or stage
  stripped Proton binaries. On macOS, generate any required dSYMs from the
  unstripped build outputs first, then strip and stage the final binaries before
  code signing and notarization.
- `node scripts/verify_prebuilt_abi.mjs <platform>`
- `moon -C cli run . -- -C .. cef setup`
- With `.proton\runtime.json` active runtime `bin` on `PATH`:
  `moon -C proton test native --target native --diagnostic-limit 80`
- With `.proton\runtime.json` active runtime `bin` on `PATH`:
  `moon -C proton check --target native --diagnostic-limit 80`
- With `.proton\runtime.json` active runtime `bin` on `PATH`:
  `moon -C examples build --target native --diagnostic-limit 80`
- With `.proton\runtime.json` active runtime `bin` on `PATH`:
  `moon -C cli test -p justjavac/proton_cli justjavac/proton_cli/arguments justjavac/proton_cli/build_cmd justjavac/proton_cli/cef justjavac/proton_cli/codegen justjavac/proton_cli/dev justjavac/proton_cli/doctor justjavac/proton_cli/new justjavac/proton_cli/package --target native --no-parallelize --diagnostic-limit 80`
- `moon check --target native`
- `moon -C cli test codegen --target native`
- `node scripts/verify_generated.mjs`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [moonbit-community/proton](https://github.com/moonbit-community/proton) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-31 -->
