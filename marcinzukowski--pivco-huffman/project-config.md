---
trigger: always_on
description: Novel Huffman decoder using SIMD tree-walk partitioning plus a flat-subtree
---

# PIVCO-Huffman

Novel Huffman decoder using SIMD tree-walk partitioning plus a flat-subtree
fast path.  On Apple M4 it beats huf0 (zstd's Huffman) on every tested
distribution by 1.0–5×, including the moderate-entropy bell / zipfian /
english cases that previously lost against huf0 / trad_4s.  Historical
strong wins on skewed distributions (proba80 3.4×, two_sym_eq 4.9×,
uniform 2.4×) are preserved.

The flat-subtree path detects at `build_table` time every maximal
internal node whose subtree is flat with depth D ≥ 2 (all 2^D leaves at
the same relative depth), replaces D levels of bitmap-per-level with a
single N·D-bit packed region in the stream, and decodes via direct
`code_to_sym[local_code]` lookup + scatter — the same mechanism that
already powered the full-tree flat path.

## Build & Test

```sh
# Prerequisites (first time only)
git submodule update --init ext/fse  # FSE entropy coder (required, for PHA)

# Build
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build

# Test
./build/pivco_huffman_tests

# Benchmark (arg = repeats per run, default 100)
./build/pivco_huffman_bench 20      # quick
./build/pivco_huffman_bench 100     # thorough
```

## Architecture

- **Backends**: scalar, NEON (ARM), x86 (SSE4.1 / AVX2), AVX-512 VBMI2 (Intel).  SVE is disabled (svcompact at 128-bit isn't competitive with NEON TBL).
- **Codec framework**: one `pivco_huffman_codec.c` compiled per backend as an OBJECT library, each pulling in `primitives_<backend>.h` (the only file with SIMD intrinsics).  Runtime dispatcher in `src/pivco_huffman.c::resolve_impl` picks the best backend per host.
- **Block size**: 8192 (ARM/AVX-512), 4096 (x86 SSE/AVX2) — auto-detected per backend at compile time
- **Wire format**: see `src/pivco_huffman_wire.h` for the canonical doc.  Per-node record: `[optional K_right:u16 LE][FSE marker:u8][bitmap or FSE payload]`.  Flat subtrees (D ≥ 2) skip the header and emit one N·D-bit packed region.
- **Key data structures**:
  - `compress_tab[256][32]` combined shuffle table (TBL/pshufb partition; per-arch in `pivco_huffman_{neon,x86}_tables.c`)
  - `expand_tab[256][8]` BU tree_merge shuffle table (same files)
  - `table->flat_depth[node]`, `table->flat_offset[node]`,
    `table->flat_code_to_sym[pool]` — per-table flat-subtree dispatch

## Test Hosts (AWS EC2)

```sh
# Sync to remote (cloud code is assumed stale — rsync before every run)
rsync -avz --delete --exclude='build/' --exclude='build-asan/' \
  --exclude='build-release/' --exclude='.git/' --exclude='.claude/' \
  --exclude='.vscode/' --exclude='*.dSYM' --exclude='.venv/' \
  --exclude='ext/oodle' \
  . test-XXX:pivco-huffman/
# NB: --exclude='ext/oodle' is required.  ext/oodle is a LOCAL symlink to a
# built OodleUE clone; without the exclude, rsync --delete pushes the dangling
# symlink and wipes the real Oodle SDK staged on the remote (disabling Oodle).
# To (re)enable Oodle on a remote, stage the SDK once: copy src/, include/, and
# lib/<platform>/ under ext/oodle/Engine/.../Sdks/2.9.16/, then configure with
# -DOODLE_LIB_VARIANT=shipped.

# SSH aliases: test-c6a (Zen 3 SSE4.1), test-c8i (Xeon AVX-512 VBMI2),
#              test-c8g (Graviton 4 NEON)
```

After every full sweep, save the per-platform raw output and a
headline-level `.md` summary to `results/` so we can diff across
revisions and cite prior numbers.  Then regenerate the HTML
figures with `cmake --build build --target figures` (or run
`python3 extras/figures/build.py` directly) — outputs go to
`figures/` at the project root, read by `extras/figures/build.py`
from the most recent sweep tag.

## Key Files

The codec is a single tree-walk + wire-format engine in
`pivco_huffman_codec.c`, compiled once per backend (`PIVCO_BACKEND_*`)
into an OBJECT library.  Each compile pulls in the matching
`pivco_huffman_primitives_<backend>.h` via the router header
`pivco_huffman_primitives.h`.  The unify-framework refactor landed in
five phases ending 2026-05-14; before that, each backend had its own
.c file with a duplicated tree walk -- now all four share one.

- `include/pivco_huffman.h` — public API + table struct
- `src/huffman_table.c` — `pivco_huffman_build_table` + flat-subtree detection
- `src/pivco_huffman_codec.c` — unified codec.  Compiled once per backend
  with `-DPIVCO_BACKEND_{SCALAR,NEON,X86,AVX512}`.  Owns: tree walk
  (encode + BU decode), wire-format I/O via `pivco_huffman_wire.h`,
  optional FSE attempt on the raw bitmap.  Does not include any SIMD.
- `src/pivco_huffman_primitives.h` — router; selects the backend header
  based on the `PIVCO_BACKEND_*` define
- `src/pivco_huffman_primitives_scalar.h` — scalar primitive implementations
- `src/pivco_huffman_primitives_neon.h` — NEON primitive implementations
- `src/pivco_huffman_primitives_x86.h` — SSE4.1 + AVX2 primitive implementations
- `src/pivco_huffman_primitives_avx512.h` — AVX-512 VBMI2 primitive implementations
- `src/pivco_huffman_wire.h` — single source of truth for the per-node wire record (K_right + FSE marker + bitmap)
- `src/pivco_huffman_neon_tables.{c,h}` — shared NEON compress_tab + expand_tab
- `src/pivco_huffman_x86_tables.{c,h}` — shared x86 compress_tab + expand_tab (used by codec_x86 + codec_avx512 BU SSE-tail)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MarcinZukowski/pivco-huffman](https://github.com/MarcinZukowski/pivco-huffman) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
