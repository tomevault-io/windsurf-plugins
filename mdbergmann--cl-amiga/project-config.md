---
trigger: always_on
description: Common Lisp environment for AmigaOS 3+ with bytecode VM, written in C (C89/C99).
---

# CL-Amiga

Common Lisp environment for AmigaOS 3+ with bytecode VM, written in C (C89/C99).

## Target Platforms

- **Primary target**: AmigaOS 3+, 68020+ CPU
- **Development host**: macOS / Linux

## Build

```
make host          # Build for host (gcc)
make test          # Fast test tier: C unit tests + shell tests (mandatory — must pass before committing)
make test-plus     # Fast tier + host-cold-test (sento cold-load smoke test)
make test-extra    # Heavyweight trunk integration scripts (quicklisp/ansi-tests)
make clean         # Remove build artifacts
```

**These gates cover the clamiga runtime — the C code and the Lisp
library it ships.** The Lambda's Tale engine and the Closure game,
formerly subprojects under `examples/games/`, are **their own repos**
since 2026-07-24 (`../../lambda-tale` and `../../closure-tale` in
development; Closure vendors this repo and the engine as submodules)
with their own suites and their own `CLAUDE.md`.  Work over there
never gates on this repo — and work that leads back into the runtime
(a compiler bug, a missing CL function, an FFI gap found by playing
the game) is a commit *here*, with every gate above applying to it in
full.  Keep the two changes, and their gates, apart.

Cross-compile for Amiga and test via FS-UAE:
```
make -f Makefile.cross amiga        # Cross-compile to build/cross/clamiga (m68k-amigaos-gcc)
make -f Makefile.cross test-amiga   # Cross-compile, copy binary, launch FS-UAE, verify results
make -f Makefile.cross clean        # Remove cross-build artifacts
```
- Uses `m68k-amigaos-gcc` toolchain from `tools/m68k-amigaos-gcc/prefix`
- **Preferred method** for building the Amiga binary — faster than compiling inside the emulator with vbcc
- `test-amiga` places the binary in `build/amiga/`, boots FS-UAE, runs the Amiga test suite, and verifies results
- Runs fully unattended: FS-UAE auto-quits (`C:UAEquit`) when the suite finishes, and a host-side watchdog (`verify/realamiga/run-fs-uae.sh`) kills it if clamiga hangs; results are checked automatically

## Release Process

Releases are tag-only (no GitHub release artifacts). Follow the `v0.4`/`v0.5` precedent:

For a downloadable **binary release** (AmigaOS 3 + MorphOS), run `scripts/make-binary-release.sh` after tagging — it cross-builds the aos3 binary, packages a natively built MorphOS binary (`MOS_BIN=...`, built with `Makefile.mos` on MorphOS), assembles `bin/aos3` + `bin/mos` + `lib/` (FASLs where portable — the header comment documents which files must ship as source and why) + `docs/` + `examples/` under `build/release/`, smoke-tests the layout, and emits `.zip`/`.lha`.

1. **Gates green first**: `make test-plus`, `make test-gc-stress`, `make test-extra`, and `make -f Makefile.cross test-amiga` must all pass (`pkill fs-uae` first if an emulator is lingering). Record the Amiga-suite and test-extra pass counts — they go in the tag message.
2. **Bump the version** — exactly two files:
   - `src/core/types.h`: the `CL_VERSION_MAJOR/MINOR/PATCH` block + `CL_VERSION_DATE` (DD.MM.YYYY). Everything derives from this block (banner, `LISP-IMPLEMENTATION-VERSION`, `$VER:` cookie, FASL cache key — stale caches invalidate themselves; `tests/test_version.c` enforces the contract).
   - `README.md`: the two hardcoded examples in the "Version" section (`lisp-implementation-version` and `Version clamiga` output).
   - `CL_FASL_VERSION` is independent — bump it only if serialization actually changed (see FASL Versioning below).
3. **Commit** as `chore(release): bump version to X.Y` touching only those two files, with a headline paragraph summarizing the cycle (see `git show v0.4` for the format). Re-run `make test` on the bumped tree.
4. **Annotated tag**: `git tag -a vX.Y -m "CL-Amiga X.Y — <headlines>; Amiga suite N/N, test-extra N/0"` with the real numbers from step 1.
5. **Push**: `git push origin master vX.Y`.

**Deriving the headlines — do not use `vPREV..HEAD`.** Master's history was rewritten (`git filter-repo`, 2026-07-24), so tags created before that point are no longer ancestors of master: `git log v0.4..HEAD` returns the *entire* rewritten history (786 commits), not the release delta, and silently yields headlines from cycles long past. Check with `git merge-base --is-ancestor vPREV HEAD`; when it fails, find the previous release's *rewritten* bump commit instead and diff from that:

```
git log --oneline --all --grep="bump version to X.Y"   # find the rewritten bump commit
git log <that-commit>..HEAD --oneline                  # the true delta
```

For 0.5 the correct base was `95bda65` (89 commits), not the `v0.4` tag.

## Architecture

- `CL_Obj` = `uint32_t` tagged value; heap pointers are **arena-relative byte offsets** (not raw pointers)
- Single arena, bump allocator with free-list fallback, mark-and-sweep GC with compaction (moving) when fragmented — see GC Safety below
- Single-pass compiler: S-expressions → bytecode; stack-based VM
- All OS calls go through `platform.h` (`platform_posix.c` / `platform_amiga.c`)
- **Threading** (MP package): kernel threads with per-thread VM, TLV dynamic bindings, stop-the-world GC coordination, locks, condition variables
  - POSIX: pthreads, pthread_rwlock, `__sync_*` atomics

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mdbergmann/cl-amiga](https://github.com/mdbergmann/cl-amiga) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
