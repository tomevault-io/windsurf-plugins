---
trigger: always_on
description: Ultra is an advanced profiler for .NET applications. It produces traces that can be opened in Firefox Profiler and currently supports:
---

# Ultra — Agent Guidance

Ultra is an advanced profiler for .NET applications. It produces traces that can be opened in Firefox Profiler and currently supports:

- **Windows** via ETW/TraceEvent, including kernel, native, CLR, GC, JIT, and managed stacks. Profiling requires an elevated/admin shell.
- **macOS Apple Silicon** via a newer custom in-process sampler that is injected at process startup and emits EventPipe events. This path is intentionally narrower than Windows: ARM64 only, launch-only (no PID attach), no kernel stacks, and currently limited to 1 ms minimum sampling interval.

Paths/commands below are relative to this repository root.

## Orientation

- CLI/global tool: `src/Ultra/`
  - Command-line parsing and user-facing `profile`/`convert` flows live mainly in `Program.cs`.
- Core library: `src/Ultra.Core/`
  - Profiling orchestration: `UltraProfiler*.cs`, `DiagnosticPortSession.cs`.
  - Event processing/conversion: `Parser/`, `UltraConverterToFirefox*.cs`, `FirefoxProfiler.cs`.
  - Native module/symbol support, including Mach-O parsing: `MachO/`, `Model/`.
- Custom sampler: `src/Ultra.Sampler/`
  - Newer NativeAOT sampler used for macOS ARM64.
  - `MacOS/` contains Mach API bindings and the sampler implementation.
  - `ultra_sampler_hook.cpp` is the small injected hook that boots the NativeAOT sampler dylib.
  - `libUltraSampler.dylib` and `libUltraSamplerHook.dylib` are packaged runtime artifacts; do not regenerate or replace them casually.
- Tests: `src/Ultra.Tests/` (MSTest + Verify snapshots under `Verified/`).
- Docs to keep in sync with behavior: `readme.md`, `doc/**/*.md`, and package readmes such as `src/Ultra.Core/readme.md` when relevant.
- macOS research notes: `doc/notes-macos.md`.

## Build & Test

```sh
# from the project root
cd src
dotnet build -c Release
dotnet test -c Release
```

Notes:

- The repo targets .NET 10 (`src/global.json`). If the SDK is not installed locally, report that as an environment blocker rather than changing target frameworks.
- Some profiler behavior is OS/privilege dependent: Windows ETW profiling generally needs admin; macOS sampler validation needs macOS ARM64. On another OS, run the portable tests/build you can and clearly state unverified platform-specific behavior.
- For docs-only guidance changes, a build is usually not necessary; still self-review the diff.

## Architecture Notes

### Windows path

- Windows profiling is ETW-based and uses `Microsoft.Diagnostics.Tracing.TraceEvent`.
- Keep ETW-specific logic separate from EventPipe/macOS logic (`UltraProfilerEtw`, `UltraConverterToFirefoxEtw`, ETL conversion paths).
- Preserve admin/elevation assumptions and user messaging unless changing the actual capability.

### macOS ARM64 path and sampler

The macOS support is new and has several intentional tricks. Be conservative when touching it.

- macOS profiling injects two dylibs at launched-process startup:
  - `libUltraSamplerHook.dylib` runs from a `__DATA,__mod_init_func` initializer.
  - The hook calls `ultra_sampler_start` in the NativeAOT sampler (`libUltraSampler.dylib`).
- The sampler is in-process, not an external attach profiler. Do not add PID attach support in docs/API unless it is implemented end-to-end.
- `UltraSampler.Instance` is a real sampler only on macOS ARM64; other platforms use a no-op sampler.
- Sampling interval is passed through `UltraSamplerConstants.SamplingIntervalEnvironmentVariable` and clamped in `MacOSUltraSampler` to 1..1000 ms. The 1 ms minimum is a macOS sleep/scheduling limitation.
- The hook temporarily changes `TMPDIR` to a `.ultra` subdirectory before starting the sampler. This is a workaround for the runtime diagnostic-port Unix socket naming; keep it in sync with `DiagnosticPortSession.TryFindConnectionAddress`.
- The sampler emits custom EventPipe/EventSource events via `UltraSamplerSource`; the parser consumes them in `Ultra.Core/Parser/`. If you change event IDs, payload order/types, constants, compression format, or provider metadata, update both sides and tests together.
- The sampler registers dyld image callbacks for native module load/unload events. Keep delegate fields alive so callbacks are not collected by the GC.
- Native stack walking currently assumes Apple Silicon frame-pointer conventions (`fp`/`lr`) and is not a general Mach-O unwind implementation. Do not claim x64 support without adding real implementation and tests.
- `task_threads` returns thread send rights and a vm-allocated array; sampler code must deallocate both to avoid leaking inside the profiled process.
- When suspending threads, always resume them on every path. Avoid allocations, blocking work, logging, or exception-heavy code while threads are suspended.
- Skip the sampler thread itself and idle threads when sampling. Thread start/stop events are synthesized from observed Mach threads.
- Repeated stack samples are delta-compressed by shared suffix frame count. Limits such as maximum frames and per-thread compression state must remain compatible with parser expectations.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xoofx/ultra](https://github.com/xoofx/ultra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
