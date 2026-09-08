---
trigger: always_on
description: This file is the on-ramp for AI coding agents working on `nes-to-sms`.
---

# Agent Notes

This file is the on-ramp for AI coding agents working on `nes-to-sms`.
Read it first. The project is a Rust workspace that translates NES
(mapper 0 / NROM) ROMs into buildable Sega Master System projects.

## Project overview

`nes-to-sms` is a pipeline, not a hand-port of any one game:

```
.nes ROM + profile.toml
    → parse / classify / discover
    → lift 6502 to semantic IR
    → lower IR to Z80
    → emit WLA-DX project (Makefile, link script, runtime, assets)
    → assemble to .sms
```

Super Mario Bros. is the active regression target, but the engine stays
game-agnostic. Game-specific facts live in TOML profiles and the Z80
runtime, not in Rust source.

Current state (measured from the workspace):

- Rust workspace with **13 crates** under `crates/`.
- About **35 kLOC of Rust** and **9.5 kLOC of hand-written Z80 runtime**.
- **`cargo test --workspace` passes** with roughly **478 tests**.
- The pipeline runs end-to-end on `Super Mario Bros. (World).nes` and
  emits a complete WLA-DX project tree.
- Generated ROMs boot under Mednafen; real-SMS speed remains an open
  issue. Phase S (docs/speed-recovery-plan.md, 2026-09-04) brought SMB
  from ~4.4× to ~1.98× over the frame budget — full speed fits GPGX's
  standard ≤200% overclock menu; frame-diff has RAM-parity AND
  VDP-parity oracles (FD_VDP_DUMP/FD_VDP_CHECK golden workflow — hashes
  are timing-sensitive, see docs/visual-parity.md) plus an
  FD_FAR_EDGES profile feeding the profile-driven bank placer; docs/handport-comparison.md explains the ceiling.
- Visual fidelity is gated by the NES ground-truth frame oracle
  (FD_NES_DUMP + scripts/nesref/compare_frames.py, docs/visual-parity.md).
  The BG variant ring uses NT refcounts (chrmap.s BGV_REFCNT) so visible
  slots are never recycled; `SMS_EXPECT_BGV_CONSISTENT=0` on the
  1-1-clear route is the regression guard. Known cosmetic limits
  (sprite behind-priority, unmapped score-popup tiles) are listed in
  docs/visual-parity.md.
- SMB builds use `[translation] stack_discipline = "native"` (native
  CALL/RET via `rt_far_tail`) and thirteen `[[replacement]]` hooks in
  `runtime/hooks_smb.s`. The canonical SMB test ROM is `.roms/smb.nes`
  (NOT the EmuDeck "Super Mario Bros. (World).nes" — different dump;
  the profile's `MoveLakitu` at $CF28 is data there).

## Canonical documentation

- `docs/master-plan.md` — architecture, principles, RAM/ROM layout,
  phased goals, and validation strategy. This wins when docs conflict.
- `docs/completion-plan.md` — the current ordered work queue.
- `docs/current-status-and-gaps.md` — status inventory only; do not use
  its old slice-driven "next step" as policy.
- `README.md` — operational on-ramp, but some counts (crates, tests,
  progress percentages) can become stale; trust the workspace and the
  docs above.

## Technology stack

- **Language:** Rust 1.95+ with edition 2024.
- **Build system:** Cargo workspace rooted at `Cargo.toml`.
- **Assembler / linker:** WLA-DX (Z80) and `wlalink`.
- **Emulators for validation:** Mednafen, Genesis Plus GX via Docker,
    Emulicious as a cross-check.
- **Containerization:** `docker/Dockerfile.toolchain` + `compose.yaml`.
- **No host installs policy:** WLA-DX, Mednafen, and other retro tooling
  should only be installed inside the Docker image, not on the host.

## Workspace layout

```
Cargo.toml              workspace root (resolver = "3", edition 2024)
crates/
  nes_rom/              iNES / NES 2.0 header parsing, PRG/CHR/vectors
  cpu6502/              2A03 instruction decoder (all official + stable
                        unofficial opcodes; unstable opcodes decode to
                        mnemonics but fail closed in the pipeline)
  analysis/             function discovery, CFG, code/data classification
  profile/              TOML profile schema + loader
  ir/                   semantic IR with explicit flags + memory tags
  lower/                IR → Z80 lowering (shadow flags, runtime calls)
  z80_emit/             Z80 instruction encoder + WLA-DX asm text emitter
  z80_emu/              in-Rust Z80 interpreter for differential tests
  oracle_6502/          in-Rust 6502 interpreter for differential tests
  assets/               CHR → SMS 4bpp, palette mapping, PPM previews
  sms_project/          WLA-DX project writer (Makefile, link.cfg, sms.asm)
  validation/           differential harness: oracle vs. z80_emu
  cli/                  `nes-to-sms` binary + `trace-sms`, `frame-diff`,
                        `z80-diff`, `replay-state` utilities
runtime/                hand-written Z80 SMS runtime (.s files)
profiles/               game profiles: smb.toml, cv1.toml, alterego.toml, cv3.toml
profiles/smb/           SMB-specific acceptance routes / checkpoints
docs/                   plans, status, research notes
tools/                  small shell/python helpers for iteration
tests/synthetic/        synthetic test ROM inputs (managed by tests)
poc/                    legacy proof-of-concept history; treat as
                        disposable unless a task explicitly asks for it
```

## Crate responsibilities

| Crate | Responsibility |
|-------|----------------|
| `nes_rom` | Parse iNES headers, split PRG/CHR, read reset/NMI/IRQ vectors. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [akitaonrails/nes-to-sms](https://github.com/akitaonrails/nes-to-sms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
