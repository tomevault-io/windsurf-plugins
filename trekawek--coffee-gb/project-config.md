---
trigger: always_on
description: coffee-gb is a Game Boy (DMG) / Game Boy Color (CGB) emulator in Java with
---

# CLAUDE.md — coffee-gb agent knowledge base

coffee-gb is a Game Boy (DMG) / Game Boy Color (CGB) emulator in Java with
Maven modules including `core`, `swing`, and `controller`.

The derived hardware models are documented in `doc/derived/` (ppu-stat-model.md,
cpu-interrupt-model.md, apu-model.md) — **the code implements those specs; read
them before touching Gpu/StatRegister/Cpu/Sound.**

## Operational workflows

Use the repository skills instead of duplicating their procedures here:

- `.claude/skills/release-new-version/SKILL.md` for Maven and GitHub releases.
- `.claude/skills/fix-compatibility-issue/SKILL.md` for issue investigation,
  isolated fixes, PR creation, screenshots, and issue follow-up.
- `.claude/skills/merge-pr/SKILL.md` for conflict handling, CI gating, merging,
  and branch cleanup.

## Build & test loop

- Use `/opt/maven/bin/mvn` in scripts (PATH lacks mvn in non-interactive shells).
- Run java/mvn from the repo root; use **absolute paths in classpaths** (relative
  classpath entries misbehaved under zsh in this environment).
- Integration profiles can be combined with commas. Their test methods run on two
  threads by default; override with `-Dintegration.test.threadCount=N`. BullyGB is
  kept serial because its two methods have tight 10-second timeouts.
- Full battery (each ~1-2 min):
  ```
  mvn -pl core test                            # unit
  mvn -pl core test -Ptest-mooneye,test-dmgacid2,test-cgbacid2
  mvn -pl core test -Ptest-blargg-individual,test-blargg
  ```
- Fast iteration mains in `core/src/test/.../integration/support/`:
  - `MooneyeMain <roms...>` — prints PASS/FAIL + register dump + test output
    (mooneye protocol: pass = B,C,D,E,H,L = 3,5,8,13,21,34; magic `LD B,B`+0x40
    loop; failure register dump shows expected/actual tables).
  - `BlarggMain <roms...>` — memory protocol ($A000 status, sig DE B0 61, text
    at $A004); **selects DMG/CGB from cartridge header byte 0x143** (bit 7).
  - `TraceMain`, `PpuProbe`, `BootProbe <rom> [cgb]`, `OamProbe`, `OamDump`,
    `SpriteStallProbe` (measures emergent mode-3 sprite stalls; expected values:
    1 sprite at screen x on a tile boundary = 11 T, offsets 1..4 → 10..7 T,
    5..7 → 6 T, +6 T per chained same-x sprite).
  - All mains need `System.exit(0)` — EventBus threads are non-daemon and hang
    the JVM otherwise.
- Classpath for direct java runs:
  `mvn dependency:build-classpath -pl core` output + `core/target/classes` +
  `core/target/test-classes` (absolute paths).
- Test runners: `MooneyeTestRunner` picks CGB for `*-C.gb`/`*-cgb*` roms, CGB0
  revision for `*-cgb0*`, NORMAL boot for `boot_*` roms, SKIP otherwise.
- `GbMicrotestRunner` reads the suite's automated result at `$FF82`; the profile
  runs all 482 self-verifying DMG timing ROMs and pins the 31 interactive/test-bench
  archive entries that have no machine-readable verdict.
- `GbcHwTestRunner` compares the AntonioND suite's SRAM output with original DMG/CGB
  captures; use `-Ptest-gbc-hw`. The smaller alyosha-tas Mooneye-derived collection
  runs with `-Ptest-misc-gb`.

## The timing model — calibrated constants and their interlocks

These constants were pinned by hardware-verified tests. **They interlock; do not
change one without re-running the full battery.**

### Clock domains and phase

- `Gameboy.tick()` = 1 T-cycle (4.19 MHz). CPU works only every 4th tick
  (`Cpu.clockCycle`, a free-running global mod-4 counter, never re-anchored).
  All memory reads/writes therefore happen on one global machine-cycle phase.
- Our CPU executes a memory-read op **one M-cycle late**: `LDH A,(a8)` samples
  the bus at tick +12 of the instruction, `LD A,(HL)` at +8. This is a known
  quirk; the DIV presets and PPU anchors are calibrated around it. If you ever
  fix read timing to the hardware M-cycle, *every* preset below must be
  re-derived.
- **PPU line grid is machine-cycle-phase-locked**: the LCD-enable line starts at
  the LCDC write itself and is **455 ticks** long (normal lines 456); the
  power-on grid starts `ticksInLine = 1`. Result: the boot/SKIP grid and any
  re-enabled grid share one phase. This equality is what makes the
  mode2-interrupt→STAT-poll distance identical between tests that re-enable the
  LCD and tests that don't — it is pinned to the T by
  `intr_2_mode0_timing_sprites` vs `intr_2_mode0_timing` + `lcdon_timing-GS`.

### PPU (see doc/derived/ppu-stat-model.md for the full spec)

- Mode 3 is a T-exact dot pipeline (SameBoy-style) in `PixelTransfer`/`Fetcher`:
  1 pixel pop + 1 fetcher state per T; 7 fetcher states with the push completing
  in the same T as the last data read; line starts at `position = -16` with 8
  junk pixels; SCX alignment during positions -16..-9; discards at -8..-1.
- Object fetch: waits until fetcher ≥ `GET_TILE_DATA_HIGH_T2` **and** FIFO
  non-empty (`max(0, 5 - state)` T), then fixed 6 T. Sprites merge into a
  separate 8-px `SpriteFifo` popped in lockstep with the bg FIFO (left-edge
  clipping emerges from discarded pops; overlay priority: transparent-fill for
  DMG, oam-index for CGB).
- Pixel end `E = 248 + SCX%8 + stalls`; **visible STAT mode 0 = E+1**;
  **mode-0 STAT interrupt line = E+4** (3 T after visible; no quantization).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [trekawek/coffee-gb](https://github.com/trekawek/coffee-gb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
