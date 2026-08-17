---
trigger: always_on
description: Guidance for Claude Code when working in this repository.
---

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project Overview

**Macintosh IIvi core for MiSTer** — MC68030 @ 15.6672 MHz with on-chip PMMU,
VASP-equivalent chipset, NuBus (mdc824 video card as primary display), no FPU.

Lineage: the chipset/framework is the **MacLCII core imported at commit
`a254a02`** (see `docs/VASP_RETARGET.md` for the exact delta plan), because
per MAME "VASP is basically V8 with slightly different video and the RAM
limit lifted to 68MB". NuBus RTL comes from the Mac II core
(`../lbmactwo_MiSTer`). The CPU-verification corpora live in
`SingleStepTests/` (68030+PMMU, real-silicon adjudicated).

Key references:
- `MacIIvi_HardwareConfig.md` — the machine, MAME-verified (memory map, VIA,
  pseudoVIA, NuBus, interrupts)
- `docs/VASP_RETARGET.md` — V8→VASP delta plan + SDRAM layout + decisions
- `68030_CPU_IMPLEMENTATION_PLAN.md`, `68030_PMMU_TESTBENCH.md` — CPU/PMMU
- Local MAME source tree: `../mame/src/mame/apple/{maciivx,vasp,v8,maclc}.cpp`
  (tree = 0.288-132) — always check MAME before guessing hardware behavior.
- **MAME version discipline (owner decision 2026-07-17):** review against
  **0.288 source** (the local tree) as primary reference, BUT the installed
  runtime oracle and every HW-validated behavior of this core to date is
  **0.264** — and 0.288's sound/IRQ rework (afed5d318e4) is twice-suspect
  (error-41 hunt; the asc_v8 semantics port 6c95e20→reverted bc31773 broke
  app sound/froze games/slowed the clock on HW). When 0.288 and 0.264
  disagree, treat neither as automatically right: port coherent subsystem
  PAIRS (e.g., ASC + pseudoVIA latch semantics together, never one half),
  and gate on hardware sound/clock validation. 0.288 pairs maciivi's
  level-style ASC with the EDGE-latch base pseudovia — a fragile contract;
  the LC pairs it with a level-through v8_pseudovia (the ASCTester-validated
  combo). Our asc.sv+pseudovia.sv pair is 0.264-faithful and HW-proven.

## Hard rules

- **CPU sync rule**: `rtl/tg68k/` must stay byte-identical to
  `../MacLCII_MiSTer` at the pinned commit (currently `a254a02`). CPU fixes
  land in MacLCII first, then get re-copied. Never fork the kernel here.
- **Line endings**: repo policy is LF (`core.autocrlf=false`, enforced for
  *.sh via .gitattributes). The sim toolchain runs under **WSL** — CRLF in a
  shell script or Makefile breaks it.
- **Hardware deploys are ask-first**: never deploy/reboot the MiSTer without
  the owner's go for that specific build. Once authorized, HW validation on
  the device is the standard, decisive loop (deploy =
  `bash scripts/deploy_screenshot.sh` from Git-Bash, never WSL).
- **Framework files law** (adopted 2026-07-18 from MacLC `0d38a1a`): `sys/`
  is off-limits except wholesale template updates — constrain framework
  behavior from `rtl/` + qsf/sdc only. Q17 Lite has NO per-instance
  RAM_BLOCK_TYPE qsf assignment (illegal name); per-instance
  AUTO_SHIFT_REGISTER_RECOGNITION is legal.
- Work happens on feature branches, merged to `main` once validated (policy
  changed 2026-07-15 by the project owner; was direct-to-main).

## MacLC family sync (laws adopted 2026-07-30, broadened 2026-08-06)

WHOLESALE from `../MacLC_MiSTer` master (kept byte-identical modulo one
asc.sv comment): the SCSI/CD set (`rtl/scsi.v`, `rtl/cd_audio.sv`,
`rtl/cd_vol_lut.vh`, `rtl/asc.sv`, `rtl/scsi_vendor.vh`) AND — since the
2026-08-06 family sync (MacLC master `182186b`, their release state) — the
floppy stack (`rtl/swim.v`, `rtl/floppy.v`, `rtl/mfm_track_encoder.v`,
`rtl/floppy_track_encoder.v`) plus the standalone floppy TBs
(`verilator/tb_*.v`) and `verilator/scsi_bench/`.

3-WAY (apply the MacLC delta, preserve the documented MacIIvi fixups):
- `rtl/ncr5380.sv` + `rtl/dataController_top.sv` — fixups: CD_RING_LOG=2,
  disk RING_LOG i==0?5:4, local comments. (TB_ADDRW i==0?12:8 now MATCHES
  MacLC — 12 is load-bearing: scsi.v TB_SEND_CAP/CAP_LARGE_SEND gate on it.)
- `rtl/via6522.sv` — IIvi-local hunks flagged "MacIIvi-local" in-file: the
  ORB output-latch read merge (POST root cause #6 — sad Mac $0F/$33 without
  it; MacLC keeps a pins-only ORB read) and one `SIMULATION` trace ifdef.
- The emu tops re-derive by hand: MacLC.sv changes port to BOTH MacIIvi.sv
  and verilator/sim.v (mount presentation, dio_menu, anchors). MacLC's
  USE_DBG_HUD on-screen forensics deck is deliberately NOT ported (single
  v8-video assumption vs our ONBOARD_DISPLAY/mdc824 mux) — fetch + adapt
  from MacLC.sv if a floppy hunt ever needs it here.

The HPS contract is the Main_MiSTer fork `add-bluescsi-toolbox-for-MacLC` —
hps_io slots: disks 0/1, PRAM 2, Toolbox 3, CD 4, CD-changer 5; CD-DA is
served as ONE 2352-byte transaction per frame (sd_buff_addr[12:8] carries
the burst word address), so core and Main MUST move together. Toolbox caps
0x82 (multi-block GET + CAP_LARGE_SEND) additionally needs the HPS lineage
MacLC ships (their releases/ MiSTer binary, dda65f18+) — older Main still
works but the client falls back to slow single-block transfers.

- **Media-change pairing law** (MacLC 2026-08-05/06): the emu-top
  DSK_EMPTY_CY leave→insert hold and floppy.v's `disk_switched` SWITCHED
  sense reg landed TOGETHER and must move together — the hold alone was the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [danifunker/MacIIvi_MiSTer](https://github.com/danifunker/MacIIvi_MiSTer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
