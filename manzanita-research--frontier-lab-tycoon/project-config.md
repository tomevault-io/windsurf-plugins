---
trigger: always_on
description: A browser tycoon game (React + react-three-fiber) about running a frontier AI lab. Read `docs/DESIGN.md` before writing game code. It defines the module boundaries every task relies on.
---

# Frontier Lab Tycoon: agent notes

A browser tycoon game (React + react-three-fiber) about running a frontier AI lab. Read `docs/DESIGN.md` before writing game code. It defines the module boundaries every task relies on.

## House rules

- The mission-control charter wins over this file. Your prompt says where it lives (it's readable from the Mini; on Modal, your prompt restates the parts that matter).
- **Do the heavy work on Modal, not the Mini.** Installs, dev servers, builds, tests and screenshots happen on cloud machines. `.bb-env-setup.sh` skips `pnpm install` on macOS on purpose. See `docs/modal.md`.
- One thread = one branch = one worktree. Branch names: `flt-<n>-<slug>` (for example `flt-6-agents`).
- Every task has one label: `ship`, `explore` or `experiment`. The first playable is `explore` work.
- Stop dev servers and watchers before you end a turn, unless you are handing someone a live link right now.
- **Parody names only.** No real companies, products, or real people. Everyone should get the joke without anyone being named.

## Commands

```sh
pnpm install          # setup does this on Modal
pnpm dev              # vite on 0.0.0.0:5173; share with `bb connect expose 5173`
pnpm test             # vitest (sim + content tests)
pnpm typecheck
pnpm build            # tsc + vite build into dist/
pnpm check            # guard + all three: run before every PR
pnpm guard            # fails if a tracked file names private infra (bb paths, hosts, project/thread IDs)
pnpm shots            # before/after screenshots of main vs your branch (see below)
pnpm shot <url> <png> # one headless screenshot (Playwright + SwiftShader)
```

### Before/after screenshots: `pnpm shots`

```sh
pnpm shots                                   # 4 standard scenes: overview, inspector, event, phone (~2.5 min cold on a Modal builder)
pnpm shots --scenes overview,ops,night --diff --out docs/img/flt-99
pnpm shots --skin frontier-95                # a skin on both builds; `--skin all` = a gallery of every skin
pnpm shots --list                            # every scene and set
```

It builds `main` in a temporary worktree (cached by commit in `shots/.cache/`) and your checkout, serves both, and writes `<out>/{before,after,compare}/<scene>.png` plus `report.md`, whose table you paste into the PR body. `shots/` is gitignored scratch; pass `--out docs/img/<task>` and commit that directory so the PR's image links resolve. A scene is data (`?seed`, `?moment=`, `?zoom`, a few steps), so a task adds its own in `scripts/shots.scenes.json`. The game is paused and the news ticker is parked, so a build compared with itself differs by about 0.2% of pixels; a scene under 0.5% is reported as unchanged. Every capture and every compare image is checked for blank or single-colour output (a dark frame, a canvas that never drew, images that didn't load), and the run exits 1 with a loud banner if any is: never paste those as evidence. `pnpm shots --verify <png|dir>` runs the same check on existing images. Run one at a time (it is CPU-heavy on 1 vCPU).

## Architecture in one breath

- `src/sim/`: pure TypeScript, deterministic, **no React, no three, no DOM, no Math.random** (use `src/sim/rng.ts`). A fixed-step `tick(state)` drives everything. Unit-test it. **No raw `Math.sin`/`cos`/`atan2`/`exp`/`log`/`pow`/`tanh`/`hypot` or `**` either:** engines round them differently, so use `src/sim/dmath.ts` (`dsin`, `datan2`, `dpow`, `dhypot`, `sq`, ...); `sim.test.ts` fails otherwise, and `pnpm engines` replays the goldens in Node and three browser engines (FLT-106).
- `src/content/`: data only (buildings, research, rival labs, events, headlines, thoughts). Adding a joke should never need an engine change.
- `src/render/`: react-three-fiber scene. Reads sim state, never mutates it except through store actions.
- `src/ui/hud/`: the 2D UI's host. `hudViewModel(snapshot)` (`vm.ts`) turns the snapshot into a plain-JSON `HudVM`; `types.ts` is the whole modding contract (`HudVM` + `HudActions`). `src/skins/`: the skin system (tokens, slots, registry, schema) and the six skins (Frontier 95 is the default; the other five are `unlisted` from the picker, which shows Frontier 95 and Classic). **The 2D UI is skinned: read `docs/SKINS.md` before touching it**, put UI in a slot (base or a skin's), and never import `src/sim/**`, the store or three from `src/skins/**` (a test fails if you do). `src/ui/juice/`: sky and photo-mode plumbing; `src/ui/WorldOverlay.tsx`: labels pinned to the scene.
- `src/render/fx/`: the juice layer (camera director, particles, day/night, photo mode). It only reads the World; see the last section of `docs/ARCHITECTURE.md`.
- `src/render/crt/` + `src/ui/juice/{crt.ts,Tube.tsx,crt.css}`: the picture tube (FLT-73), a ported MIT CRT shader on the canvas plus a CSS tube over the DOM. **Off in the game** (FLT-70: `GAME_CRT = false` in `src/render/crt/state.ts`, so no setting and no `?crt=`); the shader is used only on the box's beige PC (`src/intro/stage/Kiosk.tsx`). Credits in `docs/CREDITS.md`; reuse on a texture per `src/render/crt/README.md`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Manzanita-Research/frontier-lab-tycoon](https://github.com/Manzanita-Research/frontier-lab-tycoon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->
