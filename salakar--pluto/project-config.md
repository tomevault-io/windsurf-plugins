---
trigger: always_on
description: This is the fast path for an agent (or a new human) to become productive in the
---

# AGENTS.md — Pluto engineering playbook

This is the fast path for an agent (or a new human) to become productive in the
Pluto repository. It covers the toolchain, the device workflow, tests,
benchmarks, deployment (AOT vs. JIT), hot reload, and the `pluto` CLI. Read
[`README.md`](README.md) for the elevator pitch and
[`docs/GETTING_STARTED.md`](docs/GETTING_STARTED.md) for the guided first run;
this file is the reference you keep open while working.

## What Pluto is

Pluto runs Flutter apps on supported reMarkable e-ink tablets through one
discovery-driven product and CLI. Hardware-specific display and lifecycle
backends are selected internally:

- a custom **software-rendering Flutter embedder** (`embedder/`, C++), with an
  e-ink renderer and presenters that drive the panel's waveform controller;
- the **`pluto` CLI** (`tools/pluto/`, Dart) for device discovery,
  provisioning, build, install, logs, screenshots, and hot reload;
- a Flutter **Home/launcher** (`apps/launcher/`) that is the common entry point
  for Pluto applications;
- **`pluto_*` Dart packages** (`packages/`) exposing device, settings, pen,
  touch, sensors, manifest, provision, and UI APIs;
- **example and product apps** (`apps/`): counter, motion_lab, ink_lab,
  validation_lab, ink (drawing), codex (terminal).

Apps ship as release AOT by default. JIT exists only behind explicit `--debug`
flags for hot reload and can never become the boot default.

## Repository map

| Path | What lives there |
| --- | --- |
| `embedder/` | Native embedder, renderer, compositor, presenters (`swtcon`, `qtfb`, host preview), CMake presets, C++ tests + benches |
| `tools/pluto/` | The `pluto` CLI (standalone Dart package, resolves outside the workspace) |
| `tools/setup/` | `setup.sh` bootstrap; `camera/` panel-capture helper |
| `tools/build/` | Host and ARM device builds; target payload assemblers |
| `tools/device/` | On-device backends, installer/uninstaller, power/standby, safety harness, diagnostics, shell-contract tests |
| `tools/engine/` | Flutter engine rebuild + promotion (pin maintainers only) |
| `packages/pluto_*` | Dart API packages |
| `apps/` | Launcher, examples, ink, codex, validation_lab |
| `third_party/engine/<hash>/` | Pinned, checksummed Flutter engine payloads (committed on purpose) |
| `docs/` | Getting started, AOT runtime, lifecycle, rendering, icon style, optimisation log, camera |
| `ci/check.sh` | The complete quality gate |

## Toolchain and environment

Everything is pinned. The Flutter SDK and the device engine are exact versions;
do not use a system `flutter`.

- **Flutter**: `3.44.4` (Dart `3.12.x`) — `tools/pluto/pins/flutter.version`.
- **Engine**: `a10d8ac38de835021c8d2f920dbf50a920ccc030` —
  `tools/pluto/pins/engine.version`.

Bootstrap once:

```bash
./tools/setup/setup.sh
```

This validates the committed AOT runtimes, installs the pinned SDK under
`~/.pluto/sdk/3.44.4` if absent, activates Melos, bootstraps the pub workspace,
resolves the CLI package, and compiles `~/.pluto/bin/pluto`. Then put the three
tools on your PATH (setup prints this line):

```bash
export PATH="${PLUTO_BIN_DIR:-$HOME/.pluto/bin}:${PUB_CACHE:-$HOME/.pub-cache}/bin:${PLUTO_SDK:-$HOME/.pluto/sdk/3.44.4}/bin:$PATH"
```

Key environment variables: `PLUTO_SDK` (SDK location), `PLUTO_BIN_DIR` (CLI
install dir), `PLUTO_ROOT` (on-device runtime root, default `/home/root/pluto`).

`./tools/setup/setup.sh --verify` re-checks an installation and all committed
artifacts offline without changing anything.

**Docker** is required on macOS and x64 Linux for the committed AOT snapshot
tools and cross-builds of device-native components. Native AArch64 Linux hosts
can execute the AArch64 snapshotter directly; target-specific native builds
still use their pinned toolchains.

## The Flutter engine — when to rebuild (usually never)

The plain-embedder `libflutter_engine.so` payloads, matching `gen_snapshot`
tools, and ICU data are **committed** under
`third_party/engine/<engine-hash>/` with checksum manifests. The repository
contains AArch64 release/profile and ARMv7 release artifacts. A normal clone
never builds or downloads an engine; `setup.sh` and the CLI verify the pinned
payloads before use.

Rebuild the engine only when you are **changing the Flutter pin** or recovering
the committed artifacts. It is a pin-maintainer task, not part of app or
embedder development:

```bash
tools/engine/build-aarch64-aot.sh     # Docker linux/arm64; builds plain embedder archives
tools/engine/promote-aarch64-aot.sh   # checksum-gated copy into third_party/engine/<hash>/
```

Build products stay in the ignored `.pluto-cache/`. Promotion refuses an
unreviewed pin, re-validates checksums, and checks the built ABI header against
the tracked embedder header. See `tools/engine/README.md`.

Note the distinction: rebuilding the **embedder** (`embedder/`, our C++) is
routine and does *not* touch the Flutter engine — see the build section below.

## Connecting to a reMarkable device

Read the tested model/firmware matrix in
[`docs/device-compatibility.md`](docs/device-compatibility.md) before changing
a device. The public workflow never asks the user to choose a backend:
`pluto devices --probe` reads immutable hardware identity, and provision,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Salakar/pluto](https://github.com/Salakar/pluto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
