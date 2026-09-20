---
trigger: always_on
description: Class overrides for Project Zomboid Build 42 (Java, LWJGL/OpenGL) that improve chunk
---

# PZ_Optimization

Class overrides for Project Zomboid Build 42 (Java, LWJGL/OpenGL) that improve chunk
streaming and driving frame time, plus a hands-off benchmark harness that measures them.
Public repo (xD3I/PZ_Optimization). Maintainer: the repository owner (they/them).

## Objective (stated 2026-09-18)

"Consistent frame time if the CPU and GPU utilization allows it; the CPU and GPU should
always be used to the max if the framerate is not smoothly pegged at 240 fps."
Every benchmark report must show frame-tail metrics (p99 / p99.9 / spikes / jitter) AND
utilization (CPU/GPU load from sysmon) over the route window. "fps < 240 and hardware not
saturated" is itself a finding. Chunk-latency wins are done; do not spend more on the streamer.

## Hard rules

- **`decompiled/` stays local** (gitignored, 24 MB CFR output of the whole jar). Never `git add`
  it. `src/overrides/` (the 25 shadowed classes with our `// pzopt:` edits) IS committed since
  2026-09-19: the maintainer confirmed the sources may ship. Every edit is still described in prose in
  `docs/override-edits.md`, and the `// pzopt:` markers stay on every changed line.
- **Shared machine.** Several Claude sessions and the maintainer use the one game install and `~/Zomboid`.
  Before a launch or a reinstall check both `pgrep -f '[P]rojectZomboid64'` and
  `pgrep -f '[h]arness/run.sh'` (excluding your own). Message busy peers (ListAgents /
  SendMessage) before reinstalling or starting a batch. See `.claude/skills/bench-run`.
- **Run etiquette.** The maintainer is usually at the machine. Say a run is about to start before
  launching, one run at a time, never long batches. Never edit `harness/run.sh` while a run is in
  progress (bash reads it incrementally; a mid-edit launch died and its EXIT trap corrupted
  `latestSave.ini`). Launch from a copy (`harness/.run-snapshot.sh`) if someone else edits it.
- **Never kill with a self-matching pattern.** `pkill -f '<pattern>'` where the pattern appears in
  your own command line kills the tool shell (exit 144). Use bracket patterns like
  `[P]rojectZomboid64` or a saved PID.
- **All recordings and videos are AV1 HDR.** Every `--record` capture (gpu-screen-recorder
  `-k av1_hdr`: AV1 10-bit PQ / BT.2020) and every video published under `docs/media/` (the
  stitch scripts, `harness/encode-av1-hdr.sh`, the `-1080` README copies) is encoded AV1 10-bit
  with PQ / BT.2020 tags; never tone-map to SDR H.264. SDR sources are mapped to PQ; posters and
  GIFs are the only tone-mapped derivatives. See `harness/CLAUDE.md`.
- **Real saves are never loaded or written** by a harness run. Runs use the copied bench save
  `Saves/Sandbox/pzopt-bench` and must quit on their own. Launching the game via
  `harness/run.sh` is authorized without asking.
- Commit and push only when asked. Commits end with
  `Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>`.

## Environment facts

| Item | Value |
|---|---|
| OS | CachyOS (Arch), fish shell, `paru` for AUR, 16 cores, 30 GB, RTX 4090 |
| JDK | system `jdk-openjdk` 26; game bundles its own JRE (build.sh compiles with `--release` for it) |
| Game dir | `/games/steamapps/common/ProjectZomboid/projectzomboid/` (native Linux depot since 2026-09-18) |
| User dir | `~/Zomboid` (console.txt, Saves, Lua/, mods/, options.ini) |
| Layout detection | `scripts/pz-env.sh` (PZ_DIR / ZOMBOID env override) |
| Desktop | 5120x2160, 240 Hz; the game renders windowed at desktop resolution |
| Decompiler | CFR at `~/.local/share/java/cfr.jar` for reading; Vineflower for overrides |
| Code index | `.codegraph/` exists; use `codegraph_explore` before grep/Read |
| Java LSP | `jdtls` not installed (check `command -v jdtls` before relying on the LSP tool) |

## Reading game code

Read game classes from `decompiled/` (CFR output of all 3,407 game classes, package tree such as
`decompiled/zombie/iso/IsoChunk.java`). Never re-decompile or unzip the jar for those packages.
Only two methods lack bodies (`CompressIdenticalItems.areItemsIdentical`,
`ChooseGameInfo.readModInfoAux`); use `javap -c -p` for them. If the jar mtime changes (game
update) re-run `scripts/decompile.sh` and `scripts/regen-overrides.sh`.

## Layout

| Path | Purpose | Details |
|---|---|---|
| `src/` | overrides, shims and the `pzopt` helper package | `src/CLAUDE.md` |
| `scripts/` | build / install / decompile / test | `scripts/CLAUDE.md` |
| `harness/` | run.sh, analysis scripts, baselines, run outputs | `harness/CLAUDE.md` |
| `config/` | MangoHud profiles | `config/CLAUDE.md` |
| `docs/` | plans, findings, override edit log, dashboard | `docs/CLAUDE.md` |
| `tools/` | standalone Java probes (JFR dump, GLFW swap probe, static audit) | `tools/CLAUDE.md` |
| `tests/` | JVM-only unit tests for pzopt classes (`scripts/test.sh`) | |
| `decompiled/` | CFR output, local only | |
| `openspec/` | OpenSpec change proposals (opsx skills) | |

## Skills (in `.claude/skills/`)

| Skill | Use when |
|---|---|
| `bench-run` | launching any measurement run (bench / drive / parity / verify) |
| `showcase-drive` | recording the stock-vs-optimized drive videos and the quad stitch |
| `build-install` | compiling the overrides and installing them into the game dir |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xD3I/PZ_Optimization](https://github.com/xD3I/PZ_Optimization) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
