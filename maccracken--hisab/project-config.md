---
trigger: always_on
description: **Hisab** (Arabic: calculation/mathematics) — Higher math: linear algebra,
---

# Hisab — Claude Code Instructions

## Project Identity

**Hisab** (Arabic: calculation/mathematics) — Higher math: linear algebra,
geometry, calculus, numerical methods, spatial structures, Lie groups,
differential geometry, symbolic algebra.

- **Type**: Cyrius library + CLI (math toolkit)
- **License**: GPL-3.0-only
- **Language**: Cyrius (sovereign systems language, compiled by cycc)
- **Toolchain**: Cyrius 6.3.11 (`cyrius.cyml: cyrius = "6.3.11"`)
- **Version**: SemVer, version file at `VERSION` (manifest pulls via `${file:VERSION}`)
- **Status**: 2.6.7 — compiles cleanly under cycc 6.3.11. Library source lives in `src/` (smoke `main.cyr` + 34 math modules); `lib/` is vendored stdlib + deps only. CLI smoke binary builds; **full 34-module distlib bundle** (~553 KB / 16,878 lines, fits cycc 6.3.11's 1 MB input_buf with ample headroom) ships at `dist/hisab.cyr` and is consumer-tested end-to-end. Library validated via tests (957/957). The 2.3.x, 2.4.x, 2.5.x, and 2.6.x (differential-geometry depth) arcs are all complete. The stdlib transcendentals live in the `ganita` module (introduced at the 6.2.11 bump, which also subsumed `matrix`/`linalg`); the 6.3.11 bump (2.6.7) was infrastructure-only — see CHANGELOG 2.6.7.

## Consumers

impetus (physics), kiran (engine), joshua (simulation), aethersafha (compositor)

## Quick Start

```bash
cyrius deps                              # resolve deps into lib/
cyrius build src/main.cyr build/hisab    # build CLI
cyrius test tests/hisab.tcyr             # run a test suite
cyrius bench tests/hisab.bcyr            # run benchmarks
```

## Dependencies

- **Cyrius stdlib** — `syscalls`, `string`, `alloc`, `str`, `fmt`, `vec`,
  `io`, `args`, `assert`, `math`, `ganita`, `tagged`, `fnptr`,
  `bench`, `callback` (ships with Cyrius >= 6.2.11). `ganita` is the 6.2.x
  math umbrella — transcendentals (acos/asin/atan2/pow/sinh/cosh/tanh +
  inverses) plus the full `matrix`/`linalg` API; do **not** also list `matrix`
  or `linalg` (duplicate-definition collisions). `math` stays for the inclusive
  comparisons, clamp/lerp/min/max/sign and the exp/ln polyfills.
- **sakshi** 2.4.2 — structured logging (first-party)

No external deps. No FFI. No libc. All first-party, pinned in
`cyrius.cyml` and SHA-locked in `cyrius.lock`.

## Layout

```
src/main.cyr         — CLI smoke binary (prints version, exits — does NOT
                       include the library; library coverage is in tests/)
src/*.cyr            — the 34 math modules (the library source). Self-contained
                       (no `include` lines); stdlib resolves via [deps] stdlib
lib/                 — vendored stdlib + first-party deps ONLY (managed by
                       `cyrius deps`) — no project source here
dist/hisab.cyr       — full 34-module distlib bundle (~535 KB / 16,446 lines),
                       regenerated via `cyrius distlib`. Consumers pull this
                       single file via [deps.hisab] modules = ["dist/hisab.cyr"]
examples/            — small demos (basic_math.cyr)
tests/
  hisab.tcyr         — primary assertion suite
  foundation.tcyr    — vec/quat/mat foundations
  modules.tcyr       — per-module coverage
  edge_cases.tcyr    — degenerate inputs, boundary values
  hisab.bcyr         — benchmark harness
  hisab.fcyr         — fuzz harness
docs/
  architecture/      — module map, math reference
  development/       — roadmap, threat model, dep watch, port audit
  guides/            — usage, testing
  audit/             — dated audit reports
scripts/
  bench-history.sh   — append benchmark CSV row
  version-bump.sh    — bump VERSION + CHANGELOG header
cyrius.cyml          — package manifest (toolchain pin, [deps])
cyrius.lock          — SHA256 lockfile (after first `cyrius deps`)
VERSION              — single source of truth for version
```

## Development Process

### P(-1): Scaffold Hardening (before any new features)

0. Read roadmap, CHANGELOG, and open issues — know what was intended before auditing what was built
1. Cleanliness sweep: `cyrius lint`, `cyrius fmt --check`, `cyrius vet src/main.cyr`
2. Test + benchmark sweep of existing code: `cyrius test tests/*.tcyr`, fuzz harnesses pass
3. Baseline benchmarks: `./scripts/bench-history.sh`
4. Internal deep review — gaps, optimizations, security, logging/errors, docs
5. External research — domain completeness, missing capabilities, best practices, world-class accuracy
6. Cleanliness re-check — must be clean after review
7. Additional tests/benchmarks from findings
8. Post-review benchmarks — prove the wins
9. Repeat if heavy

### Work Loop (continuous)

1. Work phase — new features, roadmap items, bug fixes
2. Cleanliness check: `cyrius lint`, `cyrius fmt --check`, `cyrius vet src/main.cyr`
3. Test + benchmark additions for new code
4. Run benchmarks: `./scripts/bench-history.sh`
5. Internal review — performance, memory, security, throughput, correctness
6. Cleanliness re-check — must be clean after audit
7. Deeper tests/benchmarks from audit observations
8. Run benchmarks again — prove the wins
9. If audit heavy → return to step 5
10. Documentation — update CHANGELOG, roadmap, docs
11. Version check — VERSION matches CHANGELOG header (cyrius.cyml auto-syncs via `${file:VERSION}`)
12. Return to step 1


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MacCracken/hisab](https://github.com/MacCracken/hisab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
