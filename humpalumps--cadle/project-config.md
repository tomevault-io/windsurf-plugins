---
trigger: always_on
description: > Orchestrator: read `HANDOVER.md` first — it holds the live wave state, the exact commands, and what to do next.
---

# CADLE — browser FPS-RPG (Three.js)

> Orchestrator: read `HANDOVER.md` first — it holds the live wave state, the exact commands, and what to do next.

Target: **Destiny 2 moment-to-moment game feel** × **Final Fantasy XIV mystical look**. Three pillars, in order: graphics, performance, game mechanics. Everything judged against the real games.

> **SOURCE INVARIANTS (run first, takes one second): `node tools/invariants.mjs`.** It greps the source for the rules that encode bugs which have shipped repeatedly — the single pointer-lock path (`Input.lock`, plus the HUD hookup and the `synthetic`-only-under-`auto` guard), the grass emissive ceiling AND final-luminance cap, the grass roughness floor, vfx/enemy/viewmodel/prop intensity ceilings, and the bloom/exposure calibration pins. Run it before you start and again before you report; it needs no dev server, so it works even when the harness is contended. It is also enforced automatically: CI (`.github/workflows/checks.yml`) runs it on every push/PR, and the repo's `.claude/settings.json` Stop hook runs it at the end of every agent turn and blocks on failure. These rules exist because prose in this file kept being followed *literally* while a new code path reintroduced the same bug — do not edit `tools/` to satisfy them, fix the code, or say in your report that the rule is wrong.
>
> **The gate needs a MASK to be honest (2026-08-22).** `tools/inspect.mjs` captures one `mask-*.png` per burst
> from `PostFX._renderSkyMask()`: geometry green, sky and fog magenta, **ground cover RED**. `tools/blobcheck.py`
> scopes both of its tests to the red — that is what the decree is actually about — and ignores whatever the haze
> owns. Without this it fails on the sun through a treeline, on lantern flames and on loot beacons, and a gate
> that cries wolf is a gate nobody reads. **After ANY change to blobcheck.py run `python tools/blobcheck.py
> --selftest <a burst frame>`**: it paints synthetic bloom-balls onto the blades and asserts they are still caught.
> Coverage for emissives OFF the ground is `tools/invariants.mjs` (intensity ceilings), the hue-preserving aether
> cap in `src/enemies/materials.js`, and `HOT_TINT` in `src/vfx/Brush.js`.
>
> **REGRESSION GATE (mandatory): `node tools/gate.mjs` must exit 0 before any builder reports done; critics run it and ANY failure = automatic LOSE.** It mechanically enforces three decrees that have each regressed multiple times: (1) no washed-white blobs in the meadow, (2) no screen jitter (static camera must produce near-identical consecutive frames), (3) pointer lock engages on click and RE-acquires after exit (the mouse must never escape the window mid-play). If your change can affect any of these (grass/vfx/postfx/enemies materials; TAA/camera; input/HUD start-pause), run the gate FIRST, not last. Do not weaken gate thresholds — thresholds are orchestrator-owned.
>
> **THE VISUAL SIGN-OFF (user decree, 2026-08-28): ANYTHING THAT CAN BE SEEN MUST BE LOOKED AT, BY YOU,
> IN THE RUNNING GAME, BEFORE YOU SAY IT IS DONE — and before the gate is run, not after.**
> A gate is a regression alarm, not a quality judgement: `combatcheck` cannot see an opaque orange film
> that keeps its hue, `blobcheck` cannot see a nameplate floating 350 px above a creature's head, and no
> gate in this repo can see that a "seraph" is wearing the forgeknight's body. Every one of those shipped
> green and was caught by a person looking at the screen.
> The rule, mechanically:
>   1. **Capture it** — `tools/inspect.mjs` at the distances the thing is actually seen from (a landmark:
>      200 m / 40 m / 8 m; a creature: a burst while it MOVES; a HUD element: the state that draws it;
>      a material: noon AND night). A single still cannot show you an animation, and one distance cannot
>      show you a landmark.
>   2. **READ THE PNG** — actually open the image. "The code is correct" is not a visual check, and
>      neither is a passing metric.
>   3. **Say what you saw** in your report, name the file, and say what you would still dock points for.
>      A report that only quotes gate output has not been visually signed off.
> Two traps this decree exists for: a metric that is green while the screen is wrong (see the wave-6
> verdicts), and a probe that measures the wrong place — `goto(id)` drops you at a region's HEART, which
> is inside the landmark clearance where ground detail is deliberately suppressed, so "I looked down and
> saw nothing" may be your camera, not the build. Shoot off-centre too.
>
> **THE THREE-GATE SIGN-OFF (user decree, 2026-08-23): nothing is "done" until it passes GRAPHICS, PERFORMANCE and GAME MECHANICS.**
> Reporting a feature complete without all three is the same failure as reporting a shader done without a screenshot.
> 1. **Graphics** — `node tools/gate.mjs` exits 0 (invariants + blobcheck + jitter + pointer lock at q=high AND q=low), plus `tools/inspect.mjs` screenshots of every new visual element. A visual you did not look at is not done.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Humpalumps/Cadle](https://github.com/Humpalumps/Cadle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
