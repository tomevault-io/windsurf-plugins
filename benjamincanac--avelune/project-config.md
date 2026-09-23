---
trigger: always_on
description: Multiplayer fantasy world. **Nuxt** (nightly) + **TresJS** (three.js) on the client, **Nitro v3 native WebSockets** on the server, deployed to **Vercel**. It exists to demo two things: the Vercel WebSocket upgrade under a real 20 Hz authoritative game loop, and an AI NPC (the Oracle) that reads live game state. Everyone shares one persistent world: a hand-authored walled town at the centre, protected, and open land around it that players terraform and build on. The world is chunked and streamed;
---

# Avelune

Multiplayer fantasy world. **Nuxt** (nightly) + **TresJS** (three.js) on the client, **Nitro v3 native WebSockets** on the server, deployed to **Vercel**. It exists to demo two things: the Vercel WebSocket upgrade under a real 20 Hz authoritative game loop, and an AI NPC (the Oracle) that reads live game state. Everyone shares one persistent world: a hand-authored walled town at the centre, protected, and open land around it that players terraform and build on. The world is chunked and streamed; edits are validated by the server and persisted per chunk. The plan and its decisions are in [.claude/OPEN-WORLD.md](.claude/OPEN-WORLD.md).

Roadmap / status is [.claude/ROADMAP.md](.claude/ROADMAP.md) — the source of truth for what's done and next. Keep it current as work lands.

## Commands

- `pnpm dev` — dev server (port 3000 is occupied on this machine; use the preview harness / autoPort)
- `pnpm typecheck` — `nuxt typecheck` (vue-tsc)
- `pnpm lint` / `pnpm lint:fix` — ESLint
- `pnpm test` — vitest: every `scripts/*-test.ts` suite (shared physics, terrain, world, building, chunk-store, the live surface, the per-chunk index, characters and their animations, moat, ramparts, fountain, voice, the upgrade's origin gate, the socket's hold on its invocation, audio, graphics presets, icon names)
- `node scripts/ws-test.mjs ws://localhost:<port>/api/ws` — protocol test (two clients create characters over `/api/auth`, then assert `welcome`/`chunk`/`state`/`terrain`/`remove`/`chat`/`pong`/`leave`/`kicked` frames)
- `pnpm exec jiti scripts/spawn-bots.mjs --url http://localhost:<port> --count 30 --dig` — load test; `AVELUNE_TICK_LOG=1` on the server logs tick timings
- Blender is headless: `"/Applications/Blender.app/Contents/MacOS/Blender" --background --python scripts/<x>.py -- <args>`

Package manager is **pnpm**.

## Architecture invariants (load-bearing)

1. **Gameplay-affecting code lives in `shared/utils/`.** Anything touching player position, collision, elevation, or edit validation must go in the shared modules (`maze.ts` physics, `world.ts` chunks and mutations, `building.ts` edit rules) so the authoritative server (`server/utils/game.ts`) and client prediction call the *same* functions and never disagree. Never fork physics into a component or the WS handler. This is why a client-side physics engine (`@tresjs/rapier` and friends) does not fit: it could never be the authority.
2. **The server is authoritative.** It runs a fixed **20 Hz** tick loop and validates every action (grounded jumps, dash cooldowns, chat length, every terraform/build/demolish against the shared rules and a rate limit). Positions are never accepted from clients. Clients predict; the server decides.
3. **One world, chunked and streamed.** 32×32 chunks of 32 tiles hold a corner heightmap, a surface raster and placements. The town is hand-authored in `shared/data/courtyard-*.json`, seeded into the town chunks at its original coordinates; the protected footprint (`PROTECTED_FOOTPRINT`: moat ring plus the gate bridge's landing) refuses player edits, and `GATE_APPROACH` holds the strip between the bridge and the spawn open so nobody can wall or dig the way in shut; everything else is generated deterministically on first touch and then owned by the persisted store (Upstash Redis, in-memory when unset). Store keys are scoped by **realm**, one world per deployment region (`VERCEL_REGION`, override `AVELUNE_REALM`, see `shared/utils/realm.ts`), so regions never race each other's chunks. Only the server builds the world; clients receive chunks and mirror every edit with the same shared `apply*` functions.
4. **Wire protocol** is the `t`-keyed discriminated unions in `shared/types/game.ts`. Client→server: `move`/`action`/`chat`/`ping`/`terraform`/`build`/`demolish`/`voice`. Server→client: `welcome`/`join`/`leave`/`state`/`chat`/`kicked`/`pong`/`chunk`/`unchunk`/`terrain`/`place`/`remove`/`reject`/`said`/`voice-peers`/`weather`/`time`/`system`. `welcome.now` is the server clock that drives day/night + weather, and `weather`/`time` override it when a dev command or the Oracle turns the sky; `welcome.world` describes the chunk grid, and the rest of the frame is what a joining client starts from (`weather`, `timeOfDay`, the player's `pieces` and `deeds`, and the world `feed`). `state` is filtered to players within 192 tiles. Changing a frame's shape means updating both consumers.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [benjamincanac/avelune](https://github.com/benjamincanac/avelune) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
