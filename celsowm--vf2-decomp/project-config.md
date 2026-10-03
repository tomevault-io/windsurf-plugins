---
trigger: always_on
description: This file is the operational handoff for coding agents working on `vf2-decomp`.
---

# AGENTS.md

This file is the operational handoff for coding agents working on `vf2-decomp`.
It is intentionally more prescriptive than the README. Read it before making
changes, then read the focused recovery documents for the subsystem you touch.

## Mission

`vf2-decomp` is a clean-room, non-matching C17 recovery of **Virtua Fighter 2
Version 2.1** for Sega Model 2A / Intel i960.

The goal is **portable recovered behavior**, continuously proven against the
original i960 program. The reference executor is an oracle and exploration tool;
it is not the desired final implementation.

The repository contains no ROMs. Never add ROM data or derived proprietary
artifacts to Git.

## Non-negotiable rules

1. **Evidence before implementation.** Do not invent game semantics, object
   layouts, names, branch conditions or hardware behavior.
2. **Fail closed.** Unverified paths must remain `VF2_ERROR_UNSUPPORTED` or an
   explicit ROM-backed boundary. Never make an unknown path silently succeed.
3. **The original i960 execution is the oracle.** A plausible implementation is
   not accepted until it matches measured reference behavior at a controlled
   boundary.
4. **Preserve exact state where the differential contract requires it.** This may
   include registers, condition state, local frames, call/return counters,
   scheduler state, mutable Model 2A memory and modeled device-visible state.
5. **Do not weaken validation to make a recovery pass.** Fix the recovery or
   improve the evidence instead.
6. **Keep the Model 2A oracle behavior stable.** Instrumentation must be passive.
   Observers may record successful accesses; they must not decide hardware
   behavior or mutate state.
7. **Generated pseudocode is navigation only.** Never copy generated pseudo-C
   wholesale into `src/recovered/` and call it recovered.
8. **No proprietary artifacts in commits.** In particular, do not commit ROMs,
   reconstructed ROM regions, `.vf2snap` files, large/full traces, extracted
   textures/models/audio, or generated pseudo-C derived from the ROM.
9. **Prefer the smallest proven semantic change.** Broad speculative rewrites
   make differential debugging much harder.
10. **When uncertain, preserve the boundary.** Unknown is better than wrong.

## First files to read

Start with these, in this order:

- `README.md` — project overview, build and top-level tools.
- `docs/UNCOVERED_BRANCHES.md` — current native/ROM-backed frontier.
- `docs/NATIVE_DIFFERENTIAL.md` — exact validation contract.
- `docs/NATIVE_RUNTIME.md` — recovered runtime architecture.
- `docs/DECOMP_GUIDE.md` — recovery lifecycle and evidence conventions.
- `docs/PROBE_AUTOMATION_PLAN.md` — automated probing/exploration workflow.
- `decomp/i960/notes/` — address-level evidence.
- `CHANGELOG.md` — useful context for why a boundary exists.

Do not trust a status sentence in this file over newer measured evidence. If the
repository has advanced, update this handoff as part of the same work.

## Windows environment

The canonical agent workflow on this checkout is **native Windows CMake + MSVC**.
The committed `build/` directory is configured with the Visual Studio generator,
`VF2_BUILD_TESTS=ON`, `VF2_WARNINGS_AS_ERRORS=ON` and
`VF2_ROM_DIR=<repo>/roms/vf2`, so it is the primary ROM-backed differential and
strict-test validation path.

Run build and tests from the repository root with:

```powershell
cmake --build build --config Debug --parallel
ctest --test-dir build -C Debug --output-on-failure
```

MSBuild is invoked through `cmake --build` (the VS generator resolves it), so no
separate developer prompt is required. Python recovery/analysis tooling also runs
natively under `python tools/python/...`; adapt the documented `build/...` binary
paths to the native `build\Debug\...` output layout (for example
`build\Debug\vf2probe.exe`).

A WSL2 `build-wsl/` directory remains available as a secondary cross-check and is
kept in sync when the user requests it, but it is not the default path. Behavior
must stay aligned with the documented strict build/test gate and the ROM-backed
commands described throughout this handoff.

## Build and test gate

Normal strict build:

```sh
cmake -S . -B build \
  -DVF2_BUILD_TESTS=ON \
  -DVF2_WARNINGS_AS_ERRORS=ON
cmake --build build --parallel
ctest --test-dir build --output-on-failure
```

With a legally obtained supported ROM set:

```sh
cmake -S . -B build \
  -DVF2_BUILD_TESTS=ON \
  -DVF2_WARNINGS_AS_ERRORS=ON \
  -DVF2_ROM_DIR=/path/to/vf2
cmake --build build --parallel
ctest --test-dir build --output-on-failure
```

Sanitizer gate for changes touching runtime, executor, snapshots, memory or
hardware modeling:

```sh
cmake -S . -B build-san \
  -DVF2_BUILD_TESTS=ON \
  -DVF2_WARNINGS_AS_ERRORS=ON \
  -DVF2_ENABLE_SANITIZERS=ON \
  -DVF2_ROM_DIR=/path/to/vf2
cmake --build build-san --parallel
ctest --test-dir build-san --output-on-failure
```

A change is not finished merely because it compiles. Run the most specific
ROM-backed differential path that exercises the new recovery.

## Current handoff status

At the time this handoff was written, `master` already contains:

- a substantial native boot/runtime/scheduler corridor;
- repeated native dispatch through the fifth and sixth gameplay entries;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [celsowm/vf2-decomp](https://github.com/celsowm/vf2-decomp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
