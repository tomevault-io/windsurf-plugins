---
trigger: always_on
description: A Vite + TypeScript or JavaScript app, but several things do **not** work the way
---

# IWSDK project

A Vite + TypeScript or JavaScript app, but several things do **not** work the way
a normal Vite project works. Read this section before assuming anything.

Deeper material is installed in each selected harness's native path-scoped rule
format. Harnesses that use `AGENTS.md` also receive equivalent nested files near
the code they govern. Read the applicable scoped instructions before working in
that area.

## What is not standard Vite

**`iwsdk.config.json` is the project authority, not `vite.config.ts`.** It selects
the active scene, the asset module, the component module, and all XR/world
features. `vite.config.ts` only wires the plugin. Editing `iwsdk.config.json`
restarts Vite in place; the managed window remains open and its command bridges
reconnect automatically. Wait for `npx iwsdk dev status` to report
`browserCommandReady: true` before issuing browser-backed commands.

**`virtual:iwsdk-project` is a virtual module**, not a file. `src/index.ts` or
`src/index.js` imports it and passes it whole to `World.create()`. Do not
hand-build that options object.

**The dev server is CLI-managed.** Use `npx iwsdk dev up` (or `npm run dev`), not
`vite`. It launches a managed browser that hosts the MCP command bridge.
`--no-open` intentionally starts only the server: status reports
`browser.status: "not_launched"`, and browser-backed commands fail immediately
with `browser_not_launched` plus restart guidance. One managed window hosts two
roles, editor and runtime.

**`src/assets.ts`/`src/assets.js` is evaluated twice, in two different JS
realms** — once by the app runtime, once by the editor. It must be deterministic
and side-effect free: no `World`, no DOM, no timers, no reliance on shared object
identity.

**Static geometry lives in TypeScript, composition lives in JSON.** Scene JSON
never declares URLs, geometry or materials — only manifest IDs.

**Components must be declared in a system-free module** and exported from
`src/components.ts`/`src/components.js` via `defineComponents()`. The editor
imports that same manifest to build its inspector, so a component that is not in
it cannot be authored in scenes.

**Import Three.js from `@iwsdk/core`, never from `three`.** `@iwsdk/core`
re-exports all of Three; importing `three` directly creates a duplicate instance
and subtle breakage. Exception: `import type { GLTF } from 'three/addons/...'`.

## Traps that produce silent failures

| Trap                                                          | Symptom                                                                  | Fix                                                                                 |
| ------------------------------------------------------------- | ------------------------------------------------------------------------ | ----------------------------------------------------------------------------------- |
| `locomotion: true` with no `LocomotionEnvironment` on a floor | player falls through the world                                           | add the component to a walkable surface                                             |
| Scene origin left occupied                                    | player spawns inside your geometry                                       | the player origin is `0,0,0` unless the scene authors `player.transform`            |
| Scene JSON with `imports`                                     | authoring preview works, but editable open and runtime load are rejected | run `npx iwsdk scene flatten` once, then edit the flat output                       |
| `entity.destroy()`                                            | GPU memory leaked                                                        | use `entity.dispose()`                                                              |
| `setValue` on a Vec2/Vec3/Vec4/Color field                    | throws in elics 3.4.x                                                    | use `entity.getVectorView(...)`                                                     |
| Environment component on a non-root entity                    | silently ignored                                                         | `DomeGradient`/`IBLGradient` go on the level root only                              |
| Environment prop changed without `_needsUpdate`               | change ignored                                                           | set `_needsUpdate` after writing                                                    |
| `ScreenSpace` given numbers                                   | clamped with a console warning                                           | it takes CSS strings: `'400px'`, `'25vw'`                                           |
| `@iwsdk/reference` MCP tools in an old or `--no-install` app  | queries report warmup required                                           | run `npx iwsdk reference warmup`; fresh installed scaffolds do this during creation |

## Verify before you claim it works

**Always `npx tsc --noEmit` before testing.** Type errors stop systems
initialising without necessarily logging anything in the browser.

Then check the right status for the task — these are not interchangeable:

- scene/editor work → `scene_get_state`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [facebook/immersive-web-sdk](https://github.com/facebook/immersive-web-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
