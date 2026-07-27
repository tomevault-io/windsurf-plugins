---
trigger: always_on
description: Quill is a C++17 asynchronous low-latency logging library.
---

# CLAUDE.md

## Project Summary

Quill is a C++17 asynchronous low-latency logging library.

Main design goals:

- keep frontend logging overhead as low as possible,
- move formatting and I/O to a backend worker thread,
- preserve timestamp ordering across threads,
- provide a practical API without compromising hot-path performance.

The project is performance-sensitive. Many design choices prioritize low frontend latency over extra convenience or
extra validation at every call site.

## Repository Shape

Important directories:

- `include/quill/`: public headers and most implementation.
- `docs/`: Sphinx docs and API guidance.
- `test/unit_tests/`: focused component tests.
- `test/integration_tests/`: end-to-end behavioral tests.
- `examples/`: usage examples.
- `benchmarks/`: compile-time and runtime benchmarks.

## Core Architecture

The library has two main parts:

1. Frontend

- User threads log via macros or macro-free APIs.
- Each thread writes into its own thread-local SPSC queue.
- The hot path serializes arguments and stores metadata references.

2. Backend

- A single backend worker thread polls frontend queues.
- It orders events by timestamp.
- It performs formatting and sink I/O.

When reviewing or changing code, always ask:

- Is this on the frontend hot path or backend path?
- Does this add cost to every log statement?
- Does this move work from backend to frontend?
- Does this change a documented user contract?
- Before fixing a "bug", trace all call sites — the invariant may be maintained by callers rather than the function
  itself.

## Important Contracts

- Single backend worker thread.
- `Logger` objects are thread-safe for logging.
- Logger configuration is effectively immutable after creation; recreate a logger instead of mutating it in place.
- `Frontend::remove_logger_blocking()` must only be used while the backend is running.
- Once a logger is removed, that `Logger*` must not be used again.
- Removing the same logger from multiple threads is unsupported.
- The project supports both exception and no-exception builds.
- Ideally used as a static library, but some users integrate it through shared-library / `.so` boundaries. Keep
  shared-library pitfalls in mind as well, especially singleton lifetime, duplicated state across DSOs, symbol
  visibility, and shutdown behavior.
- If you change public behavior, keep headers, docs, tests, and changelog aligned.

## Layering

Respect the library layering:

- Avoid introducing dependencies from lower-level code to higher-level code.
- Flag dependency inversions in reviews.
- `quill/core` is part of the frontend-facing surface and should remain self-contained: it may depend on `quill/core`
  and `quill/bundled`, but not on other layers.
- For normal user-side logging, the intended lightweight include surface is `quill/Logger.h` and `quill/LogMacros.h`. Be
  very careful with every include reachable from those headers, including standard library headers. Any extra `#include`
  there propagates into user translation units and increases compile-time cost and header pollution.

## Updating Bundled libfmt

The bundled fmt copy under `include/quill/bundled/fmt/` is vendored source with Quill-local patches. Treat each upgrade
as two separate jobs: first identify the upstream delta, then port the Quill-specific changes.

- Identify the current bundled fmt version from `FMTQUILL_VERSION` in `include/quill/bundled/fmt/base.h`; the value uses
  fmt's `major * 10000 + minor * 100 + patch` encoding. Pick the target version from the official fmt releases at
  https://github.com/fmtlib/fmt/releases.
- Download and unpack the official current and target fmt release archives outside `include/quill/bundled/fmt/`. Compare
  the official current release with Quill's bundled tree to find local patches, then compare the target release with the
  official current release to understand upstream changes.
- Replace the vendored fmt files from the official target release, then run:
  `python3 scripts/rename_libfmt.py include/quill/bundled/fmt FMTQUILL fmtquill`
- Preserve upstream formatting in bundled fmt files. Do not run clang-format or broad formatting tools over vendored fmt;
  port Quill patches as the smallest necessary edits.
- Reapply and re-check Quill-local changes that still apply to the target fmt version:
  - In `base.h`, force header-only mode with `FMTQUILL_HEADER_ONLY`.
  - In `base.h`, keep the end-of-file `format.h` include removed or disabled when it is only included because
    `FMTQUILL_HEADER_ONLY` is set.
  - Keep the buffer `append` fast path that uses `memcpy(ptr_ + size_, begin, count * sizeof(T))` when
    `std::is_same<T, U>::value`, with the element-wise copy fallback when the types differ.
  - In `chrono.h`, keep the `std::micro` suffix as `"us"` unless upstream already matches that behavior.
  - In `format.h`, retain only the warning suppressions still needed by supported compilers, such as GCC/Clang
    `-Wfloat-equal` and GCC `-Wstringop-overflow` false positives.
  - Keep `is_fast_float<T>::value` in places where supported compilers require the type-trait form; if upstream changes
    this code, confirm with a targeted compile instead of applying a blind search-and-replace.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [odygrd/quill](https://github.com/odygrd/quill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
