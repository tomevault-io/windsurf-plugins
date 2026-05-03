---
trigger: always_on
description: SuperPSX is a PSX (PlayStation 1) emulator running **natively on PlayStation 2** hardware (EE/R5900 CPU at ~294MHz). It uses a MIPS→MIPS JIT recompiler (dynarec) since both the PSX (R3000A) and PS2 (R5900) share the MIPS instruction set.
---

# SuperPSX — Copilot Instructions

## Project Context

SuperPSX is a PSX (PlayStation 1) emulator running **natively on PlayStation 2** hardware (EE/R5900 CPU at ~294MHz). It uses a MIPS→MIPS JIT recompiler (dynarec) since both the PSX (R3000A) and PS2 (R5900) share the MIPS instruction set.

The emulator runs inside **PCSX2** for development/testing. The final target is real PS2 hardware.

A **PSP (Allegrex) port** also exists, running on PPSSPP for development. The PSP uses a similar MIPS ISA but lacks R5900 extensions (VU0, MMI, TLB, dual HI/LO).

## Communication Rules

- **ALWAYS use the `ask_questions` tool** to communicate with the user. The user speaks Spanish.
- **NEVER write plain text responses** for questions, confirmations, or status updates. Route everything through `ask_questions`.
- After completing a task, ask what to do next via `ask_questions`.
- If the user's intent is ambiguous, clarify via `ask_questions` before proceeding.

## Build & Test Commands

```bash
# Build (from workspace root)
cmake --build build 2>&1 | tail -5

# GTE test (expect: 1150 passed, 0 failed) — 20s is enough
rm -f build/superpsx.ini && printf "gte_vu0 = 0\n" > build/superpsx.ini && \
perl -e 'alarm 20; exec @ARGV' make -C build run \
  GAMEARGS=tests/gte/test-all/test-all.exe > ./build/gte_out.txt 2>&1; \
pkill -f pcsx2 2>/dev/null; \
grep -E "Passed|Failed" ./build/gte_out.txt | head -5; \
rm -f build/superpsx.ini && ln -sf $(pwd)/superpsx.ini build/superpsx.ini

# CPU test (expect: Result 00000101) — 20s is enough
perl -e 'alarm 20; exec @ARGV' make -C build run \
  GAMEARGS=tests/psxtest_cpu/psxtest_cpu.exe > ./build/cpu_out.txt 2>&1; \
pkill -f pcsx2 2>/dev/null; \
grep 'Result:' ./build/cpu_out.txt | head -3

# Timer test — 40s needed; compare line-by-line against PSX reference
perl -e 'alarm 40; exec @ARGV' make -C build run \
  GAMEARGS=tests/timers/timers.exe > ./build/timer_out.txt 2>&1; \
pkill -f pcsx2 2>/dev/null; \
awk '/timers test/,/Done\./' ./build/timer_out.txt \
  | sed -E 's/\x1b\[[0-9;]*m//g' | sed -E 's/^\[[ 0-9.]+\] //' \
  | grep -v '^Set GS\|^Update\|^Frame rate\|^ResetGraph\|^$\|^make' \
  > ./build/timer_clean.txt; \
diff ./build/timer_clean.txt tests/timers/psx.log | head -80

# Crash Bandicoot (manual test — ask user)
make -C build run GAMEARGS=isos/CrashBandicoot/CrashBandicoot.cue
```

**IMPORTANT:**

- All tests run BIOS first, so if you broke jit maybe the Phase 1 (BIOS) test will fail instead of Phase 2 (CPU/GTE). Check the actual test output to confirm.
- macOS has no `timeout`/`gtimeout`. Use `perl -e 'alarm N; exec @ARGV'` for timeouts.
- **Always redirect to file** (`> ./build/out.txt 2>&1`), NEVER pipe (`|`). Pipes cause SIGPIPE to kill the emulator prematurely.
- After each `perl -e 'alarm ...'` test, add `pkill -f pcsx2 2>/dev/null` to clean up the PCSX2 process.
- For GPU/rendering changes, do NOT run automated tests — ask the user to launch Crash Bandicoot and MK2 manually and report results.

## PSP Build & Test Commands

```bash
# Configure PSP build (separate build directory)
cmake -S . -B build-psp 2>&1 | tail -5

# Build PSP main target
cmake --build build-psp 2>&1 | tail -5

# Build PSP playground
cmake --build build-psp --target jit_playground.elf 2>&1 | tail -5

# Run PSP playground (expect: 110/110 passed) — 25s is enough
/Applications/PPSSPPSDL.app/Contents/MacOS/PPSSPPSDL -v \
  build-psp/jit_playground.elf > ./build/psp_playground_out.txt 2>&1 &
PID=$!; sleep 25; kill $PID 2>/dev/null; wait $PID 2>/dev/null; \
grep "PRINTF" ./build/psp_playground_out.txt | sed 's/.*stdout: //' | \
  grep -E "Results|FAIL"
```

**PSP IMPORTANT:**

- PSP build uses `$PSPDEV/psp/share/pspdev.cmake` toolchain, auto-detected by CMake.
- PPSSPP at `/Applications/PPSSPPSDL.app/Contents/MacOS/PPSSPPSDL`.
- Use `-i` (interpreter) and `-v` (verbose) flags to capture `printf` output as `I[PRINTF]` lines.
- `printf` output is in PPSSPP's log at `I[PRINTF]: HLE/sceIo.cpp:... stdout: <text>`.
- PSP SDK libraries must be linked LAST (after `-lm -lc`). Never link both `-lpspuser` and `-lpspkernel` — use only `-lpspkernel` to avoid "stubs out of order" error.
- `psp-fixup-imports` must run on every ELF after linking (handled automatically by `create_pbp_file` for main target; added as POST_BUILD for playground).

## JIT Playground

A separate ELF (`jit_playground.elf`) for testing the dynarec in isolation with a mini-DSL. 110 micro-tests split across 8 category files, plus an expansion ratio report.

```bash
# Build playground (EXCLUDE_FROM_ALL — not built by default)
cmake --build build --target jit_playground.elf 2>&1 | tail -5

# Run playground (expect: 135/135 passed on PS2, 110/110 on PSP) — 20s is enough
perl -e 'alarm 20; exec @ARGV' make -C build run-playground \
  > ./build/playground_out.txt 2>&1; \
pkill -f pcsx2 2>/dev/null; \
grep -E "Results|FAIL" ./build/playground_out.txt
```

**Key files:**

- `tests/jit/playground.h` — DSL header (opcode encoding macros, test framework macros)
- `tests/jit/playground_main.c` — Entry point, `pg_run_jit()` dispatch loop

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [frangarcj/superpsx](https://github.com/frangarcj/superpsx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
