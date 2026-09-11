---
trigger: always_on
description: Monoscopic 6DOF VR mod for **Project Sunrise** (offline Destiny 2, Season of Arrivals build), as a
---

# Destiny 2 Sunrise VR mod — project context

Monoscopic 6DOF VR mod for **Project Sunrise** (offline Destiny 2, Season of Arrivals build), as a
proof of concept. Engine internals live in `docs/RESEARCH.md`; read it before touching offsets.

## Layout

- `sunrise-vr/` — git clone of `stanuwu/Sunrise`, branch `vr`, `upstream` remote, pinned to master
  commit `7e3875d`. All mod code goes in `Sunrise/src/client/hooks/vr/`.
- `tools/` — the Sunrise installer and the official 0.3.2 DLL, kept for comparison and rollback.
- `docs/` — research and status. Project-level docs live HERE, not inside the fork, so nothing of
  ours can ever conflict with upstream.
- Game install: `C:\Games\Sunrise` (~93 GB, separate from any live Destiny 2).

## Scope (agreed with the user)

PoC, **no stereo**. Everything is monoscopic: one view rendered, same image to both eyes. Head
tracking 6DOF carries most of the sense of presence; stereo disparity matters mainly within a few
metres and these are big open spaces. The user knows the world will read as flat/infinitely far.

Phases: **F0.a** build from source · **F0.b** prove the camera pose can be written · **F1** OpenXR
+ 6DOF mono head tracking · **F2** controllers as a gamepad · **F3** weapons 6DOF.

F3 is in scope even though Sunrise has no enemies: it retires the hardest remaining risk
(decoupling the weapon viewmodel from the camera). Validate aim with a `world_lines` impact
marker, not with something to shoot.

## Fork discipline (option C)

All VR code in its own directory; upstream files get only additive lines. Current footprint into
theirs: **48 insertions, 5 files** (`Sunrise.vcxproj`, `graphics_renderer_lifecycle.cpp`, and three
in `hooks/teleport/`). The only non-addition is the `AdditionalIncludeDirectories` line, extended
with `vendor\openxr\include`.
Keep it that way — it is what makes `git pull upstream` merge cleanly. `Sunrise.vcxproj` lists all
931 sources explicitly with no wildcards, so new files must be added to it by hand.

The target build is pinned to a **fixed Steam depot manifest**, so reverse-engineered offsets can
never be broken by a game patch. Only Sunrise's own internals can drift.

## Build & test loop (Claude runs this solo)

1. `sunrise-vr\scripts\build.ps1` → `sunrise-vr\build\x64\Release\steam_api64.dll`
2. `sunrise-vr\scripts\deploy.ps1` → swaps it into `C:\Games\Sunrise\bin\x64\`
   (`-Restore` puts the stock DLL back)
3. `sunrise-vr\scripts\lib\GameIO.ps1` → dot-source for `Start-Game`, `Wait-GameWindow`,
   `Focus-Game`, `Send-GameKey`, `Send-GameText`, `Send-GameClick`, `Save-Shot`,
   `Invoke-GameSteps`, `Get-GameStats`.
4. Read the PNG for visual verification; read `C:\Games\Sunrise\bin\x64\Sunrise\logs\sunrise.log`
   for the structured log (note the `logs\` subdirectory). The game window appears on the user's
   screen — that is fine and expected.
5. Verification scripts, all in `scripts\testing\`: `Test-HandPose.ps1` (controller pose, every
   axis and rotation against hand-computed values, orbit is enough), `Test-WeaponPivot.ps1` (**the two
   regressions the rest of the suite walks straight through**: pure rotation must not translate the
   weapon, and artificial turning must not either; `-Solve` also measures the pivot constant by a
   least-squares solve rather than a search, and `Probe-Pivot.ps1` is the visual reconnaissance to
   run before believing it — the absolute placement moves the gun's rest position, and if it has
   left `weapon_shift.py`'s template box the test would measure a beautiful zero for the wrong
   reason), `Test-Weapon6DOF.ps1` (the F3
   matrix: hand moving with the head still, then head moving with the hand still; `-Quick` for just
   the decisive steps), `Test-BodyServo.ps1` (servo convergence, horizon stability, and that walking
   follows the gaze), plus the measurement helpers `weapon_shift.py` (template match, exact to a
   pixel), `diff_panel.py` (A/B difference sheets), `contact_sheet.py` and `weapon_metric.py`.
   **Compare captures taken seconds apart, never minutes**: Io's lighting drifts and the weapon has
   an idle animation, so a whole-frame metric across a long session has a noise floor of tens of
   pixels, while an A/B pair plus a null control pair is reliable.
6. `sunrise-vr\scripts\testing\Run-Orbit.ps1` is the whole boot-to-orbit-to-F9 cycle on the mock
   (about 4 min), with screenshots in `build\shots` and both logs dumped at the end.
   `Run-Ember.ps1` continues through Sunrise's Activity override into a mission (see gotchas).
   `scripts\diagnostics\` holds the loader forensics (code-page diff, symbol naming, probe DLLs).

`build.ps1` and `deploy.ps1` both kill the game first. To rebuild while a session is being used,
call MSBuild directly with the two overrides below; only the deploy needs the game gone.

**Two MSBuild overrides are mandatory and are NOT project changes:**

- `/p:PlatformToolset=v143` — the project asks for v145 (VS 2026); only VS 2022 BuildTools is here.
- `/p:PreferredToolArchitecture=x64` — without it MSBuild picks the 32-bit `HostX86` compiler,
  which dies with `error C1060: out of compiler heap space` on this codebase's C++20 templates.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thecosmictangerine-cloud/destiny-2-vr](https://github.com/thecosmictangerine-cloud/destiny-2-vr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
