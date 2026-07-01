---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Deduce is a proof checker and small functional language for teaching logic, written in Python. Source `.pf` files contain definitions, theorems, and proofs; running `deduce.py file.pf` either prints `... is valid` or fails with an error. The standard library lives in `lib/` and is auto-imported as a prelude unless `--no-stdlib` is passed.

Single dependency: `lark==1.2.2` (Python 3.12+, Makefile uses 3.13).

## Code style

This repository is closed-world: nothing outside this repo depends on its Python API, and the `.pf` source language is the only stable surface. **Backwards compatibility within the repo's own internals is not a concern** — when refactoring, delete old shims rather than carrying them forward.

Smaller code is better. Prefer the change that removes lines over the one that adds them. When two paths are functionally equivalent, pick the one with less surface area. Don't leave deprecation aliases, legacy fallbacks, or "just in case" branches around — if a thing is unused, delete it; if it's needed later, recover it from git.

This applies to refactors especially: when a global goes away, the module-level alias for it goes away too. When a function's signature changes, every caller updates in the same PR — no overload that accepts both shapes.

## Filing bugs you notice in passing

When you spot a bug — or a strong-suspicion bug — while doing something else, **file it as a GitHub issue before declaring your current task done**. Do not just mention it in your reply summary; the reply gets thrown away, the issue persists. **If you noticed a bug, the task is not done until the issue exists.**

Trigger this when any of the following hold:

- A test, sweep, or experiment surfaced concrete evidence of a defect (failing fixture, divergent output, crash, wrong AST, broken round-trip).
- Reading the code, you saw a clear logic error a fix-this-now ticket would describe.
- A documented invariant is violated in code you read.

Do **not** file for: vague code smells, "could be cleaner," missing features, or hunches without a concrete repro.

Each issue must include:

- One sentence stating the bug.
- A minimal repro (source file or shell command — prefer pulling an existing in-tree file over fabricating one).
- Observed vs. expected behavior.
- A link back to where you found it (PR, sweep script, file path).

Group related defects into one issue with categorized sub-bugs (see #931 for the shape) rather than spamming N tiny issues. Cross-reference any umbrella issue the bug sits under with `Refs #N`.

Default to filing without asking first. The exception is when the "bug" might be intentional design — in that case, file anyway but lead the body with "Is this intentional? If so, close." Better to over-file with a quick close than to silently drop a real bug.

## Running and testing

```sh
# Check a single file (uses recursive-descent parser by default)
python deduce.py path/to/file.pf

# Pick a parser explicitly
python deduce.py --recursive-descent file.pf
python deduce.py --lalr file.pf

# Make targets run static checks and BOTH parsers across the test/lib tree
make static       # ruff + mypy
make tests        # static + should-validate + should-error + should-warn
make tests-lib    # checks the stdlib itself
make              # static + token checks + tests (default)
```

`test-deduce.py` is the higher-level harness used in CI:

```sh
python test-deduce.py                  # default: lib + should-validate + should-error + should-warn + prelude + parser equivalence
python test-deduce.py --lib            # only ./lib
python test-deduce.py --passable       # only test/should-validate
python test-deduce.py --errors         # only test/should-error (diff vs .err files)
python test-deduce.py --warns          # only test/should-warn (valid + diff vs .warn files)
python test-deduce.py --equiv          # compare RD/LALR ASTs for a curated grammar corpus
python test-deduce.py --parser         # only test/parse (parser-error fixtures)
python test-deduce.py --site           # generates and checks doc code from gh_pages/doc

# Regenerate the expected stderr fixture for a should-error test
python test-deduce.py --generate-error test/should-error/foo.pf
python test-deduce.py --regenerate-errors      # all of them

# Regenerate the expected warning fixture for a should-warn test
python test-deduce.py --generate-warn test/should-warn/foo.pf
python test-deduce.py --regenerate-warns       # all of them
```

The test harness runs under the active Python interpreter, which must be Python 3.12+ with `lark` installed. Both parsers must pass — when changing parsing or AST, run with `--lalr` and `--recursive-descent`.

Useful `deduce.py` flags while debugging: `--verbose` (or `--verbose full`), `--unique-names`, `--trace <function>`, `--traceback`, `--quiet`, `--suppress-theorems`, `-r` (recurse into directories).

## Architecture

The pipeline lives in five Python files. The flow for one file is in `deduce.py:deduce_file`:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jsiek/deduce](https://github.com/jsiek/deduce) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
