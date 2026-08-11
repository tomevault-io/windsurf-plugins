---
trigger: always_on
description: A three.js app that flies a camera from orbit onto a landmark on Google's
---

# Working on this codebase

A three.js app that flies a camera from orbit onto a landmark on Google's
photorealistic 3D tiles and records the flight to MP4. Read the
[README](README.md) first for what it does; this file is how it is built.

No framework, no build step beyond Vite, no state library. Plain ES modules,
plain DOM.

## Commands

```bash
pnpm install          # also vendors the sky binaries into public/assets
pnpm dev              # vite on :5180
pnpm build            # -> dist/
pnpm assets           # re-vendor public/assets by hand
```

There is no test suite and no linter. Verify by running it: the flight is the
test, and `world.seek( t )` will step it without a clock (see *Verifying a
change*).

## Layout

```
src/
  main.js                 wiring only: UI ↔ world ↔ recorder. No logic.
  sun.js                  NOAA solar position, the daylight solve, light presets
  assets.js               local-first asset resolution, CDN fallback
  tilesAuth.js            Cesium Ion / Google Maps key resolution + verification
  camera/
    Rig.js                geographic camera rig, singularity-free
    shots.js              the five moves
    easing.js             curves
  world/
    World.js              renderer, tiles, atmosphere, clouds, the loop
    EffectAdapter.js      pmndrs postprocessing → renderer.setEffects()
    TileCreasedNormalsPlugin.js  local three addon — minify-safe worker alias
  record/Recorder.js      frame-locked WebCodecs capture
  data/landmarks.js       the landmark table, plus text → target
  data/mapLinks.js        pasted map links and codes → lat/lon, offline
  ui/UI.js                all of the DOM
  ui/Venue.js             the conference mark, and its card over the venue
scripts/copy-assets.mjs   vendors the atmosphere and cloud binaries into public/
```

## How a flight works

```
UI.on.go
  └─ main.launch()
       ├─ world.prepareShot()      solve the sun → derive the end bearing →
       │                           build the shot sampler
       ├─ world.preroll()          ease to the top of the shot while tiles stream
       ├─ world.settle()           pump the tile pipeline until quiet
       ├─ world.autoExpose()       meter 5 points, build an exposure curve
       └─ world.play()             state = 'flying'; the rAF loop samples the
                                   shot by shotTime and applies it to the rig
```

Recording replaces the last three steps with `Recorder.record()`, which drives
`world.seek( frame / fps )` + `world.render()` itself on a virtual clock.

The state machine is `idle → loading → flying → done → free`, and `returnToIdle`
from any of them. `free` is the only state where the user owns the camera; there
the rig reads the camera back instead of driving it.

## The load-bearing decisions

Change these only on purpose.

**`three` is pinned to `0.185.1`.** Later builds lost the depth texture on the
output render target, and the atmosphere reconstructs world position from depth.
It fails silently — the sky just goes wrong.

**Two upstream shaders are patched at runtime**, in `patchSkyRayMiss` and
`patchStarDepth` at the bottom of `World.js`. Both patch *source strings*, not
compiled programs, because `postprocessing` reassembles an effect's material
whenever the pass updates. Both return whether the match succeeded, and
`_applyAltitudeLook` has a fallback for the sky one. If a takram upgrade
rewrites those functions the console says so — do not let it fail quietly.

**Distance is always interpolated geometrically** (`mixLog`), never linearly.
Over four decades a linear interpolation reads as a stall followed by a slam.

**The camera basis is built analytically, not with `lookAt`.** The interesting
shots point straight down, which is exactly where `lookAt` degenerates. See the
header of `Rig.js`.

**Azimuth is derived from the sun, not chosen.** A given (elevation, azimuth)
pair is usually unreachable at a latitude; any elevation the sun visits there is
reachable from *some* direction. So `sun.js` solves elevation exactly and the
camera's final bearing falls out of the preset's `gamma`.

**The bearing is held north-up while the Earth is still a globe**, then released
over all the time that is left. `createShot` solves the `t` at which the shot
crosses `NORTH_HOLD_DISTANCE` and multiplies the authored azimuth progress by a
gate delayed to there. The hold is keyed to *distance* because `t` does not know
how high the camera is — the same t=0.2 is 5 000 km into a descent and 200 km
into an orbit. The release is keyed to *time* because the suppressed sweep has to
be given back somewhere, and handing it back over a second distance band spends
it crossing a few hundred kilometres: a whip pan, measured at 0.8-1.3 frame
widths a second, high up where there is nothing to whip past. Multiplying rather
than overlaying a counter-turn is what keeps the sweep from reversing mid shot;
the gate is exactly 1 with slope 0 at t=1, so the landing bearing and the exit
rate both survive.

**Shots end with live rates, and the landing carries them.** Do not "fix" a
non-zero terminal rate — it is the hand-off. `shot.settle( baseTime )` reads the
exit rate of every channel and relaxes it, azimuth easing into a perpetual

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Makio64/threejs-cinematic-world-zoom](https://github.com/Makio64/threejs-cinematic-world-zoom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
