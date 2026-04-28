---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repo layout

```
src/platform/      C++ engine (logic/, net/, server/, agent/, client/, debug/, shaders/, fonts/, docs/)
src/artifacts/     Python game content — hot-loadable, no rebuild needed
                   behaviors/ living/ items/ blocks/ models/ worlds/ structures/ effects/ resources/ annotations/
src/python/        Python helpers (behavior_base, conditions_lib, local_world, pathfind, action_proxy, …)
src/config/        Runtime config (controls.yaml — copied into build dir)
src/resources/     Shipped assets (textures, sounds, models)
src/tests/         test_e2e.cpp, test_pathfinding.cpp, behavior_scenario_validation/
src/model_editor/  Authoring tooling (modelcrafter, bbmodel_export/import, capture_samples.sh)
docs/civcraft_legacy/  Game design docs (overview, actions, physics, inventory, …)
```

**Read `docs/civcraft_legacy/00_OVERVIEW.md` before making ANY gameplay changes.**
For engine/protocol/behavior-API details see `src/platform/docs/` (14_ARCHITECTURE_DIAGRAM.md, 21_BEHAVIOR_API.md, 04_SERVER.md, 06_NETWORKING.md).

## Mandatory Design Rules

These rules are NON-NEGOTIABLE. Every code change must respect them.

### Rule 0: The Server Accepts Exactly Four Action Types ← HIGHEST PRIORITY

**This is the most important architectural invariant. It overrides any other
design consideration.**

The server validates exactly four `ActionProposal` types — nothing more, nothing
less. Every action any entity can ever take must compile down to one of these:

```
TYPE_MOVE     (0) — set entity velocity/direction (no tunnelling)
TYPE_RELOCATE (1) — move an Object between containers (inventory ↔ ground ↔ entity);
                    value is conserved, nothing created from nothing
TYPE_CONVERT  (2) — transform an Object from one type to another (value must not
                    increase); consuming → convert to nothing; casting effect →
                    convert item to hp/effect; empty to_item = destroy
TYPE_INTERACT (3) — toggle interactive block state (door, button, TNT fuse)
```

**Consequences:**
- `Follow`, `Flee`, `Wander` are **NOT action types** — they are Python helpers
  that compute a target position and return `MoveTo` (TYPE_MOVE).
- All AI decision logic (follow, flee, wander, pathfind) lives in Python behaviors,
  never in the C++ bridge or server.
- The C++ bridge (`platform/server/python_bridge.cpp`) must never resolve or interpret
  high-level action names — it only translates the 4 primitives into `ActionProposal`.

See `docs/civcraft_legacy/00_OVERVIEW.md` § Action Types and `docs/civcraft_legacy/03_ACTIONS.md`.

### Rule 1: Python Is the Game, C++ Is the Engine

**All game content and gameplay rules are defined in Python. C++ only provides
the engine (physics, networking, rendering).**

- Creature definitions, behaviors, items, blocks, effects, models → `src/artifacts/`
- **NEVER hardcode gameplay constants in C++.** Every magic number (distance,
  speed, timer, radius) MUST be configurable from Python.
- **Creature AI behaviors are game logic** — they live in Python (`src/artifacts/behaviors/`).
- **Player click-to-move** uses server-side greedy steering (`platform/server/pathfind.h`)
  for simplicity and reliability. Creature pathfinding still runs in Python
  (`src/python/pathfind.py`) via AgentClient. Navigation tuning lives in `ServerTuning`.

### Rule 2: The Player Is Not Special

**Entity = Living + Item.** That's it.

- **Living** — moves, has HP, has inventory. Players, NPCs, animals are all Living.
  The player is just a Living entity with `playable=true`. Any Living can be
  played by hijacking its agent client. NPCs are Living with a BehaviorId.
- **Item** — on the ground or in an inventory.
- **Blocks** are NOT entities — they live in the chunk grid. Some blocks (chests)
  have inventories managed separately, keyed by block position.

### Rule 3: Server-Authoritative World Ownership

**The server is the sole owner and modifier of world state.**

- Entities submit `ActionProposal` (intent). Server validates and executes.
- Client NEVER writes to `chunk.set()`.
- **Client-side prediction**: the GUI client runs `moveAndCollide()` locally
  for the player entity (same physics as server) and reports `clientPos` in
  Move actions. The server accepts `clientPos` if within tolerance (8 blocks),
  otherwise snaps back.
- Singleplayer uses the same TCP code paths as multiplayer — no in-process shortcut.

### Rule 4: All Intelligence Runs on Agent Clients

**The server has ZERO intelligence.** All AI runs on agent clients.

- The AgentClient now lives **inside the player-client process** (one per client,
  not one per entity — see `platform/client/game_vk.cpp` where `m_agentClient` is
  constructed). It receives world state over TCP, runs Python `decide()` for every
  NPC owned by this client, and submits `ActionProposal`s back to the server.
- Python behavior code NEVER runs on the server.
- **Player click-to-move navigation** is handled server-side (greedy local
  steering in `platform/server/pathfind.h`). The GUI client sends `C_SET_GOAL` /

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [haoxuw/AiCraft](https://github.com/haoxuw/AiCraft) — distributed by [TomeVault](https://tomevault.io/claim/haoxuw).
<!-- tomevault:4.0:windsurf_rules:2026-04-18 -->
