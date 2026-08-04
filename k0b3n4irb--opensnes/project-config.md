---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Design philosophy — read first

Before proposing or accepting a refactor, consult **`PHILOSOPHY.md`**
at the repo root. OpenSNES is a 2D game engine, not a thin C-over-asm
wrapper, and five principles guide every API decision (sane defaults
with escape hatches, hidden quirks with documented escape, opt-in
modules, type-safe boundaries, predictable performance). Use them as
acceptance criteria when evaluating a change. The "Non-goals" section
of that doc is also load-bearing — it lists patterns the project
deliberately refuses (no GC equivalent, no monolithic engine class,
no `printf` in core lib, no mandatory framework lifecycle).

## Build Commands

```bash
make                    # Build everything: compiler → tools → library → examples
make compiler           # Build cc65816 (cproc+QBE) and WLA-DX assembler/linker
make tools              # Build gfx4snes, font2snes, smconv
make lib                # Build OpenSNES library (LoROM + HiROM + SA-1 + SuperFX)
make examples           # Build all example ROMs
make clean              # Clean all build artifacts
make clean && make      # Full rebuild (REQUIRED after compiler or runtime changes)
```

Build a single example:
```bash
cd examples/text/hello_world && make
```

## Testing

All testing goes through **luna** (cycle-accurate native emulator, pinned
binary — no Node/WASM/Mesen2):
```bash
scripts/install-luna.sh                                  # fetch pinned luna (tools/luna-test/luna.version)
python3 tools/luna-test/luna_runner.py --coverage        # corpus liveness
python3 tools/luna-test/luna_runner.py --compare         # visual regression (fbhash; multi-point for animated examples)
python3 tools/luna-test/probes/run_all.py                # functional probes (input→WRAM)
make tests                                               # all of the above
```

luna runs SA-1 / Super FX / DSP-1 natively, so there is no chip-ROM side
channel. See `.claude/rules/testing.md` for the validation workflow and
`.claude/notes/chantiers/luna_migration.md` for the migration off snes9x-WASM.
Changes are classified A/B/C/D by impact scope.

## Compilation Pipeline

```
main.c → cproc (C11 frontend) → QBE w65816 (codegen) → wla-65816 (assembler) → wlalink (linker) → game.sfc
```

The `bin/cc65816` wrapper orchestrates cproc→QBE→wla-65816. QBE's w65816 backend emits WLA-DX syntax directly (`.db`, `.dw`, `.SECTION`) — no post-transform.

## Architecture

### Key Directories

- **compiler/** — Three git submodules: `cproc/` (C frontend), `qbe/` (65816 backend), `wla-dx/` (assembler/linker). Also builds `wla-superfx` for SuperFX GSU code.
- **lib/** — Hardware library. C sources in `lib/source/*.c`, ASM in `lib/source/*.asm`, headers in `lib/include/snes/`. Built as separate LoROM, HiROM, SA-1, and SuperFX object sets.
- **templates/** — ROM bootstrap: `crt0.asm` (startup + NMI handler), `hdr*.asm` (ROM headers), `runtime.asm` (math routines, now in lib/source/), `memmap*.inc` (memory maps). These are the single source of truth — examples don't duplicate them.
- **make/common.mk** — Universal build rules included by every example. Handles graphics conversion, multi-file C compilation, SNESMOD audio, SA-1/SuperFX/HiROM mode selection, module linking.
- **tools/** — `gfx4snes` (PNG→SNES tiles), `smconv` (IT→SPC700), `luna-test/` (luna-driven test harness: runner, manifest, baselines, probes)
- **examples/** — 54 ROMs organized by category (text, graphics, input, audio, maps, memory, games)

### Enhancement Chip Support

- **SA-1** (`USE_SA1=1`): Same 65816 ISA at 10.74 MHz. Shares I-RAM ($3000-$37FF) with main CPU. Per-example `sa1_boot.asm` for custom coprocessor code. See `docs/tutorials/sa1.md`.
- **SuperFX** (`USE_SUPERFX=1`): Custom RISC ISA (GSU). Two-stage build: `.sfx` → `wla-superfx` → `wlalink -b` → `.sfx.bin` → `.incbin`. GSU code is assembly-only (no C compiler). **Validated by luna**, which detects and executes the GSU natively in the headless test harness (the old snes9x-WASM harness could not detect the GSU — "GSU: NOT DETECTED" — and needed a Mesen2 side channel; both are gone since the luna migration).

### Example Makefile Pattern

```makefile
OPENSNES := $(shell cd ../../.. && pwd)
TARGET   := game.sfc
ROM_NAME := GAME NAME
USE_LIB  := 1
LIB_MODULES := console sprite dma input   # only link what you need
CSRC := main.c
include $(OPENSNES)/make/common.mk
```

## Code Style

- C: 4 spaces, K&R braces, snake_case functions/vars, UPPER_CASE constants
- Use fixed-width types: `u8`, `u16`, `s16`, `u32` (from `snes.h`). `unsigned int` = 2 bytes, `unsigned long` = 4 bytes on this target (since chantier A1; `compiler/cproc/type.c` `typeint` size 2 / `typelong` size 4). This matches `.claude/rules/compiler.md`.
- ASM: labels at column 0, instructions indented with tab, `.section` for organization
- Commits: [Conventional Commits](https://www.conventionalcommits.org/) — `feat(scope):`, `fix(scope):`, `perf(scope):`, etc.
- Scopes: `lib`, `compiler`, `runtime`, `tools`, `examples`, `build`
- IMPORTANT: Do NOT add `Co-Authored-By` trailers for AI tools in commit messages.

## Critical Constraints


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [k0b3n4irb/opensnes](https://github.com/k0b3n4irb/opensnes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
