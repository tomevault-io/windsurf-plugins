---
trigger: always_on
description: This file is for AI coding agents (and is equally useful to new human
---

# Conventions for agents working in Tilefinch

This file is for AI coding agents (and is equally useful to new human
contributors). It records the conventions that this repository actually
enforces — most of them through CMake, CTest, or a committed ratchet file — so
that a change either follows them or fails a gate.

Start with [README.md](README.md) for what the project is, and
[docs/README.md](docs/README.md) for the documentation map.

## Build and test

The canonical gate is the optimized host build and its test suite:

```sh
cmake --build build-preset-release -j8
ctest --test-dir build-preset-release -j8 --output-on-failure
```

Release currently registers **148 tests: 147 enabled plus the opt-in
`tilefinch-device-cost-tests`, which is registered but disabled by default**.
All enabled tests must pass. The localhost
redirect test may report `Skipped` in a sandbox that forbids loopback sockets;
the update-root proof can likewise skip when its external prerequisite is not
available. A skip is not a passing substitute for running either gate in an
ordinary host environment.

For a faster edit loop, configure rather than trusting a previously-created
development tree:

```sh
cmake --preset dev
cmake --build --preset dev
ctest --preset dev
```

The development preset omits release-only gates such as
`tilefinch-fidelity-floor-tests`. The legacy `./scripts/dev.sh unit` command
remains useful for aggregate unit filters. Test counts legitimately differ
between configurations, so compare a tree against itself, not against another
tree.

Sanitizers (`cmake --preset sanitize`) and the hostile-input parser harness
(`cmake --preset hostile`) are explicit gates, not part of the edit loop.
Details are in [docs/DEVELOPMENT.md](docs/DEVELOPMENT.md).

## The PSP cross-build is a separate gate

It is never part of the test suite and never runs implicitly:

```sh
PSPDEV=/path/to/pspdev cmake --preset psp
cmake --build build-preset-psp \
  --target tilefinch_core psp-browser-fixture psp-browser-script
```

Build named PSP targets only. The host lab and input-latency executables are
not device targets, and the bare cross-build `all` target is not the PSP gate.

Two things only this build enforces:

- **A 4,480,000-byte ordinary `.text` ratchet** (4,500,000 bytes when
  validation logging is compiled in). `cmake/CheckPspTextSize.cmake` reads the
  actual ELF `.text` sections with `psp-objdump` after every link, reports
  `.rodata` separately, and fails the build above the appropriate limit.
  The aggregate `psp-size` `text` column is not used because it includes
  read-only data and does not isolate executable code.
  Raising either ratchet requires a re-measurement and a device-cost
  justification, not a bigger number.
- **The real Allegrex toolchain.** Host-only assumptions (64-bit pointers,
  `%u` with `uint32_t`, glibc-isms) surface here and nowhere else.

Run it after any change that adds code paths, dependencies, or data tables,
and record the result. `./scripts/dev.sh psp` wraps the same two commands.

**On macOS, never invoke `PPSSPPSDL` directly from a non-GUI automation shell.**
That process context can abort in Cocoa's `_RegisterApplication` before any
PSP code runs. Use `scripts/launch-ppsspp-safe.sh` for manual runs or
`scripts/run-ppsspp-network.sh` for the automated smoke; both launch the app
through LaunchServices and repair only a disposable copy when Homebrew's app
bundle seal is invalid.

**Device-facing code is shared, not copied.** The browser EBOOT
(`src/psp_script_main.c`) and the qualification fixture (`src/psp_main.c`) must go
through the same modules for anything the hardware sees — scanout lives in
`src/psp_display.c` and the chrome in `src/psp_ui.c`, each behind a header in
`include/tilefinch/`. A seam that lets the host substitute a fake (see
`PspDisplayBackend`) makes device logic testable in CTest; prefer it to a
second implementation.

The two PSP executables also write their EBOOTs to different directories on
purpose: the browser to `build-preset-psp/`, which is what every script and doc
means, and the fixture to `build-preset-psp/fixture/`. Never merge these output
paths: validation must not mistake the fixture for the browser.

## Regenerating the JavaScript bootstrap

`src/bootstrap/*.js` is compiled into two committed C files. If you edit any
of those `.js` sources you must regenerate **both** artifacts and the manifest:

```sh
cmake --build build-preset-release --target regenerate_tilefinch_bootstrap
```

That target rewrites `src/generated/js_bootstrap.c` and `src/generated/js_bootstrap_bytecode.c`
and derives `src/bootstrap/generated.sha256` from the exact authored and
generated tree. Never edit `generated.sha256` by hand.

Both host and PSP builds make `tilefinch_core` depend on
`check_tilefinch_bootstrap_generated`, so a stale artifact fails the *next* core
build either way. The difference is what each can check: a host build checks
bytecode through the generator's `--check` mode (with content-keyed reuse as
described below) *and* verifies the manifest, while the
PSP build cannot run a host QuickJS generator through the cross toolchain and
therefore verifies the manifest only. A hand-edited manifest would pass the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stjanovitz/tilefinch](https://github.com/stjanovitz/tilefinch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
