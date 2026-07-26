---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**hs-to-rocq** converts Haskell source code to Coq (Gallina) using the GHC API. It is part of the DeepSpec/CoreSpec project. The tool parses Haskell via GHC, applies user-specified "edits" to guide the translation, and pretty-prints valid Coq output.

- **Current target**: GHC 9.10.3, Coq 8.20 (branch `ghc910-coq820`)
- **Languages**: Haskell (the tool, ~14K LOC in `src/`), Coq (generated output and proofs)

## Build Commands

```bash
stack build                                    # Build hs-to-rocq executable
stack exec hs-to-rocq -- -e edits -o output/ Input.hs  # Run on a file
make                                           # Build base + base-thy + containers
make -C examples/base-src vfiles               # Re-generate base/ from Haskell sources
make -C examples/tests                         # Unit tests (.hs → .v → coqc)
make -C examples/base-tests                    # Tests requiring base/
make -C examples/containers                    # containers lib + theories (regenerates + builds)
make -C examples/transformers                   # transformers lib
make -C examples/ghc clean && make -C examples/ghc  # Regenerate+compile GHC lib + theories
cd examples && ./boot.sh                       # Full bootstrap (all examples)
# Individual Coq dirs: cd <dir> && coq_makefile -f _CoqProject -o Makefile && make -j
```

Use relative path instead of absolute path when `cd` to a directory.

### CI commands
- `/ci` — run CI checks locally, report pass/fail
- `/ci-fix` — run CI checks, diagnose and fix failures, commit fixes

## Architecture

### Translation Pipeline

1. **Parse & typecheck** Haskell via GHC API (`src/lib/HsToRocq/ProcessFiles.hs`)
2. **Load edits** from `.edits` files (`src/lib/HsToRocq/Edits/Parser.y`, `Types.hs`)
3. **Convert** GHC AST → Coq Gallina AST (`src/lib/HsToRocq/ConvertHaskell/`)
4. **Pretty-print** Gallina to `.v` files (`src/lib/HsToRocq/Rocq/Pretty.hs`)
5. **Output** `.h2ci` interface files for cross-module translation

### Key Source Directories

- `src/lib/HsToRocq/Rocq/Gallina.hs` — Coq AST data types (Term, Sentence, Definition, Inductive, etc.)
- `src/lib/HsToRocq/Rocq/Pretty.hs` — Pretty-printer rendering Gallina AST as Coq source
- `src/lib/HsToRocq/ConvertHaskell/` — Core conversion engine:
  - `Expr.hs` — Expression conversion (largest file, ~1700 lines)
  - `Module.hs` — Whole-module conversion
  - `Monad.hs` — Conversion monad carrying edits/renamings/state
  - `Declarations/` — Data types, classes, instances, type synonyms
  - `HsType.hs`, `Pattern.hs`, `Variables.hs` — Type/pattern/variable conversion
- `src/lib/HsToRocq/Edits/` — Edit DSL parser and types
- `src/lib/HsToRocq/CLI.hs` — Command-line interface and file processing orchestration
- `src/include/ghc-compat.h` — CPP macros for GHC version compatibility (8.4–9.10)
- `src/lib/HsToRocq/Util/GHC/` — GHC API compatibility wrappers

### The Edits System

Edits files are a plain-text DSL that guides translation of constructs that don't directly map to Coq. Key directives: `skip`, `rename`, `rewrite`, `redefine`, `add`, `order`, `termination`, `coinductive`, `axiomatize`, `manual notation`. Full documentation in `doc/source/edits.rst`.

Edits are organized per-module:
```
module-edits/<Module>/<Path>/edits       # per-module edits
module-edits/<Module>/<Path>/preamble.v  # Coq code prepended to output
module-edits/<Module>/<Path>/midamble.v  # Coq code inserted mid-file
```

### Output Directories

- `base/` — **Generated** Coq base library. **Do not edit directly** — regenerate from `examples/base-src/` instead.
- `base-thy/` — Hand-written proofs and lawful typeclass instances for `base/`
- `examples/` — Translation examples; each has `Makefile`, `edits`, `module-edits/`, `lib/` (output), `theories/` (proofs)

### Build Plumbing

- `common.mk` — Included by all example Makefiles; defines `HS_TO_ROCQ` variable (resolves to `stack exec hs-to-rocq --`)
- Each Coq directory uses `_CoqProject` + `coq_makefile` to generate its Makefile
- `.h2ci` files store interface information for cross-module translation
- `make vfiles` (base-src) skips existing `.v` files — `rm` the target file before running to force regeneration

### Stale .vo recovery
If you see "inconsistent assumptions over library Coq.Init.Prelude", rebuild the full chain: `base` → `base-thy` → `examples/containers` → `examples/ghc`. For coq_makefile dirs: `find <dir> \( -name '*.vo' -o -name '*.vok' -o -name '*.vos' -o -name '*.glob' -o -name '.*.aux' \) -delete && coq_makefile -f _CoqProject -o Makefile && make -j`.

### Axiomatized lib functions
When lib/*.v functions are `Axiom`, theories/*.v proofs that unfold them must be `Admitted`. Check with `grep "^Axiom" lib/Module.v` before attempting computation-based proofs. See "GHC example" section for the full list of axiomatized functions.

### False theorems

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [plclub/hs-to-rocq](https://github.com/plclub/hs-to-rocq) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
