---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Silk is a cooperative fiber scheduler for Linux: per-CPU scheduler threads pinned to cores, io_uring-based async IO, topology-aware work stealing, and fiber synchronization primitives (futures, events, mutexes, futexes, sequencers, multi-locks), plus a utility library (lock-free structures, memory pools, TSC timing, perf counters, a BPF profiler, and gdb/crash-dump tooling).

The design docs under `docs/` are the source of truth for the architecture (`scheduler.md`, `work-stealing.md`, `sync.md`, `coroutines.md`, `tls.md`, `util.md`, `perf.md`).

## Build System

`./bb` is the standard build tool — always use it instead of invoking `cmake`, `ninja`, or `ctest` directly. See `README.md` for the full command reference.

**Always build debug unless running benchmarks or sanitizer runs.**

Build presets: `debug`, `release`, `debug-{sanitizer}`, `release-{sanitizer}`. Build directories live under `build/<preset>/`.

When capturing command output to a file (e.g. tee-ing build or test output for later grepping), write to `build/tmp/` — never to the system `/tmp`. Create the directory with `mkdir -p build/tmp` if needed.

## Profiling and Flamegraphs

Generate a flamegraph by appending `--flamegraph` to a perf-target run, e.g. `bb -b release net-perf --flamegraph`. Always profile a `release` build. Output lands at `build/release/<target>.flamegraph.svg` and the raw folded stacks at `build/release/<target>.flamegraph.folded`.

The profiler is **silk's own BPF profiler** (`bin/profiler --on-cpu --off-cpu --kernel-stacks`), NOT `perf`. It walks stacks via frame pointers, so there is no `perf record --call-graph dwarf` option, and frames can be dropped (FP omission in small release functions, and at syscall boundaries).

**The folded file is a COMBINED on-CPU + off-CPU profile — each line carries TWO trailing numbers, not one:** `<semicolon;stack> <on_cpu_samples> <off_cpu_ns>`. Frame names themselves contain spaces (demangled templates), so parse the last two whitespace tokens, never `$NF` alone. On-CPU lines have `off_cpu_ns == 0`; off-CPU lines have `on_cpu_samples == 0` and end in the `schedule;__schedule;__bpf_trace_sched_switch` tail. On-CPU samples are the real CPU cost; off-CPU is blocked/wait time and is usually dominated (~99%) by idle scheduler-thread park in `parkThread;io_uring_enter2`. `bb` itself sums the two columns when rendering the SVG.

**Frame-loss caveat:** a leaf frame's self-time over-credits the deepest *surviving* frame. The clearest case: `silk::SpinLock::lockSlow` calls `sched_yield` as backoff, but the `lockSlow` frame does not survive the syscall — so its time shows mis-parented as `<caller>;sched_yield`. Attribute `sched_yield` under a lock caller back to `lockSlow`, and treat per-leaf self-time as approximate near syscalls and hot spin loops.

For aggregate rates and latencies without the frame-loss problem, append `--print-counters` instead. It prints the run config, a throughput summary, latency histograms with p50/p90/p99/p999 for the silk scheduling phases (`ready_wait`, `fiber_run`, `suspend_wait`, `cq_wait`), and the named scheduler counters (`FiberSuspended`, `FiberStolen`, `SchedulerThreadParked`/`Waked`, `SchedulerUserTime`/`SystemTime`/`IdleTime`, etc.). Use it to confirm ratios the flamegraph cannot give cleanly — e.g. park/wake balance or suspend-wait latency.

## Layout and namespaces

All code is wrapped in `namespace silk` — never `using namespace`.

Public headers live under `include/silk/<component>/` and are included as `<silk/util/...>` and `<silk/fibers/...>`. Implementation lives under `src/<component>/` alongside the matching `tests/` and `benchmarks/` subdirs; private headers (test fixtures, TU-internal helpers) sit next to their `.cpp` files.

## Performance discipline

**Silk MUST be fast — treat performance as a correctness requirement.**

- No exceptions — all code is `noexcept`; errors are errno returns (see Error handling)
- No std containers or strings in library code — use the structures under `include/silk/util/` (`List`, `IntrusiveQueue`, `BoundedQueue`, `MemoryPool`, `ShardedStack`, `Stack`, `Tree`, `Bitmap`) and `std::string_view` for borrowed ranges; std containers are OK in tests
- No allocations on a hot path — allocate at initialization; steady-state memory comes from pools and preallocated per-CPU state

## Naming and vocabulary

- When referring to a function by name in prose, comments, or docs — **never** append `()`. Write `allocate`, not `allocate()`. The `()` operator means invocation; it is not part of a name.
- Variable names must be fully descriptive — no single-letter abbreviations (`future` not `f`, `params` not `p`, `state` not `s`)
- Member variables use plain camelCase — no trailing underscores (`foo`, not `foo_`)
- Return-code variable is `int r` — never `rc`, `ret`, or `err`
- Reuse the codebase's exact identifier for a concept everywhere — don't coin synonyms or metaphors; if tempted to invent a term, ask first

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ydb-platform/nbs](https://github.com/ydb-platform/nbs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
