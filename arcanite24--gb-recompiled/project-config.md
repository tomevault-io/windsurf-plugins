---
trigger: always_on
description: GB Recompiled turns Game Boy ROMs into generated C and links that code with a shared runtime.
---

# Working on GB Recompiled

## Project and documentation map

GB Recompiled turns Game Boy ROMs into generated C and links that code with a shared runtime.

The main areas are:

- `recompiler/`: ROM loading, decoding, analysis, IR, code generation, and the `gbrecomp` CLI
- `runtime/`: CPU fallback, mappers, PPU, DMA, timers, audio, persistence, SDL, and ImGui
- `tests/`: repository-owned synthetic and unit regression tests
- `tools/`: accuracy, trace, differential-log, frame, audio, and benchmark helpers
- `roms/`: local test ROMs; do not commit copyrighted commercial ROMs
- `output/`: ad hoc generated projects and launcher builds
- `logs/`: durable repro captures, state dumps, frame captures, and benchmark results

Documentation roles:

- `README.md`: short public introduction and successful first build
- `RUNTIME.md`: generated-executable controls and diagnostics
- `ACCURACY.md`: current external-ROM test snapshot
- `GBC.md`: CGB implementation and validation status
- `ANDROID.md`: Android generation and APK workflow
- `GROUND_TRUTH_WORKFLOW.md`: trace-assisted coverage, including its limits
- `NATIVE_PATCHES.md`: exact-ROM native replacement manifest and callback ABI
- `PORT_MODULES.md`: exact-ROM read-only port/frontend module ABI
- `TODO.md`: prioritized live backlog
- `docs/CODE_IMPROVEMENT_AUDIT_2026-07-12.md`: audit evidence and completed P0 remediation
- `docs/RECOMPILER_CORRECTNESS_AUDIT_PLAN.md`: current correctness roadmap
- `docs/NL0_POST_WIN_PERFORMANCE_TRUTH_2026-07-14.md`: three-game attribution, estimator, build footprint, and NL-0 decision
- `docs/NL1_ARITHMETIC_TIMER_RESULT_2026-07-14.md`: retained timer optimization and scalar-oracle evidence
- `docs/NL2_LAZY_PPU_RESULT_2026-07-14.md`: rejected lazy-PPU experiment and measured limits
- `docs/APU_EVENT_BATCHING_RESULT_2026-07-14.md`: retained lazy APU scheduling and deterministic PCM evidence
- `docs/NL4_GENERATED_BUILD_RESULT_2026-07-14.md`: retained generated chunking and compiler-memory evidence
- `docs/NL3_POST_SCHEDULER_REPROFILE_2026-07-14.md`: post-scheduler compiled-region eligibility decision
- `docs/NL5_NATIVE_PATCH_SDK_RESULT_2026-07-14.md`: native replacement tracer-bullet design and gates
- `docs/NATIVE_RECOMPILATION_STRATEGY_2026-07-14.md`: active performance and AOT execution strategy

## Hardware references

For hardware behavior, consult `tech_docs/pan_docs.md` first. Use the local `SameBoy/` source as the second reference when Pan Docs is ambiguous or a proven implementation is needed.

This is mandatory for timing-sensitive work, especially:

- CGB registers and DMG-on-CGB behavior
- mapper address resolution
- PPU modes, STAT, palettes, and bus visibility
- OAM DMA and HDMA
- timers, interrupts, HALT/STOP, and speed switching

## Build and test standards

Always use CMake with Ninja. The root project requires CMake 3.20 or newer.

```bash
cmake -G Ninja -B build . -DBUILD_TESTS=ON
ninja -C build
ctest --test-dir build --output-on-failure
```

Do not trust a relocated CMake cache. If a build directory points to a different checkout, configure a fresh build directory instead of treating its failure as a source regression.

The repository-owned CTest suite is the default fast gate. It covers analyzer state, mapper resolution, 9-bit MBC5 banks, bus phases, HALT/OAM-bug CPU transitions, PPU/DMA timing primitives, state/test-runner protocols, multi-ROM isolation, and release relocation.

Native replacement changes must additionally run `native_patch_sdk_end_to_end`.
Keep `gb_native_call_original()` as a scheduling disposition: generated bodies
can return at safepoints before the guest function returns, so post hooks must
remain tied to the captured guest call frame rather than the native C stack.

## Required validation for generated behavior

Changes to analyzer logic, code generation, generated CMake, runtime behavior, or runtime headers must validate both the root build and a freshly generated project.

```bash
ninja -C build

./build/bin/gbrecomp <test-rom> -o output/<test-project>
cmake -G Ninja -S output/<test-project> -B output/<test-project>/build
ninja -C output/<test-project>/build
./output/<test-project>/build/<test-project> --headless --limit-frames 120
```

Use a legal local ROM for the smoke run. The synthetic CTest fixtures remain the portable CI gate. When a generated build appears to ignore a runtime change, regenerate and explicitly rebuild it; stale generated trees are a recurring source of false conclusions.

Apply focused external tests according to the subsystem:

```bash
python3 tools/run_tests.py --filter ppu
python3 tools/run_tests.py --filter oam_dma
python3 tools/run_tests.py --filter timer
python3 tools/run_tests.py --filter misc
```

The accuracy runner is fail-closed: an empty selection, timeout, build error, missing state dump, or incomplete result is a failure. Use `--rebuild` when verifying changes that could invalidate generated output.

Blargg `oam_bug` uses its documented external-RAM protocol rather than serial output: `$A001-$A003` must contain `DE B0 61`, `$A000=00` passes, `$A000=80` is incomplete, and any other signed status fails. A missing or malformed signature never passes.

## Accuracy workflow


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arcanite24/gb-recompiled](https://github.com/arcanite24/gb-recompiled) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-04 -->
