---
trigger: always_on
description: FastCov is a Ruby gem with a native C extension that tracks which source files are executed during test runs. It's built for test impact analysis — figuring out which tests need to re-run when code changes. It hooks directly into the Ruby VM's event system rather than using Ruby's built-in `Coverage` module, which makes it significantly faster.
---

# FastCov Development Guide

## What is this project?

FastCov is a Ruby gem with a native C extension that tracks which source files are executed during test runs. It's built for test impact analysis — figuring out which tests need to re-run when code changes. It hooks directly into the Ruby VM's event system rather than using Ruby's built-in `Coverage` module, which makes it significantly faster.

## Quick reference

```sh
bundle exec rake compile   # compile the C extension (required before tests)
bundle exec rake spec      # compile + run tests (--fail-fast)
bin/rspec                  # run tests via binstub
bin/benchmark              # run performance benchmarks
bin/benchmark --baseline   # save benchmark results as baseline for comparison
ITERATIONS=5000 bin/benchmark  # override iteration count
```

**Testing locally:** Always test in both Ruby 4.0 and Ruby 3.4.

**Before starting new changes:** Ask the user if they want to save a baseline benchmark first (`bin/benchmark --baseline`). This allows comparing performance before and after the changes.

## Project structure

```
ext/fast_cov/
  fast_cov.c          # Core C extension (~610 lines). All the performance-critical code.
  fast_cov_utils.c    # Shared C utilities: path filtering, string helpers, const resolution.
  fast_cov.h          # Header shared between the two C files.
  extconf.rb          # Build config (mkmf). Generates Makefile for compilation.

lib/fast_cov/
  fast_cov.rb         # Entry point. Loads C extension + Ruby modules.
  version.rb          # VERSION constant.
  configuration.rb    # FastCov.configure block API. Currently a stub for future options.
  cache.rb            # FastCov::Cache module. C defines .data, .data=, .clear.
  trackers/
    abstract_tracker.rb    # Base class for Ruby trackers (FileTracker, FactoryBotTracker).
    coverage_tracker.rb    # Wraps the C extension for line/allocation/constant tracking.
    file_tracker.rb        # Tracks File.read/File.open calls.
    factory_bot_tracker.rb # Tracks FactoryBot factory definition files.
  benchmark/
    runner.rb          # Benchmark harness: measurement, baseline comparison, reporting.
    scenarios.rb       # The benchmark scenario definitions.

spec/
  lib/fast_cov/coverage/   # Integration tests organized by feature.
  fixtures/                 # Calculator, app models, vendor — test fixture code.
  support/                  # Shared contexts, file helpers.

bin/
  benchmark    # Run benchmarks, compare against baseline.
  console      # IRB with FastCov loaded.
  rspec        # RSpec binstub.
```

## How the C extension works

The C extension defines `FastCov::Coverage` (a Ruby class) and `FastCov::Cache` (a Ruby module). Everything performance-sensitive lives in C.

### Two coverage mechanisms

**1. Line coverage** — The core feature. Hooks `RUBY_EVENT_LINE` which fires every time the Ruby VM executes a new line. The callback (`on_line_event`) records the source file path. It uses a pointer-caching optimization: `rb_sourcefile()` returns a `const char*` whose address doesn't change for the same file, so we compare pointers (a single integer comparison) instead of strings to skip files we've already seen.

**2. Allocation tracing** — Optional. Hooks `RUBY_INTERNAL_EVENT_NEWOBJ` which fires on every object allocation. We only care about `T_OBJECT` and `T_STRUCT` types (regular classes and structs). During `stop`, we iterate every class that was instantiated, walk its full ancestor chain (`rb_mod_ancestors`), and resolve each ancestor to its source file via `Object.const_source_location`. This catches classes that have no executable methods (empty models, structs, Data objects).

### In-memory cache

`Object.const_source_location` results are cached in a process-level Ruby Hash (`fast_cov_cache_hash`) and shared across all `FastCov::Coverage` instances. `FastCov::Cache.clear` resets it.

### GC integration

The C struct uses Ruby's TypedData API with proper `mark` and `free` callbacks. `rb_gc_mark` (non-movable, pins objects) is used for all VALUE fields — on Ruby 3.4+, `rb_gc_mark_movable` with compaction causes crashes, so we pin objects instead. The `klasses_table` (an `st_table`) stores raw VALUE pointers as keys, which are also marked via `rb_gc_mark`.

**Important for C code:** When storing raw `const char*` pointers from Ruby strings, freeze the string first (`rb_str_freeze`) to prevent GC compaction from moving it. Alternatively, re-read the pointer immediately before use with no intervening Ruby calls.

### Utils module

`FastCov::Utils` provides C-implemented utility functions:

- `path_within?(path, directory)` — Returns true if `path` is within `directory`. Correctly handles trailing slashes and sibling directories with longer names (e.g., `/a/b/c` does NOT match `/a/b/cd`).
- `relativize_paths(hash, root)` — Mutates `hash` in place, converting absolute path keys to relative paths from `root`. Called automatically by `FastCov.stop`.

## Benchmark scenarios


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Gusto/fast_cov](https://github.com/Gusto/fast_cov) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
