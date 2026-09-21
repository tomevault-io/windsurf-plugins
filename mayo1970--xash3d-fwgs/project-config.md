---
trigger: always_on
description: handles `__BIG_ENDIAN__`/`__BYTE_ORDER__` detection generically, and a real
---

# AGENTS.md - xashPS3 porting bible

## 1. What this is

A PlayStation 3 homebrew port of [Xash3D-FWGS](https://github.com/FWGS/xash3d-fwgs)
(GoldSrc-compatible engine), built against the open-source **PSL1GHT /
ps3toolchain** stack only -- never the official Sony SDK. The engine source
here is a full vendored copy (not a submodule), matching how the other
console ports in this workspace (ioQuake3-PS4, ioQ3-One) are structured.
Upstream `xash3d-fwgs` (sibling directory) is read-only reference material,
not edited from here.

Testing is **real PS3 hardware only** (CFW/HEN). All hardware validation
below assumes the UDP-log-and-console workflow.

## 2. Hardware validation protocol

This project follows a strict goal-oriented, hardware-validation-gated
workflow. Do not implement more than one goal-stack item ahead. Do not
speculate on hardware behavior -- verify on the actual console.

- **ANALYSIS** -> **IMPLEMENTATION** -> **BUILD_REQUEST** -> **WAITING_FOR_HARDWARE** -> **VALIDATION** -> **NEXT_GOAL**
- `BUILD_REQUEST` must give: exact build command, expected output
  (`EBOOT.BIN`/`.pkg`), deploy method (FTP via webMAN, or PKG install from
  USB), and what should be observed on screen/audio/controller.
- `WAITING_FOR_HARDWARE` stops all speculation until the user reports back
  exactly `SUCCESS: ...` or `FAILURE: ...`.
- On `FAILURE`, produce a minimal isolated test case or a 3-item diagnostic
  checklist -- never rewrite the whole implementation and guess again at the
  same time.
- If the same goal fails hardware validation more than twice in a row: stop,
  escalate (UDP register trace, webMAN/ps3mapi memory peek, or community
  consultation), and ask whether to mark the goal BLOCKED or continue with
  new diagnostic data.

## 3. Goal stack

- [x] **0. Scaffolding** (this session) -- repo structure, build-system
      wiring, platform stub skeleton. No build attempted yet.
- [x] **1. Toolchain bring-up**: null `main()` -> fself -> pkg -> boots to a
      black screen on real hardware, with the UDP debug log sink
      (`nc -ul 18194`) wired up first, before anything else. **VALIDATED on
      real hardware 2026-07-19** via `tools/ps3_bringup` built inside the
      `ps3dev/ps3dev:latest` Docker image (native sfo/pkg tools, no pyexpat
      issue) -- black screen, no crash/XMB-return, UDP heartbeat confirmed
      (also confirmed `sizeof(void*)==8`, `sizeof(long)==8` on real hardware,
      matching the LP64 finding in section 6).
- [x] **2. Platform stubs compile**: `./waf configure --ps3 && ./waf build`
      reaches the link stage. **DONE 2026-07-19** -- `engine/xash` links as a
      real ELF64 big-endian PowerPC64 EXEC inside the `ps3dev/ps3dev:latest`
      Docker image (the project's canonical toolchain). Numerous real,
      build-verified fixes landed along the way (see AGENTS.md section 6 and
      project memory for the full list -- wrong toolchain-header assumptions,
      missing libc functions, a GNU ld static-archive ordering bug, etc.).
      **Follow-up RESOLVED 2026-07-19 (build-verified, not yet hardware-
      validated)**: `filesystem_stdio`/`ref_soft` now build via
      `--static-linking=filesystem_stdio,ref_soft` (`ref/soft/exports.txt`
      added, `GetRefAPI`). Three real bugs found and fixed getting there,
      all in the generic `scripts/waifulib/xshlib.py`/`xcompile.py` tooling,
      none PS3-specific hacks:
      1. `xcompile.py`'s `PS3` class had no `ld()`/`objcopy()` methods and
         never pointed `conf.environ['LD']`/`['OBJCOPY']` at the cross
         binutils (PSP's block already does this) -- `xshlib.py`'s
         `conf.find_program('ld'/'objcopy')` fell back to the host's.
      2. Even after adding those, top-level `wscript`'s
         `conf.load('xshlib xcompile ...')` loaded `xshlib` *before*
         `xcompile`, so the environ overrides didn't exist yet when
         `xshlib.configure()` ran `find_program`. Fixed by reordering to
         `conf.load('xcompile xshlib ...')`.
      3. With the real cross `ld -r` in place, host `/usr/bin/ld` had been
         silently mangling `filesystem/VFileSystem009.cpp`'s PPC64 ELFv1
         `.opd`/`R_PPC64_TOC` relocations (the only C++ TU in
         `filesystem/`) -- `ld: Relocations in generic ELF (EM: 21)` /
         `error adding symbols: file in wrong format`. Root cause was
         purely #2; once real `ld` was used this was a non-issue. Separately,
         `xshlib`'s `ld -r` task didn't link each relocatable module's own
         STLIB `use` deps (e.g. `ref_soft` -> `ref_common`), so `Matrix4x4_*`
         /`gEngfuncs`/etc were undefined at `xash`'s final link; fixed by
         adding `${STLIBPATH_ST:STLIBPATH} ${STLIB_ST:STLIB}
         ${LIBPATH_ST:LIBPATH} ${LIB_ST:LIB}` to `xshlib`'s `run_str` (bare
         `ld`, not gcc, so no `-Wl,`/`STLIB_MARKER` wrapping) -- this lets
         `ld -r`'s normal archive-member selection pull `ref_context.c.o`
         into `ref_soft.o`, where the existing `objcopy -G
         lib_ref_soft_exports` step then localizes those symbols so they
         don't collide with the engine's own identically-named globals
         (each ref module is designed to carry its own private copy of
         these, normally isolated by being a separate `.so`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mayo1970/xash3d-fwgs](https://github.com/Mayo1970/xash3d-fwgs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
