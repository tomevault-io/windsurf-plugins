---
trigger: always_on
description: ARM64-to-x86_64 binary translation for Android, built on AOSP's Berberis NativeBridge framework. Digitalis enables ARM64-only Android apps (specifically Vulkan apps) to run on x86_64 Android emulators by translating ARM64 instructions to native x86_64 machine code at runtime.
---

# Digitalis

ARM64-to-x86_64 binary translation for Android, built on AOSP's Berberis NativeBridge framework. Digitalis enables ARM64-only Android apps (specifically Vulkan apps) to run on x86_64 Android emulators by translating ARM64 instructions to native x86_64 machine code at runtime.

## What This Is

This is an **AOSP 16** (Android Open Source Project, API 36) source tree with modifications to the Berberis binary translator to support ARM64-to-x86_64 translation. Berberis originally supported only RISC-V-to-x86_64; Digitalis adds the ARM64 backend.

`sample/hellodigitalis/` holds 150 ARM64-only sample app modules — the integration test suite. They span ndk-samples ports, proxy-lib smoke tests (including NDK proxy-surface probes: AHardwareBuffer, AImageDecoder, AMediaDataSource/Muxer, ADPF, ASharedMemory, system fonts, OpenMAX AL, OpenSL ES), ARM-extension probes, UI engines (Qt 6, React Native, Lynx), and third-party native libraries; `ls` the directory for the current set. All 150 are exercised by `test-samples.sh` (`hello-realm` is a standalone pinned-toolchain Gradle project; the suite auto-builds it via its `build-apk.sh` and verifies it by launch).

## Architecture

```
ARM64 APK (arm64-v8a only)
  -> Android Framework (x86_64 host)
  -> NativeBridge (libberberis_arm64.so)
  -> Guest Loader (TinyLoader) -> ARM64 linker64 + libraries
  -> ARM64 App Code
  -> Proxy Libraries (libvulkan, libc, libm, etc.) -> host APIs
  -> Host GPU (GFXStream VkDecoder for Vulkan)
```

Three execution tiers, all reachable for the same guest code:
- **Lite translator** (first gear, the default entry point): single-pass JIT, ARM64 region -> x86_64, ~98% of instructions, cached.
- **Heavy optimizer** (second gear): re-translates hot regions with SSA MachineIR, global register allocation and loop optimization. Bailing here is correct-but-slow, never a crash.
- **Interpreter**: per-instruction fallback for syscalls, complex SIMD, and anything the JITs don't cover.

## Key Directories

All paths relative to repo root.

| Directory | What It Contains |
|-----------|-----------------|
| `frameworks/libs/binary_translation/` | Berberis core — the binary translator |
| `frameworks/libs/binary_translation/lite_translator/arm64_to_x86_64/` | JIT compiler (ARM64 -> x86_64 native code) |
| `frameworks/libs/binary_translation/heavy_optimizer/arm64/` | Second-gear optimizing JIT (SSA MachineIR frontend) |
| `frameworks/libs/binary_translation/backend/x86_64/` | Shared MachineIR backend — register allocation, guest-context and loop optimizers |
| `frameworks/libs/binary_translation/interpreter/arm64/` | Instruction-by-instruction interpreter fallback |
| `frameworks/libs/binary_translation/decoder/include/berberis/decoder/arm64/` | ARM64 instruction decoder (`decoder.h`, `semantics_player.h`) |
| `frameworks/libs/binary_translation/runtime/arm64/` | Translation cache, dispatch, region management |
| `frameworks/libs/binary_translation/kernel_api/` | Linux syscall emulation (including `arm64/syscall_emulation.cc`) |
| `frameworks/libs/binary_translation/guest_loader/` | ELF loading, NativeBridge init |
| `frameworks/libs/binary_translation/android_api/` | Proxy libraries (libc, libm, libvulkan — forward API calls from guest to host) |
| `frameworks/libs/binary_translation/prebuilt/` | Prebuilt configs including `ld.config.arm64.txt` |
| `device/generic/goldfish/` | Emulator (goldfish) product definitions |
| `device/generic/goldfish/64bitonly/product/sdk_phone64_x86_64_digitalis.mk` | Digitalis product config |
| `sample/hellodigitalis/` | The 150 sample app modules |

## Build

```bash
source build/envsetup.sh
lunch sdk_phone64_x86_64_digitalis-trunk_staging-userdebug
m
```

Or `source digitalis/scripts/lunch-digitalis.sh`. Emulator: `emulator -memory 4096 -writable-system -qemu -cpu host &` (never `-no-window` when debugging an APK).

```bash
# Host unit tests
out/host/linux-x86/nativetest64/berberis_arm64_host_tests/berberis_arm64_host_tests --gtest_filter='Arm64*'
# Samples on the emulator — also --screenshots and --update-references; pass a module name to narrow
.claude/scripts/test-samples.sh
# Sample APKs
cd sample/hellodigitalis && ./gradlew assembleDebug
# Performance: sweep the tiers, then compare against docs/benchmark-results.md
digitalis/scripts/run-benchmarks.sh --repeats 2
digitalis/scripts/summarize-benchmarks.py digitalis/out/bench/<stamp>.ndjson \
    --markdown digitalis/docs/benchmark-results.md
```

Methodology, traps, and how to read a noisy row: `digitalis/docs/benchmarking.md`.

**Upstream ARM64 regression check, before committing.** Berberis lives in shared paths, so translator, makefile and proxy-library edits can leak into the native ARM64 image. Verify `lunch sdk_phone64_arm64_minigbm-trunk_staging-userdebug && m` still builds clean, then `lunch` back to the Digitalis target before resuming x86_64 work. Note `m` on the arm64 product builds no berberis at all — check `m libberberis_riscv64` too.

## Prebuilt-APK regression (sample/prebuilts/)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DigitalisX64/digitalis](https://github.com/DigitalisX64/digitalis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
