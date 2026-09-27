---
trigger: always_on
description: Run every project command from the repository root. Project scripts reject a
---

# Copilot instructions for memories-decomp

## Build, validation, and repository checks

Run every project command from the repository root. Project scripts reject a
different working directory and paths that escape the repository.

Keep all work inside this repository root. Put temporary sources, probes,
build artifacts, logs, diffs, and other scratch work under `tmp/`; do not use
paths elsewhere on the machine. User-provided reference material for
decompilation and naming work is placed under `tmp/references/`.

The build requires a legally obtained North American executable at
`game/SLUS_014.11`. Full disc checks also require every ignored input listed in
`config/slus_01411/files.sha256`.

```sh
# First-time local setup
make verify-target
make tools
make check-tools

# Clean exact-match gate used by CI
make clean
MAKEFLAGS=-j"$(nproc)" make match

# Full local repository audit; requires all retail inputs and a clean worktree
MAKEFLAGS=-j"$(nproc)" make audit
```

There is no unit-test runner or conventional lint target. Validation is based
on reproducing the complete executable byte-for-byte plus repository policy
checks:

```sh
make basic-types          # every C file uses src/types.h
make external-attempts    # validate external/refinement attempt records
make check-global-usage   # optional generated global-usage consistency check
make check-progress       # optional README progress consistency check
```

For repeated work on one function or translation unit, seed the incremental
object cache after a clean match, then use the incremental exact-match target.
It rebuilds changed objects but still relinks and hashes the entire executable:

```sh
MAKEFLAGS=-j"$(nproc)" make match
tools/environments/python/bin/python \
  tools/project/build_incremental.py --seed-existing

# After editing one C source
MAKEFLAGS=-j"$(nproc)" make match-incremental

# Final acceptance must still use the clean path
MAKEFLAGS=-j"$(nproc)" make match
```

Use `make match-incremental` for intermediate C/header edits in the worker loop.
Do not run `make split` or `make clean` between those iterations: warm builds
validate and reuse the split output as well as unchanged objects. Seed only
immediately after an unchanged clean match. Keep the final clean acceptance
above; incremental success does not replace it.

Keep performance probes local beneath `tmp/`; do not upload build or performance
artifacts. Prioritize worker edit/build iterations over heavyweight CI benchmarks.

Use `MAKEFLAGS=-j"$(nproc)"` so Make follows the host's available logical
CPUs. Set a lower job count explicitly on memory-constrained systems.
Matching, attempt-ledger updates, and integration remain sequential.

## Architecture

- `game/` contains ignored, immutable retail inputs. Nothing from this
  directory may be committed.
- `config/slus_01411/functions.csv` is the authoritative resident-function
  inventory: address, size, name, status, ownership, and durable notes.
  `matching_c.json` maps each matching function to its source and named
  compiler profile; `compiler_profiles.json` defines the exact GCC/MASPSX
  pipeline. Symbol, relocation, image-map, and disc metadata live beside them.
- `src/game/` contains tracked matching C. A source may define one function or
  a contiguous group of functions. `src/types.h` is the sole definition point
  for the fixed-width primitive aliases.
- `make split` runs `tools/project/generate_build_config.py`, which combines
  the static `config/slus_01411/split.yaml` template with `matching_c.json`.
  Splat then writes generated assembly, data, assets, symbols, and diagnostics
  beneath `tmp/splat/`; generated build manifests go beneath `tmp/generated/`.
  These generated files are not source and must not be edited or committed.
- `tools/project/build_baseline.py` assembles unmatched generated MIPS,
  compiles matching C with its per-function profile, normalizes compiler
  assembly through the matching MASPSX version, converts binary regions to
  objects, and links everything in executable order.
- `config/slus_01411/split.yaml` fixes the original image layout and directs
  Splat to generate `tmp/splat/slus_01411.ld`;
  `config/slus_01411/link_symbols.ld` supplies the extra layout symbols that
  generated script does not define.
  `tools/project/build_baseline.py` links through those generated/tracked
  scripts, and `tools/project/match.py` accepts the build only when
  `tmp/project-build/SLUS_014.11` has the retail target hash.
- `notes/` holds durable research and workflow documentation. Progress and
  global-usage data are generated from tracked metadata, not maintained as
  independent sources of truth.
- `tmp/references/` contains user-provided comparison sources, exports, and
  other evidence used during decompilation. Treat references as evidence
  rather than authoritative declarations or build configuration, and keep all
  derived reference work under `tmp/`.

## Matching and metadata conventions

- Functional equivalence is insufficient. Preserve instruction bytes,
  relocations, object order, section placement, executable size, and final
  SHA-256. Source that looks unusual may intentionally control old-GCC
  register allocation or scheduling; do not simplify it without an exact
  match.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Unchiga/Yu-Gi-Oh-Forbidden-Memories-Recompiled](https://github.com/Unchiga/Yu-Gi-Oh-Forbidden-Memories-Recompiled) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
