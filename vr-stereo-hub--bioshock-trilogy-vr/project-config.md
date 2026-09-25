---
trigger: always_on
description: Native VR mod for BioShock Remastered, BioShock 2 Remastered and BioShock Infinite: a 32-bit DLL
---

# bioshock-vr - Claude session guide

Native VR mod for BioShock Remastered, BioShock 2 Remastered and BioShock Infinite: a 32-bit DLL
injected via an `xinput1_3.dll` proxy shim, hooking the game's D3D11 renderer and the engine's
camera/aim paths, driven by an OpenXR session (Quest 3 via Virtual Desktop/VDXR or Steam
Link/SteamVR). Architecture is a game-agnostic VR core plus per-game adapters; the adapter
registry picks by host exe name (`BioshockHD.exe` -> bioshock1r, `Bioshock2HD.exe` -> bioshock2r,
`BioShockInfinite.exe` -> bioshockinf).

The two remasters are Vengeance (UE2.5). **BioShock Infinite is Unreal Engine 3 build 6829** - a
different engine, so no number transfers, and even shapes are suspect.

**`main` is THE branch: all three mods live there, all three work, all three shipped in v0.8.0
(2026-08-13, session 59).** Branch every new session off `main` and merge back to it - the old
per-game branches (`bioshock-2`, `bioshock-infinite`) are historical and fully contained in
`main`; do not start work on them.

## Hard rules

- **NEVER commit game-derived content**: no decompiled UnrealScript, no extracted assets, no
  RenderDoc captures, nothing from the game folder. `tools/uscript/` is gitignored for a reason.
  Summarize findings in the per-game ENGINE_NOTES instead of pasting game code.
- **Commit messages**: plain conventional commits (`feat:`/`fix:`/`docs:`/`build:`/`tools:`/`chore:`),
  imperative, subject ≤72 chars. No trailers.
- **32-bit (Win32) only.** All three games are x86. The CMake guard will stop you; don't fight it.
- **No code from UEVR** (all-rights-reserved - concepts only). REFramework (MIT) may be adapted
  with an attribution comment in the file.
- Engine addresses/signatures live ONLY in the per-game `src/game/<title>/patterns.cpp/.h`, and
  every one is documented in that game's `docs/<game>/ENGINE_NOTES.md` with its derivation method.
  NEVER copy a number between games - same engine tree, different link; derive fresh.
- **BS2 is NOT bound by BS1's methods** (user directive, 2026-07-29 session 24). Much of BS1's
  machinery - the foreground/viewmodel FOV counter-modeling, weapon scaling compensation, aim-seam
  workarounds - exists because of BS1-specific limitations, not because it is the right design.
  If BS2's build affords a better or more native method (it has a native FOV slider, native
  dual-wield, ProcessEvent-by-name event hooking), USE THE BETTER METHOD. Before porting any BS1
  compensation machinery to BS2, first test whether BS2 needs it at all.
- **The same gate applies to Infinite, harder** - it is a different engine (UE3 build 6829), so
  no number transfers and even shapes are suspect. Check what UE3 does natively, test whether the
  BS1/BS2 defect even EXISTS, and only then port compensation machinery, and only the parts proven
  necessary. A mono screenshot is not a sufficient check for a lens question.
- **NEVER run BioShock Infinite while BioShock 2 is running** (user directive, 2026-07-31
  session 34). BS2 development runs in parallel and only one game can own the headset at a time.
  Check `Get-Process Bioshock2HD` before any test that launches or drives Infinite; if it is
  running, WAIT or POSTPONE the test - do not close the other game. Building, installing,
  packaging and tailing logs do NOT contend and must keep working while BS2 runs. The `-Game bsi`
  harness scripts enforce this via `tools/lib/assert-no-conflict.ps1`.
- **KEEP THE PER-GAME MODS DECOUPLED. Duplicate code is fine** (user directive, 2026-07-31
  session 34; the same was said for the BioShock Infinite mod). Copy a BS1 behaviour into
  `bioshock2r/` and adapt it rather than promoting it to `src/core/` or parameterising the BS1
  version - BS1 is the headset-accepted baseline and must not be put at risk to serve BS2, and
  BS1 regressions cost headset time to even detect. Put something in `src/core/` only when it is
  genuinely game-agnostic AND new; if a core change is unavoidable, keep it purely additive so
  no BS1 path changes behaviour. Consolidation and de-duplication are deferred to a dedicated
  "healing" session in the polish milestone.

## Session protocol

- **START**: read `docs/STATUS.md`, then the current milestone in `docs/ROADMAP.md`, then
  `git log --oneline -10`. **ALWAYS BRANCH FROM `main`** - every game ships from it since
  v0.8.0; there is no per-game branch to hunt for. **Working on Infinite?** Same `main`, but
  the ladder is `docs/bioshockinfinite/ROADMAP.md` (milestones I0-I11 after the 2026-08-05
  BS2-shaped restructure), which is separate from M0-M10.
- Touching engine internals? Read the game's `docs/<game>/ENGINE_NOTES.md` first
  (`docs/bioshock1/`, `docs/bioshock2/` or `docs/bioshockinfinite/`). New findings go there, in
  the same commit as the code that uses them.
- **Validate in the SIMULATOR before handing a build to the user.** `tools\xrsim-launch.ps1`
  runs the game against `bvr_xrsim32.dll`, a simulated 32-bit OpenXR runtime that presents as a
  Quest 3, so head/hand poses, every controller button, deterministic frame stepping and per-eye
  compositor captures - **including the quad layers a window screenshot can never show** (the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VR-Stereo-Hub/bioshock-trilogy-vr](https://github.com/VR-Stereo-Hub/bioshock-trilogy-vr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
