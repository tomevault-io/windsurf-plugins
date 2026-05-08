---
trigger: always_on
description: EOWeb is a browser-based client for Endless Online (classic MMORPG, 0.0.28 protocol). It connects to game servers over WebSocket and renders the world using Pixi.js, with a Preact UI for menus/dialogs.
---

# EOWeb – Copilot Instructions

EOWeb is a browser-based client for Endless Online (classic MMORPG, 0.0.28 protocol). It connects to game servers over WebSocket and renders the world using Pixi.js, with a Preact UI for menus/dialogs.

## Commands

```sh
pnpm dev          # Vite dev server (port 3000)
pnpm build        # tsc && vite build → dist/
pnpm lint         # biome check .
pnpm format       # biome check --write .
```

There is no test suite.

## Architecture

The app has two rendering layers that serve different purposes:

- **Preact UI** (`src/ui/`) — overlaid on the canvas at all times. For pre-game screens (main menu, login, character select, etc.) it renders full-screen containers. In the `InGame` state it renders the in-game overlay: HUD, chat, hotbar, dialogs, touch controls, etc. Conditionally rendered based on `GameState` enum.
- **Pixi.js** (`src/map.ts`, `src/atlas.ts`, `src/render/`) — handles all in-game rendering: map tiles, characters, NPCs, animations, effects.

The central object is `Client` (`src/client/client.ts`). It owns all game state, every controller, the packet bus, the Pixi.js `Application`, and the `MapRenderer`. Nearly everything receives a `Client` reference.

### Data flow

```
WebSocket → PacketBus → handlers/ → client state / emit events → UI / Pixi render
```

- **`src/bus.ts` (`PacketBus`)** — manages the WebSocket connection and dispatches incoming packets by `PacketFamily + PacketAction` to registered handler functions.
- **`src/handlers/`** — one file per packet family. Each exports a `registerXxxHandlers(client: Client)` function that calls `client.bus.registerPacketHandler(family, action, fn)`. All are registered at startup via `registerAllHandlers(client)`.
- **`src/controllers/`** — encapsulate client-side logic for sending packets and managing related state (e.g., `MovementController`, `InventoryController`). Each takes `Client` in its constructor.
- Controllers expose typed `subscribe*` methods (e.g. `subscribeConfirm`, `subscribePaperdollOpened`) to notify subscribers of events.

### Persistence

`src/db.ts` wraps IndexedDB (via `idb`) with three stores:
- `pubs` — serialized EIF/ENF/ECF/ESF pub files (keyed by string: `'eif'`, `'enf'`, etc.)
- `maps` — serialized EMF map files (keyed by map ID number)
- `edfs` — EDF dialog/string files (keyed by file ID); fetched from `/data/datNNN.edf` on cache miss

### Game loop

`src/main.tsx` sets up a Pixi.js ticker at 120 ms per tick:
```ts
client.tick();            // game logic
client.render(interp);    // interpolated Pixi render
```

### Graphics

`src/atlas.ts` (`Atlas`) builds a dynamic texture atlas for character/NPC sprites on demand. Graphics files are loaded from `public/gfx/gfxNNN.egf` (Custom PE Files). `src/gfx/gfx-loader.worker.ts` loads and parses these files in a Web Worker.

## Key Conventions

### Imports

- **Always use `@/` path alias** — never `../` relative parent imports. This is enforced by Biome as an error.
- **Import from barrel files**, not deep paths. The following modules have barrels and must be imported from the top level:
  - `@/utils`, `@/render`, `@/controllers`, `@/handlers`, `@/gfx`, `@/fonts`
  - For in-game UI: `@/ui/in-game` (not `@/ui/in-game/chat/chat-manager`, etc.)
  - For UI enums (`ChatTab`, `ChatIcon`, `DialogIcon`, `SlotType`, `ISlot`): import from `@/ui/enums` directly to avoid circular dependencies, not from `@/ui`

### Style

- Single quotes for JS/TS strings and JSX attributes (enforced by Biome).
- Biome (not ESLint/Prettier) is the sole linter/formatter. Run `pnpm lint` to check, `pnpm format` to auto-fix.

### Packet handlers

Register handlers in `src/handlers/<family>.ts`:
```ts
export function registerWalkHandlers(client: Client) {
  client.bus!.registerPacketHandler(PacketFamily.Walk, PacketAction.Player,
    (reader) => handleWalkPlayer(client, reader));
}
```
Each handler function deserializes using the corresponding `eolib` packet class: `XxxServerPacket.deserialize(reader)`.

### Controllers

Controller classes store a private `client: Client` reference. They send packets by constructing `eolib` client packet objects and calling `this.client.bus!.send(packet)`.

### Hair style

`hairStyle` is 1-based. A value of `0` means no hair. Rendering uses `(hairStyle - 1) * 40 + hairColor * 4`.

### Nullable returns

- `getEmf(id)` returns `Promise<Emf | null>` — always handle the null case.
- `client.getDialogStrings(id)` always returns a 2-element `string[]` (fallback `['', '']`).

### UI Look and Feel

The UI uses DaisyUI components with Tailwind CSS. Reference @.github/daisyui-llms.txt for component usage and styling conventions.

Shared components (e.g., `Button`) are in `src/ui/components/`. Container components for each screen are in `src/ui/containers/`.

### In game UI elements

All in-game UI elements should be movable by the player (Toggle in settings, then drag to reposition) with default
positions based on screen size.

A global UI Scale setting should allow the player to adjust the size of all UI elements (except the main game canvas) for better readability on different screen sizes.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sorokya/eoweb](https://github.com/sorokya/eoweb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
