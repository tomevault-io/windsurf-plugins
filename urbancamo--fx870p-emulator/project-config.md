---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev          # dev server on http://localhost:3007/fx870p-emulator/ (with /log endpoint)
npm run build        # vue-tsc type-check + vite build → dist/
npm run preview      # serve the production build locally
npm run dis          # run the CLI disassembler: npx tsx tools/dis.ts <romfile> [start] [end]
npm run compile      # compile BASIC to HD61700 machine code: npx tsx tools/compiler/compile.ts <file.bas>
npm test             # run vitest test suite (headless emulator tests)
```

Type-check via `vue-tsc -b` (run as part of `npm run build`).

ROM files are **not in the repo** — place `rom0.bin`, `rom1.bin`, `charset.bin` in `public/roms/` before running.

## Architecture Overview

Pure-browser emulator for the Casio FX-870P (Hitachi HD61700 CPU). No backend. State persisted to IndexedDB. A Vite dev-server plugin adds `POST /log` → `emulator-debug.log` for debug tracing.

### Emulator module pipeline

```
emulator.ts   rAF loop → cpuRun() [cpu.ts]
                                  → execInstr() [exec.ts]  ← 256-entry dispatch table
                                  → firePcMonitor() [def.ts]
              LCD: lcdRender() [lcd.ts] → Canvas ImageData
              I/O: onSerialTick() [port.ts], commDecTimer() [comm.ts]
```

All CPU state lives as **module-level `let` variables** in `def.ts`. Other modules import them by value and use setters (e.g. `setUa(v)`) to mutate. This is the ES module constraint workaround for mutable shared state.

### Memory map (18-bit physical address space)

| Region | Physical range  | Access      | File                        |
|--------|-----------------|-------------|-----------------------------|
| ROM0   | 0x00000–0x00BFF | 16-bit word | `rom0.bin`                  |
| ROM1   | 0x00C00–0x0FFFF | byte        | `rom1.bin`                  |
| RAM0   | 0x10000–0x1FFFF | byte        | (volatile)                  |
| ROM1b  | 0x20000–0x2FFFF | byte        | `rom1.bin` (offset 0x10000) |

**Critical**: `mr[]` in `def.ts` is only 36 bytes — the CPU register file, **not** the 64 KB address space. RAM is `memdef[2].data`.

### Address translation (`addr18` in `def.ts`)

```typescript
addr18(segment, offset):
  if (offset < ((ib << 8) & 0xC000)) segment = 0  // low addresses always ROM
  return offset + ((segment & 3) << 16)
```

- Stack (`push`/`pop`): segment = `ua >> 2`
- IX data (`ldd`/`std`): segment = `ua >> 4`
- IZ data: segment = `ua >> 6`
- Interrupt fetch: segment = 0 (forced, ignores UA)
- Normal fetch uses `delayed_ua` (UA from the previous instruction cycle)

For the stack to be in RAM, bits 3:2 of UA must be `01` (e.g. UA=0x14, 0x44, 0x54, 0xD4).

### HD61700-specific quirks

**`jr` offset encoding** — negative offsets use `0x80 - raw` (word arithmetic), NOT standard sign-extension. Raw byte 0x99 → offset = `0x80 - 0x99 = -0x19`. Both `disassemble.ts` and `tools/dis.ts` implement this correctly. **Do not change to `raw - 0x100`.**

**Flag register** — `setFlagsB(x)` sets Z_bit when `x ≠ 0` (inverted from standard CPUs). Condition code `nz` fires when Z_bit is SET (result was non-zero).

**Condition codes** in `testCC()` use `opcode[0] & 7` (lower 3 bits), not the upper bits. CC=7 = unconditional.

**`cal`/`rtn` return address** — `cal` pushes `pc - 1`; `rtn` pops and adds 1. Net result is correct return.

**`anc $1,$sz`** — reads `mr[sz]` (a general register at index `sz`), ANDs with `mr[1]`. If `sz = 0`, result is always 0 → z=1.

### Debug infrastructure

`emulator.ts` installs two monitors via `def.ts`:
- `setRamWriteMonitor(fn)` — called on every RAM write (used for boot-phase diagnostics)
- `setPcMonitor(fn)` — called before each instruction via `firePcMonitor(pc)` in `cpu.ts`

`remote-log.ts` batches log entries and POSTs to `/log` when `isRemoteLogEnabled()` is true. Toggle from the **Log** button in `CommPanel.vue`, or `window.ioDebug(true)` in the browser console.

### Debug workflow

1. `npm run dev`
2. Open browser → click **Log** (turns red) **before** clicking **Fresh Start**
3. Wait ~3 s for boot
4. Click **Log** again to flush
5. Read `emulator-debug.log`

## Reference files

| File                                  | Purpose                                                                                       |
|---------------------------------------|-----------------------------------------------------------------------------------------------|
| `reference/fx870p-rom-annotations.md` | Annotated ROM labels, call-flow, and bug analysis — **read this first** for any ROM debugging |
| `reference/fx870p-roms.md`            | Full ROM disassembly (regenerate with `npm run dis` after any disassembler fix)               |
| `reference/ROM Disassembly/*.src`     | Inline-commented source disassemblies (authoritative)                                         |
| `reference/fx870_es/`                 | Original Delphi 5 reference implementation                                                    |

## Known CPU emulation bugs fixed


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [urbancamo/fx870p-emulator](https://github.com/urbancamo/fx870p-emulator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
