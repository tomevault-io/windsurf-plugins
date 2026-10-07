---
trigger: always_on
description: Guidance for coding agents (and humans) working in this repository. `README.md`
---

# AGENTS.md

Guidance for coding agents (and humans) working in this repository. `README.md`
says what Manticore is and how to use it; this file says how to change it
without breaking the one thing that makes it work: **the compiler compiles
itself**.

## What this is

Manticore is a PHP-to-native AOT compiler written in PHP. It compiles a large
subset of PHP 8.5+ to standalone arm64 / x86_64 binaries through LLVM IR, and it
compiles its own `src/` to a byte-identical fixpoint. The Zend interpreter is the
oracle: if `php` runs a program, the native binary must print the same bytes.

There is no Rust, no C, no PHP runtime in the output. `src/` is pure PHP, one class per
file, path mirrors FQN. `bin/manticore` and `lib/*.o` are build artifacts and
are gitignored.

## Build

```bash
bin/build                      # self-host rebuild via the manifest — THE normal loop
bin/build --seed               # cold bootstrap (Zend runs the compiler once to seed it)
bin/build --verify             # rebuild, then the fixpoint + suite gate
bin/build --auto-seed          # on a bootstrap gap, escalate to the seed (CI)
bin/build --fast               # -O1, application only, to bin/manticore.fast — NOT gate evidence
bin/compile                    # cold seed + stdlib build; fallback only, not the loop
```

Iterate with `bin/build`, not `bin/compile`. `bin/compile` runs the whole compiler
under Zend (~8 min); `bin/build` uses the installed native compiler (~1–2 min).

**A failed `bin/build` poisons `bin/manticore` and `lib/*.o`.** The build refuses
to write a library `.o` that contains an undefined-function trap, but a build that
died half-way leaves the previous artifacts in an unknown state. Re-run it to a
clean finish before trusting anything.

## Test

```bash
bash tests/aot/run.sh                # AOT suite: cases/*.php + expected/*.out, auto-discovered
bash tests/aot/run.sh -k <substr>    # filter cases (do this first — the full suite is minutes)
bash tests/aot/run.sh -j 0           # all cores (note: `-j 0` is two args)
bash tests/aot/xfail.sh              # known-bug repros (tests/aot/repro): XFAIL = still open, XPASS = fixed, promote it
bash tools/difftest.sh               # byte parity vs the `php` interpreter over the corpus
bash tools/selfhost_fixpoint.sh      # gen2 IR == gen3 IR, self-host suite, rebuild stability
bash tools/install_smoke.sh          # an INSTALLED compiler (on $PATH, behind a symlink) finds its own lib/
bash tools/docker/run_tests.sh --gate [--amd64]   # the same on Linux (~2 h arm64)
```

Rules of evidence:

- **A green build proves nothing about the suite.** Run the cases.
- **Never rebuild the compiler while a suite or difftest is running.** The runner
  invokes `bin/manticore` per case; a mid-run rebuild mixes two compilers into one
  result.
- **`tools/selfhost_fixpoint.sh` replaces `bin/manticore` with a stage binary
  while it runs.** Kill it mid-way and that stub is what stays in `bin/`. Let it
  finish, or restore `bin/.manticore.prev` (the last binary `bin/build` swapped out).
- The Linux gate is not optional for anything touching `src/Runtime/`, syscalls,
  errno or the async engine — macOS and Linux diverge there.
- The `dump-*` commands do not link the stdlib, so a stdlib/FFI callee resolves as
  `unknown` and a bug in that path vanishes from the dump. Read the final binary.
- A test that passes without the fix is not a test.
- Cap every log you write (`| tail`, `head -c`). A runaway trace log has filled
  the disk more than once. Delete probe binaries and `.ll` files as soon as they
  are read.

### Adding a test

Create `tests/aot/cases/<name>.php` + `tests/aot/expected/<name>.out`. No manifest.
The expected output should be what `php` prints (`php -d xdebug.mode=off` — xdebug
poisons the oracle). Run it with `bash tests/aot/run.sh -k <name>`.

### Reporting a bug you are not fixing now

A bug found on the way to something else is **a repro plus a GitHub issue** — never
a line in a ledger, a memory note or a `TODO`. A note cannot be run, so nobody
learns when it stops being true: the first sweep of the old ledgers found 46 of
~100 "open" entries no longer reproducing.

1. Minimise it to one self-contained file: `tests/aot/repro/{wrong,crash,leak}/<name>.php`
   — `wrong` = runs, output differs; `crash` = SIGSEGV / SIGBUS / invalid IR;
   `leak` = memory php releases and the binary does not. Line 2 is a one-sentence
   comment that becomes the issue title.
2. Record the oracle beside it: `php -d xdebug.mode=off -d display_errors=stderr <name>.php > <name>.expected`.
   A leak repro measures itself — `memory_get_peak_usage()` (peak RSS natively)
   around the loop, printing `flat` under a threshold — copy `repro/leak/throw_path.php`.
   A superset feature has no oracle: write the `.expected` by hand.
3. `bash tests/aot/xfail.sh -k <name>` must say `XFAIL`. An `XPASS` means the
   repro does not show the bug; a claim without a red repro is not filed.
4. `bash tools/file_bug.sh tests/aot/repro/<kind>/<name>.php` (`-n` prints the
   body and files nothing). It refuses a repro that passes, files the issue with
   the PHP source, php's output and the actual output in the body (labels `bug` +

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [manticorephp/compiler](https://github.com/manticorephp/compiler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
