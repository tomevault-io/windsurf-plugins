---
trigger: always_on
description: Where things live and why they are the way they are. Deliberately not a
---

# AGENT.md — orientation for coding agents

Where things live and why they are the way they are. Deliberately not a
substitute for reading the code: when this file describes a mechanism it says
*why it exists*, not how it is spelled. Grep for a named symbol rather than
trusting a description that looks stale.

`README.md` is the *user's* introduction — what the library does, why, and how
to call it. It is also where the design rationale now lives; this file is the
part that only matters once you are editing the code.

## 1. What this project is

A from-scratch, **async-signal-safe backtrace library for profilers** ("aw" =
the prefix on all public symbols). Unwinds from `.eh_frame` / `.eh_frame_hdr`
CFI parsed by hand, with no libunwind/libgcc dependency.

The contract:

* Never crash, never deadlock, never allocate on the sampling path. An
  occasional *wrong* backtrace is acceptable; crashing is not.
* Only PC, SP and FP are unwound. Full DWARF register-state unwinding is
  deliberately absent — the bet is that essentially all real frames are either
  fixed-offset-from-SP or RBP-framed.
* Anything outside that model is an explicit, reported error, never a silent
  guess.
* Modern Linux, glibc 2.35+ (`_dl_find_object`). x86-64 is where the testing
  is; aarch64 builds and passes the basic tests but has no comparer and no
  `GuessUnwindInfo`. riscv and 32-bit eventually.

## 2. Build and test

**Two build systems that build different things.** Neither is a superset.

### Bazel — the library and the test suite

bzlmod, one top-level package, module `aw-backtrace`.

```
bazel test ...:all               # the normal loop: 12 tests
bazel test -c opt ...:all        # NDEBUG: drops assert(), keeps CHECK()
./test-all-cfg.rb                # gcc/clang x dbg/opt sweep, all four green
```

Eleven tests: nine in the top package plus `//perf-convert`'s two. `:all` instead
of `...:all` skips perf-convert. `v/mini-x86-int` is in `.bazelignore`, so
`@mini-x86-int//:sim_stepper_test` runs only by explicit label.

Traps worth knowing before you fight the build:

* **`-std=c++20 -fno-exceptions -fno-rtti` live in `BUILD.bazel`'s `CXXOPTS`,
  per target, on purpose.** A project that consumes this module never reads our
  `.bazelrc`, so anything we need has to travel with the targets. `cxxopts`
  rather than `copts` so it stays off the C compiles (`aw-addrcheck.c`). The
  public header is deliberately standard-agnostic — it compiles as C99 and as
  C++11 and up — so a consumer on an older standard can still include it.
* **All internal headers are one target, `:internal`** (a `glob(["*.h"])`).
  Every target that compiles C++ deps `:internal` and `#include`s whatever it
  needs — there are no per-header libraries and no dependency-layering to keep
  in sync. `:internal` carries no compile options (headers only), so it leaks
  nothing into a consumer; the `.cc` implementations still have their own
  targets. Two things stay out of the glob: `aw-addrcheck.h` (its own `:aw-
  addrcheck` C target, `exclude`d) and the public header (`:aw-backtrace-hdr`,
  shipped under `aw-backtrace/` via `strip_include_prefix`). `:internal` also
  carries `linkopts = ["-latomic"]`, so every C++ target gets it.
* **`.bazelrc` is for *this* workspace only.** It forces `-std=c++20` across the
  whole graph so abseil matches what `//perf-convert` compiles against. It does
  nothing for anyone depending on us.
* **`.bazelversion` pins 9.2.0, load-bearing.** `dev_dependency` belongs on the
  `bazel_dep` line; 9.2.0 rejects it on `local_path_override` and every `bazel`
  invocation dies.
* **GNU ld intermittently segfaults linking this package.** Worked around with
  `features = ["-supports_start_end_lib"]` on the targets that hit it — a
  toolchain interaction, not a code bug. Copy the line onto any new binary
  target that starts crashing `ld`.
* **`-latomic` is required for gcc *and* clang.** `UnwindInfoCache` uses 16-byte
  atomics; without `-mavx -mcx16` both compilers call libatomic (~2.5ns/step
  under clang). `ATTR_DWORD_ATOMICS`, which used to inline them, is deliberately
  an empty macro — it emitted VEX unconditionally and `SIGILL`ed on pre-AVX.
* **`v/mini-x86-int` is "vendored" only mechanically** — co-developed, same
  author, no upstream. A fix belonging in the stepper goes in the stepper.
* `:aw-backtrace` is the only public target and the only public header
  (`include/aw-backtrace/aw-backtrace.h`, via `strip_include_prefix`).
* `with-exit.h` is header-only: `WithExit::Run`/`Exit` are a thin
  `_setjmp`/`_longjmp` wrapper.

### genbuild.rb / ninja — the LD_PRELOAD comparer

`genbuild.rb` is a self-contained generator (`build!` at the top is the part you
edit). **Edit `genbuild.rb`, never `build.ninja`.**

```
./genbuild.rb ninja                          # regenerate if stale, then build
./genbuild.rb ninja CC=clang CXX=clang++     # ENV overrides; re-execs itself
```

Defaults are `-ggdb3 -O2 -DNDEBUG -Wall -Wextra -march=native`, i.e. an NDEBUG,
machine-specific build — don't copy the `.so` to another machine, it will
`SIGILL`. Five artifacts, none of which Bazel builds: `backtrace-comparer.so`,
`sym_helper_bin`, `test-signal-disable`, and `cjm`/`cjm0`.

### Two things about test configurations


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gperftools/gperftools](https://github.com/gperftools/gperftools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
