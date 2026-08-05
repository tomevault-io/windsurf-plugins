---
trigger: always_on
description: This repo is a Nintendo DS source port of the BattleShip Smash 64 decompilation:
---

# AGENTS.md

## Mission

This repo is a Nintendo DS source port of the BattleShip Smash 64 decompilation:

```text
Original Smash 64 decomp game code + Nintendo DS backend = playable port
```

Do not turn it into a handwritten Smash clone or DS-native gameplay rewrite.

## Hard Rules

- Treat `decomp/` as read-only reference source. Never edit it.
- Inspect relevant BattleShip source before changing gameplay or renderer behavior.
- Inspect `decomp/sm64-nds` before DS backend architecture changes.
- Prefer coherent original translation-unit imports under `src/import`.
- Put DS/backend behavior under `src/nds` or `src/port`; compatibility declarations belong under `include`.
- Graduate imported subsystems live. Do not add proof-only branch reruns, one-bit proof masks, or permanent seed/restore wrappers.
- Migrate or delete obsolete bounded modes when natural runtime replaces them.
- New harness modes are only for scene-level capabilities.
- Fix root causes at the shared seam. Do not hide symptoms with arbitrary
  offsets/constants, duplicated state, frame checks, or asset-specific hacks.
- Treat flashes, corruption, nondeterminism, hangs, and unexplained state
  differences as failures.
- Respect DS CPU, RAM, VRAM, bandwidth, alignment, fixed-point, and graphics
  limits.
- Treat generated outputs and emulator payloads as generated; never hand-edit them.
- Publish exactly `smash64ds.nds` and
  `smash64ds-battle-playable-hwtri.nds`; all lab outputs stay in `builds/`.
- User-facing ROMs must be verifier-covered configurations.
- Use only repo-local scripted melonDS. Do not commit runner configs, binaries,
  logs, or shard artifacts.
- In a Task 24 quiet slot, hash-migrate permanent performance and visibility
  evidence before deleting any closed lab build or worktree. Rotate only
  uncited verifier/emulator telemetry; `artifacts/performance` and
  `artifacts/visibility` are permanent evidence. Never combine cleanup with an
  active implementation or remove an ambiguous/dirty worktree.
- Performance features must prove ENGAGEMENT on retail hardware (a counter, HUD row, or device photo), not only a melonDS win. A feature that silently
  degrades, falls back, or disables itself on device may not ship enabled.
- Rendering-side changes may approximate: they gate on a reported fidelity
  budget (synchronized screenshot diffs plus the owner's visual approval), not
  pixel exactness. Gameplay/source behavior remains bit-exact and verifier-
  gated. Engagement proof is a counter on the shared engagement HUD row,
  confirmed by batched device smoke boots; per-feature retail runs are reserved
  for cache/TCM/DMA/IO-class claims, which melonDS cannot referee. CPU-work-
  removal claims may KEEP on melonDS typed A/B evidence behind their flag until
  the next device checkpoint.
- Device A/B reports must show the 2/3/4/5+ VBlank-interval histogram and the
  max interval, never min FPS or a half-second average. Min FPS is
  discontinuous: one frame crossing the 4->5 VBlank boundary reads as ~12 FPS
  while the histogram stays continuous and is the actual pacing signal.
- Run `scripts/New-Smash64DSSnapshot.ps1 -Mode Lean` after verified progress as the final project command. Run nothing after it.

## DS Visual Fidelity

Gameplay, collision, rules, state, camera meaning, and flow stay source-faithful.
Presentation targets roughly 90% overall likeness, not pixel identity. Timebox
cosmetic exactness to one measured experiment; on a tick, memory, or P1 miss,
keep the cheapest recognizable source-derived approximation and move on.
Prefer still frames, reduced cadence, simpler geometry/layout, and DS-native
effects. Record the source, visible delta, measured reason, and screenshot under
`artifacts/visibility`. Never approximate gameplay semantics or telegraphs and
never accept missing or corrupt presentation. Dream Land water is frozen at
exact source frame 0 on the original geometry; its animated replacement is gone.

## Operating Model

Start each cycle with:

```powershell
.\scripts\verify-all.ps1 -Profile Boundary -List
git status --short
```

Then read `docs/P1_EXECUTION_BOARD.md` and `docs/HANDOFF.md`. The board is the
only dynamic queue; handoff contains only the restart surface. Select its
highest-impact unowned red P1 row.

Preserve a known-good checkpoint before risky changes. On regression, find the
first bad change before layering fixes; trace shared dependencies before edits.

For performance iteration, use one synchronized eight-frame A/B comparison on
an identical ROM/configuration/window. Primary evidence is ticks, FPS, a dated
screenshot, and automated screenshot analysis; semantic/state/geometry counters
are cheap correctness guards. Stop on a decisive KEEP or REVERT. Run a third A
only when A/B is noisy, near its gate, surprising, or internally inconsistent.
Do not require routine A/B/A, 32-frame, or 128-frame promotion runs.
Milestone tick targets are directional, not per-cut discard gates: keep every
repeatable correctness-preserving gain and accumulate it toward the target.

Use the smallest focused checker or benchmark while editing. Run one widest
relevant verifier for a kept checkpoint: Boundary for battle-only work, or

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rockenrooster/Smash64DS_Port](https://github.com/rockenrooster/Smash64DS_Port) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
