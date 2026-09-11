---
trigger: always_on
description: > **Resuming a session? Read `docs/FINDINGS.md` first.** It records everything
---

# art-of-sim-rally — Codex working notes

> **Resuming a session? Read `docs/FINDINGS.md` first.** It records everything
> verified on disk about the game. Do not re-derive it, and do not describe a
> component as working if the status table below says it has never run.
> Open defects live in `docs/KNOWN-ISSUES.md` — check there before treating a
> symptom as new.

## Repository Purpose

Turn art of rally into a sim rig game: force feedback, Forza-compatible UDP
telemetry, bonnet camera. The game's physics are already a real load-sensitive
tire model; this project connects that simulation to a wheel, a dashboard and a
viewpoint.

The founding discovery: **art of rally ships a complete force feedback
implementation that never runs, because `UnityForceFeedback.dll` was left out of
the build.** The managed `ForceFeedback` class P/Invokes seven entry points from
a module that does not exist in the install. The clean-room implementation of
it now lives in dbce-wheel-mod-toolkit as `WheelFfb.dll`, vendored here and
shipped under the name the game's dead code P/Invokes.

Unlike the sibling `dbce-mod-toolkit` (private by design), this is intended to
be **public and shareable**. Keep it that way: no game assemblies committed, no
third-party binaries, nothing that would force the repo private.

## Repository Structure

| Path | Contents |
|---|---|
| `src/ArtOfSimRally.Mod/` | The whole mod. One project, one assembly. `Main.cs` is the only loader-aware file. |
| `lib/toolkit/` | **Vendored** from dbce-wheel-mod-toolkit (pinned by `VERSION`; refresh with `tools/Sync-Toolkit.ps1`): `native/WheelFfb.dll` (shipped as `UnityForceFeedback.dll`, the name the mod P/Invokes) and `dotnet/Dbce.Wheel.Ffb.dll` / `Dbce.Wheel.Telemetry.dll`. The native source and the encoder live in that repo now. **These binaries are committed** — see the gitignore note under Findings. |
| `lib/umm/` | UnityModManager.dll + 0Harmony.dll, extracted locally, **never committed**. |
| `tests/` | Executable consumer regression, CameraTuning, WheelInput, GameState, lifecycle, telemetry, Signals, Support, recorder and hook suites; Python replay/evidence tests. Run through `tools/testing/Test-Rc.ps1`. |
| `tools/` | `Sync-Toolkit.ps1` (toolkit pin), `package/` (release zip), `installer/` (the double-click installer), `dinput-enum/` (lists DirectInput devices without launching the game). |
| `docs/OVERNIGHT-QUEUE.md` / `docs/USER-FEEDBACK.md` | Prioritized follow-up work, user reports and unsent support drafts. |
| `docs/KNOWN-ISSUES.md` | **The defect register.** Open, resolved and will-not-fix, with severities. Read before diagnosing anything. |
| `docs/TROUBLESHOOTING.md` | User-facing fixes by symptom; the Fanatec section is the most-needed page. |
| `docs/` | FINDINGS, FORCE-FEEDBACK, TELEMETRY, CONTROLS, CAMERA, ROADMAP, RELEASING |

## Status (2026-09-09) — do not overstate this

Released and installed **0.2.4** (2026-09-09 UTC) following the owner's installed RC5 acceptance:
"I think everything looks good. Let's ship another release." RC5's preserved drive
log has one normal manager initialization and zero initialization errors,
connection failures or exceptions. KI-20's menu polling/error flood is resolved
on this rig; the T300 user's unrelated intermittent slowdown remains unconfirmed.

Production source and toolkit are unchanged from RC5 (`5701ebb`): camera keys/save
retry (FR-1/KI-14), direct-input recovery/Flip/live values (KI-15), CameraMod
isolation (KI-18), telemetry units/local axes (KI-16), bounded support logs and
frame aggregates (KI-19 diagnostics). Toolkit **v0.12.0**, native **0.5.0**; shared
wrapper, AxleForceCurve@1 and telemetry are consumed in production. No force tune,
new wheel effects, physics or assist behavior changes.

RC5 and final 0.2.4 pass all 16 local automated checks. Published ZIP/checksum
were downloaded and verified; six installed payloads/native copy match and settings
are unchanged. Tag/source `dc14fe7`; final artifact/publication evidence is
recorded in docs/reviews/2026-09-09-release-0.2.4.md. The full attended matrix,
final-labelled drive, motion/shaker comparison and TSS/Fanatec/combined-camera-mod
checks remain pending. Owner acceptance does not mark those cases passed.
Read docs/LOCAL-DEPLOYMENT.md for the current installed identity and receipt;
the Stream Deck Steam 550320 key targets that installation. Keep it current under
the standing deployment rules below. "Verified" means confirmed on the owner's
MOZA R12 rig unless stated otherwise.

| Component | State |
|---|---|
| Force feedback | Verified. Front-axle lateral force × pneumatic trail (reference 11,500 N after two retunes), faded out below 12 km/h, re-acquires the wheel after alt-tab. Sign confirmed on a MOZA R12; the MOZA R5 one-sided inversion fixed by user report. |
| Steering fixes, bind-any-device, glyph text fallback | Verified. |
| Shifter (sequential + H-pattern), read directly from the device | Verified by users. |
| Bonnet + bumper cameras | Owner RC6 camera smoke passed; offline handback tests pass. The complete stock/replay/finish transition matrix remains pending. Issue #1 reporter separately says unplugging a PS5 pad resolved their symptom (KI-1/KI-2). |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [d-b-c-e/art-of-sim-rally](https://github.com/d-b-c-e/art-of-sim-rally) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-11 -->
