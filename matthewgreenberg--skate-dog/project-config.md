---
trigger: always_on
description: React 19 + @react-three/fiber + three.js. A boy rides a dachshund through a
---

# Skate Dog

React 19 + @react-three/fiber + three.js. A boy rides a dachshund through a
pastel skatepark. Entirely procedural — no image files, no network fetches.
Textures painted into canvases at load, geometry built in code, props and
foliage baked into InstancedMeshes once and never touched.

Assets: `public/boy.glb`, `public/dog_compressed.glb`. No animation clips —
both driven every frame by pose tables (`player/boneRig.js`). Re-decimate after
any asset change:

```
gltf-transform simplify in.glb tmp.glb --ratio 0.2 --error 0.002
gltf-transform draco tmp.glb out.glb    # simplify decodes draco; must re-encode
```

Decoders in `public/{draco,basis}`, intro font in `public/fonts` — drei/troika
default to gstatic CDNs and nothing here touches the network.

## Layout

```
src/game/
  palette.js        art-direction contract — C (albedo), M (roughness), LIGHT, TONE, RAMP
  photo.js          deterministic capture poses
  store.js          useGame = UI state; P = per-frame state (never React state)
  goals.js          the run's challenge table
  level/
    levelData.js    authored layout; renderer AND colliders read it
    rails.js        grind paths (drawn tubes + derived wall/planter/bench lips)
    decals.js       world-space floor detail quads over a texture atlas
    colliders.js    simplified collision built from levelData
    parkGeometry.js plaza / grass / ramp meshes
    bowlGeometry.js analytic bowl — the drawn surface IS the ridden surface
    textures.js     every procedural map: albedo, normal, roughness, baked AO
    foliage.js      plant generation, pure data -> instance rows
    levelEdits.js   editor contract shared by Editor.jsx and EditorPanel.jsx
  components/       Game, Lighting, Skatepark, Props, Player, Effects, UI, Editor
  audio/AudioManager.js   fully synthesised SFX, no files
  player/           PlayerController.js (movement/tricks/grinding), boneRig.js
tools/              capture harness
ref/                reference stills the art is measured against
```

## Palette

> A shadowed surface reads at (0.62, 0.62, 0.78) of its sunlit self → a golden
> key plus a cool violet ambient **whose sum is neutral white**. The scene is
> warm-*painted*, not warm-*lit*.

- `C` is albedo under white light. Never pre-warm it. **Too orange = the light
  is wrong, not the paint.** An albedo must BE the reference's sunlit reading.
- Shadow tints come from `SHADOW_TRANSFER`, never a grey multiply.
- Per-instance variation samples `RAMP.*`; HSL jitter reads as noise.

## The run

A 2:00 clock you extend by playing: bone or challenge +15s, bail −5s, zero →
scorecard (`RUN_TIME`/`TIME_BONUS`/`TIME_BAIL` in store.js). Level blobs may
carry `rules: { time, goalIds, timeBonus, subtitle }`; `setRunRules` syncs
`P.timeLeft` + UI clock, `activeGoals()` is the one filtered list every HUD
count reads. Time is the only resource — there is no `lives`.

- **The clock lives on `P.timeLeft`, not the store.** GameLoop mirrors it only
  on whole-second change. `addTime()` writes both.
- **Every goal is detected from an event the controller already emits** — a goal
  that owns its own timer/collider/probe drifts from the scorer. Two score tiers
  poll at 4Hz.
- **`complete()` must be idempotent** — every predicate is on a repeating event.
- **Grind payout does not go through `award()`** (double-count); grind
  challenges listen for `'trick'`.
- **`P.inBowl` is a SURFACE flag** — false while airborne over the hole.
- Restart is `resetPlayer()` + `resetGoals()` + `restart()`; the `runId` bump
  remounts Bones/Letters/Cans and **a remount IS the reset** for their `useRef`
  "already got" flags.
- **The bowl is a hole with no side walls.** `resolveCollision` only pushes x/z,
  so `clampToBowl()` runs after every integration in `step`/`stepAir`/
  `stepBail`. POSITION ONLY — zeroing `vel.y` there ate descent momentum.
- **Personal best is per LEVEL, one writer** — `highScore.js` keys
  `skatedog.best` by level id, `endRun()` is the only writer. Reads try/catch'd,
  `localStorage` reached lazily (`ls()`) so node can run the checks. Cans
  progress under `skatedog.canBest`.

**Bail is two bodies.** The boy is thrown clear (`P.bailBoyPos/Vel`, world
space) keeping the momentum the dog loses. Below `SPLAT_SPEED` (3.6) he hops
into the `land` crouch; above it (`P.bailSplat`) he belly-slides — `sec.lie`
pitches him about his feet origin, LIE 1.5 stops short of π/2. His origin uses
`P.pos`'s paw-line convention, so grounding is a clamp against `groundHeightAt`.
`sec.eject` blends the world offset in through the inverse of `P.quat` while
fading the `backY` mount out; both SNAP to 0 at bail end (respawn is a teleport).

- **Dog tumble clearance is angle-aware** (`bailClearance()`): pivoting about the
  paw line, a nose-down pitch needs half the dog's LENGTH, an inversion needs the
  back height.
- Tumble→settle is TIME-based (`BAIL_SETTLE`), not contact-based — the clearance
  floor rises faster than the pop climbs, so a contact latch kills the tumble
  before it turns once. Settle damps pitch/roll to the nearest 2π (never π).
- `tumbleW`/`rollW` scale with crash speed; `BAIL_HITSTOP` 0.07s freezes both
  bodies (world/camera/FX keep moving); contact bleeds spin at a RATE (1.1/s) —

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MatthewGreenberg/skate-dog](https://github.com/MatthewGreenberg/skate-dog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
