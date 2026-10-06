---
trigger: always_on
description: - **Targets:** the route target is unchanged: normal menu/New Game -> r100 -> r101 -> r103 playable. The frame-time target is now 30 fps (per tick: gameplay G <= 24 + render R <= 6 + a 3.33 margin; the 2026-09-25 rethink, a coarse complete square first) on real NTSC Dreamcast hardware via GDEMU + VMU; 20 fps and a 15 fps fight fallback were the earlier targets. The SH-4 hardware model (`tools/hwmodel/hwproject.sh`) is the hardware estimate; Flycast is a proxy.
---

# RE4 Dreamcast implementation instructions

## Current direction (D367): supersedes the sections below where they conflict

- **Targets:** the route target is unchanged: normal menu/New Game -> r100 -> r101 -> r103 playable. The frame-time target is now 30 fps (per tick: gameplay G <= 24 + render R <= 6 + a 3.33 margin; the 2026-09-25 rethink, a coarse complete square first) on real NTSC Dreamcast hardware via GDEMU + VMU; 20 fps and a 15 fps fight fallback were the earlier targets. The SH-4 hardware model (`tools/hwmodel/hwproject.sh`) is the hardware estimate; Flycast is a proxy.
- **Rendering:** the user dropped GameCube rendering tech in favour of Dreamcast-native approaches and allows new offline assets.
- **Never altered:** collision, event sequencing, game state and AI. Render-only changes must keep the logic trace STRICT.
- **Decisions and work order:** every option is chosen by its hw ms impact. Since 2026-09-25 performance work runs in parallel, non-overlapping agent lanes, each change measured by one cost arm and one STRICT gate; the main session lands every patch (lane map: `port/dreamcast/docs/D367_SQUARE_PERF_PLAN.md`, "Current order and status"; every standing user decision: `port/dreamcast/docs/D367_THIRTY_FPS_ROUTE.md`).
- **How to resume:** use the shared skill `re4-dreamcast-d367`, which covers the harness, commit recipe, state files and decisions.
- **Historical:** the Astra/Sol (GPT-6) assignment and the D349-cutover milestone below are history. Current work is user-directed through that skill. Codex sessions follow the same lane, rules and commit procedure.

## Objective and architecture

Deliver normal boot/main menu/New Game and consecutive playable r100 -> r101 ->
r103, including required combat/events, HUD/audio, inventory, transitions, death
and retry. Cutscene presentation may be skipped through its qualified source
completion path. Room initialization or a scripted picture is not acceptance.

The north star is to beat D349's visual workload cost with the real recovered
game, while retaining source simulation/state and the proven native architecture.
The active milestone is [the r100 native cutover](port/dreamcast/docs/R100_NATIVE_CUTOVER_GOAL.md):
recovered GameCube source owns simulation/state; the existing D349 converter,
`.re4room` packages and native renderer own presentation. The live GX/ModelPart
bridge is fallback for unconverted content. GC and PS2 are first-class offline
visual inputs. D362 ends lossless scavenging; continue the complete cutover,
not bridge-cache tuning or another converter architecture.

## Context routing

Use [CLAUDE.md](CLAUDE.md) when resuming for current paths, baseline and active work.
Read supporting documents when the task touches their contracts:

- Rendering/asset cutover: [R100_NATIVE_CUTOVER_GOAL.md](port/dreamcast/docs/R100_NATIVE_CUTOVER_GOAL.md).
- Route, source systems and outstanding integration: [PLAYABLE_PATH.md](port/dreamcast/docs/PLAYABLE_PATH.md).
- Measurements: [REALTIME_PATH.md](port/dreamcast/docs/REALTIME_PATH.md).
- Resource ownership/loading: [R4_ASSET_RESIDENCY_PLAN.md](port/dreamcast/docs/R4_ASSET_RESIDENCY_PLAN.md).
- Visual candidates: [PS2_INSPIRED_DREAMCAST_PROFILE.md](port/dreamcast/docs/PS2_INSPIRED_DREAMCAST_PROFILE.md).
- A specific prior result: its checkpoint and implementation, not every historical log.

The active cutover contract supersedes dated next-task/gate language in historical
checkpoints. Keep their evidence and rejected results. Do not repeatedly read
all backlogs before routine edits or turn history labels into execution order.

## Implementation boundaries

Preserve source gameplay/AI/collision/camera, activation, progression, current
pose and attachments, material/texture animation, required alpha/depth/cull,
changing lights, audio, owner generations and retirement. Maintain one PVR/frame
owner and corrected actor facing. The historical prototype loop, sampled poses,
fixed camera and reduced encounter logic are not the recovered game's state.

Before designing a new Dreamcast rendering structure, converter, cache, package
or lifecycle mechanism, inspect the preserved D349 implementation and existing
port/dreamcast/tools, port/dreamcast/room, native texture/storage/lifecycle code,
and isolated completed experiments. Reuse or extend the proven mechanism unless
a documented source semantic or measured budget makes it unsuitable. Record why
an existing mechanism cannot be used before introducing a replacement.

Work through implementation, relevant checks, target execution and routine fixes
until the defined milestone works. A first patch, passing host tests or a commit
is not a stop instruction. Sol and Astra use the same contracts. Current user
assignment: Astra locks architecture/handoff, then GPT-6 Sol at Max implements.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lamb2k/re4dc](https://github.com/lamb2k/re4dc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
