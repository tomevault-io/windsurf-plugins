---
trigger: always_on
description: All indicator code is **generated** by a single generator, **`ta_codegen`**
---

# CLAUDE.md - TA-Lib Code Generation Guide

## Architecture Overview

All indicator code is **generated** by a single generator, **`ta_codegen`**
(`ta_codegen/generator/`, Rust): it parses `ta_codegen/input/` → IR → renders
per-backend (C, Java, .NET, Rust). The C backend is generated **in place** into
`src/ta_func` / `src/ta_abstract` (the shipped library); the Rust/Java/.NET bindings
live under `ta_codegen/output/`. It also generates the JSON-RPC test servers, the bench
binary, `include/ta_func_unguarded.h`, the `include/ta_defs.h` FuncUnstId enum, the
shipped Java (`ta_codegen/output/java/library/.../Core.java`, `CoreAnnotated.java`, `FuncUnstId.java`), and owns the
build-system source lists (CMake `LIB_SOURCES`, `src/ta_func/Makefile.am`,
`ta_func_list.txt`). It also generates the **ta-lib.org website** — one page per function
under `website/src/functions/` (from each function's `ta_codegen/input/<name>/<name>.md`)
plus a grouped `website/src/functions/index.md`, written directly into the VuePress site
source tree (`website/`) — the one generated output that lives there rather than under
`ta_codegen/output/`. (`docs/` itself now holds only hand-written dev-docs.)

> The legacy C generator `gen_code` was **removed** in the canonical cutover (v0.7.1);
> `ta_codegen` is the only generator.

**Why the C is generated in place and not symlinked** to `ta_codegen/output/c`: a whole-dir
symlink breaks autotools' per-dir libtool recursion (`make` enters the symlink's *physical*
path, so the Makefile's relative `../../libtool` fails with `Error 127`), and it would also
force a packaging dereference step. Real files in `src/` avoid both — and downstream
consumers (notably the PHP `trader` extension) glob `src/ta_func/*.c` straight out of the
released source tarball.

**Build separation (important):** the C build systems (CMake + autotools) build **only
C** — the library + the C tools (`ta_regtest`, `ta_bench`). `ta_codegen` is Rust and is
built/run with cargo via the developer script `scripts/build.py` (`ta_codegen` /
`generate` / `servers`); **CMake never invokes cargo**, so a C-only setup needs no Rust
toolchain.

The correctness baseline that all `ta_codegen` backends are verified against is the
frozen pre-cutover reference (the `reference-pre-cutover` tag, served as `ta_ref_serve`)
plus the hardcoded `ta_regtest` expected values.

See `ta_codegen/generator/CLAUDE.md` for ta_codegen internals and
`src/tools/ta_regtest/CLAUDE.md` for the test-runner spec.

### Source of Truth: ta_codegen/input/

`ta_codegen/input/` is the single source of truth for ALL generated code
(161 indicator definitions).

- **YAML** = data, config, enums, IDL. Pure definitions with no logic.
  - MAType and FuncUnstId enums (`ta_codegen/input/enums.yaml`)
  - Function metadata (inputs, outputs, optional params, groups) — per-function `<name>/<name>.yaml`
  - Shared library types — RetCode, CandleSetting defaults, Compatibility — are hand-written templates the generator emits (NOT under `input/`, which is algorithms only, and not YAML); they live with the generator under `ta_codegen/generator/templates/` (e.g. `templates/rust/types.rs`, `templates/c/ta_retcode.c.template`)
- **C source files** = logic. Anything with computation.
  - Indicator implementations (`ta_codegen/input/<name>/<name>.c`)
  - Helper functions (`ta_codegen/input/helpers/`)
  - **No logic in YAML, ever.**

No hand-coded string literals for type definitions or scaffolding in the codegen.
Do not hand-edit **generated** files under `ta_codegen/output/` — they are
overwritten on the next `generate`. Note some hand-written library source now
lives under `output/` too (e.g. the Java `meta/` reflection layer and tests under
`output/java/library/`); the generator preserves those and never overwrites them.

## Quick Reference Commands

```bash
# Build (from any directory in the repo; binaries land in bin/)
scripts/build.py                # C library + all C tools (CMake)
scripts/build.py ta_regtest     # Just the C test runner (CMake)
scripts/build.py ta_codegen     # Rust codegen tool (cargo)
scripts/build.py generate       # Regenerate per-function source for all backends (cargo)
scripts/build.py servers        # Generate + compile JSON-RPC language servers (cargo)

# Test
scripts/build.py test           # C reference tests only (quick)
scripts/build.py regtest        # Full pipeline: servers (cargo) + C tests + cross-language verification
scripts/build.py regtest-only   # Codegen verification only (skip C reference tests)

# ta_codegen (run from ta_codegen/generator/)
cargo run -- generate                            # Generate indicator code for all backends
cargo run -- generate --func=SMA --backend=rust  # Specific function + backend
cargo run -- generate-servers                    # Generate JSON-RPC servers
cargo run -- build                               # Compile servers into bin/
cargo run -- extract                             # Extract indicators from C source → YAML
cargo test                                       # ta_codegen's own test suite

# ta_regtest directly (from bin/)
./ta_regtest                                     # C reference tests only

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TA-Lib/ta-lib](https://github.com/TA-Lib/ta-lib) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
