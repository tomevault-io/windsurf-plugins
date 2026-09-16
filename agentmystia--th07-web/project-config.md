---
trigger: always_on
description: Operating manual for AI agents and contributors. It encodes the working
---

# AGENTS.md

Operating manual for AI agents and contributors. It encodes the working
rules, the verification loop, the file-format facts, and the orchestration
protocol that produced the current codebase. Follow it exactly; nearly
every rule in it was earned by a real defect.

## 1. Mission

Reimplement **Touhou Youyoumu ~ Perfect Cherry Blossom (TH07)** in
TypeScript for the browser, driven by the original game data. Stages 1-8
are data-driven and playable; the current fidelity target is original-grade
Stage 1-6 behavior and presentation. Extra/Phantasm remain lower-confidence.

**Default rule: reproduce the original game exactly.** Do not simplify,
rebalance, redesign, modernize, or approximate original behavior unless it
is explicitly approved here or requested by the user. The engine is
*data-driven*: original `.ecl/.std/.anm/.msg/.sht` binaries are embedded
(`src/data/th07-data.ts`) and executed by our parsers and VMs. Hand-written
behavior is a last resort and must carry a comment flagging it.

## 2. Authority order

When sources conflict, higher wins:

1. Current user instruction.
2. Approved modernizations (§3).
3. Original data and executable in `reference/`: readable disassemblies in
   `reference/ECL7|DSTD7|MSG7|ANM7`, raw unpacked files in
   `reference/th07-original/`, `reference/Th07.exe` for Ghidra. **The binary
   present is v1.00, not v1.00b** — build tag `"0100_"` at `.rdata` `0x48D230`,
   title string `ver 1.00`, 607744 bytes, sha256 `1251458d…`. Every replay in
   this tree was recorded on **v1.00b** (tag `"0100b"`, exe size 650752,
   checksum `0xAEC5445C`, stored at image `+0xE0/+0xD8/+0xDC`). Cited decompile
   addresses do resolve against the v1.00 binary, so static reads stay usable —
   but it cannot play the replays (see docs/REPLAY_ALIGNMENT_HANDOFF.md) and a
   build difference is an open, unfalsifiable-from-v1.00 hypothesis for residual
   drift. Label new findings with the build actually inspected.
4. Existing project implementation.
5. External docs (thtk source, PyTouhou, priw8's sht-webedit docs, wikis) —
   cross-validation only, never sole authority. TH06 semantics are NOT
   TH07 semantics; several opcode tables differ (§6).

`reference/` is git-ignored, local-only, read-only. Never commit, ship, or
serve it. Browser runtime code must never read from it.

**Current implementation is not proof of correctness.** If the data says
otherwise, the implementation is wrong.

## 3. Approved modernizations

- Focus hitbox dot rendering while focused (collision identical).
- Dev/debug tooling (`?test=1` hook, `dev-shot`/`dev-menu` scripts) — must
  not change shipped gameplay behavior.
- Web Audio BGM looping via `loopStart/loopEnd` sample frames from
  `thbgm.fmt` instead of whole-file loops.
- Plain-text control hints on menu screens.
- Stage-start player fly-in: the original places the player in-residence at
  the spawn point with a 240-frame invuln window and no entrance animation
  (its init preloads the materialize timer past its threshold). On stage
  start we instead fly the player up from below the playfield over 60
  frames (input/firing locked, invulnerable), then hand off to that
  240-frame invuln. Respawn after death is unchanged. Player-only visual;
  no gameplay, timing, or collision semantics change once landed.
- Low-latency canvas presentation, ON by default: the display context is
  requested with `desynchronized: true` (+ `alpha: false`). Browsers that
  grant it (Chromium) skip 1–2 compositor vsyncs; the renderer then draws
  every frame to an offscreen backbuffer and `present()` copies it in one
  op — a granted context is never drawn incrementally (that incremental
  front-buffer drawing was the 8552afe Stage-5 spell-card flicker, before
  the backbuffer existed). Non-granting browsers feature-detect (context
  attribute readback, no UA sniffing) to the direct path, byte-identical
  to the old behavior. Output pixels are identical either way; sim/replay
  untouched. Kill switches: `?desync=0` (player-facing), `?backbuffer=1`
  (test-only, forces the present() path on engines that never grant).
  Note: headless Chromium GRANTS desynchronized, so every headless harness
  exercises the backbuffer path; readback/screenshots cannot prove scanout
  flicker absent — a manual Stage-5 spell-card eyeball on real desktop
  Chrome stays part of acceptance for presentation-layer changes.

Nothing else. In particular: **no invented visual content**. If the data
has no moon, there is no moon. Absence of data gets a flagged fallback and
a report, not fabrication.

## 4. Non-negotiable invariants

Every commit must satisfy ALL of:

1. `npm run check` — zero TypeScript errors.
2. `npm run build` — clean esbuild bundle.
3. `npm test` — all unit tests pass.
4. Clean headless boot: `node scripts/dev-shot.mjs /tmp/s.png 300` prints a
   snapshot with enemies spawning and **no `PAGE ERRORS` line**.
5. No isolation hacks in the tree: no debugging early-`return`, no
   commented-out subsystems, no hardcoded test state. A crashed agent once
   left `return;` at the top of `drawBackground()` — it made all code after
   it unreachable, which disables TypeScript control-flow narrowing and
   produced seven phantom "possibly null" errors. Treat unreachable code as

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AgentMystia/th07_web](https://github.com/AgentMystia/th07_web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
