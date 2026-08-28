---
trigger: always_on
description: Pure-TypeScript renderer library for the Nitro retro Habbo client.
---

# Octane Renderer — Claude project context

Pure-TypeScript renderer library for the Nitro retro Habbo client.
Wraps **PixiJS v8** for room/avatar rendering and provides the WebSocket
+ event-bus infrastructure that the React client (`../octane`) sits on
top of.

## Stack

- **TypeScript 6.0** (root) + **tsgo** (`@typescript/native-preview`,
  TS 7 preview compiler — used by `yarn compile:fast`, ~7× faster on
  this codebase)
- **PixiJS v8** (`pixi.js@8.18`)
- **Vite 8** for build + bundling
- **Vitest 4** for unit tests
- **Yarn 4.18 workspaces** (`packages/*`) through Corepack, using the
  `node-modules` linker for compatibility with the existing build tooling.
- **No React** — this is a pure TS library; React lives in `../octane`.

## Workspace layout

Twelve internal packages under `packages/*/src/`, each pinning
`typescript: ^6.0.3` in its own `devDependencies`:

```
packages/
  api              public interfaces (IEventDispatcher, ISessionDataManager, ...)
  assets           asset loading + caching
  avatar           avatar rendering / figure resolution
  camera           in-room camera widget
  communication    WebSocket + composer/parser pipeline
  configuration    runtime config loader
  events           EventDispatcher + NitroEventType + per-domain events
  localization     LocalizationManager
  room             RoomEngine + RoomVisualization
  session          SessionDataManager + RoomSessionManager + handlers
  sound            SoundManager (howler-based)
  utils            shared utilities (BinaryReader, Logger, …)
```

Root `index.ts` re-exports everything from `@nitrots/*` so the React
client gets a flat `import { … } from '@nitrots/nitro-renderer'`.

## React-friendly API additions (v2.1.0)

Three additions matter for the React client integration. Keep these
backwards-compatible:

### `EventDispatcher.subscribe(type, callback): () => void`

Signature matches what `useSyncExternalStore` expects — returns an
unsubscriber, no need to juggle callback identity. Implemented in
`packages/events/src/EventDispatcher.ts`. The legacy
`addEventListener` / `removeEventListener` still work.

### `CommunicationManager.subscribeMessage(eventCtor, handler): () => void`

Equivalent for packet streams. Implemented in
`packages/communication/src/CommunicationManager.ts`.

### Snapshot getters (referentially stable, lazy-frozen, invalidated on mutation)

Pattern: `getXxxSnapshot()` returns a frozen value cached internally;
mutators call `invalidateXxxSnapshot()` which drops the cache AND
dispatches an invalidation event. The React side reads via
`useSyncExternalStore`.

| Manager | Getter | Invalidation event |
|---|---|---|
| `SessionDataManager` | `getUserDataSnapshot(): Readonly<IUserDataSnapshot>` | `SESSION_DATA_UPDATED` |
| `RoomSessionManager` | `getActiveRoomSessionSnapshot(): Readonly<IRoomSessionSnapshot> \| null` | `ROOM_SESSION_UPDATED` |
| `IgnoredUsersManager` | `getIgnoredUsersSnapshot(): ReadonlyArray<string>` | `IGNORED_USERS_UPDATED` |
| `GroupInformationManager` | `getGroupBadgesSnapshot(): ReadonlyMap<number, string>` | `GROUP_BADGES_UPDATED` (only on real changes — no-op refresh stays quiet) |
| `UserDataManager` | `getRoomUserListSnapshot(): ReadonlyArray<IRoomUserData>` | `ROOM_USER_LIST_UPDATED` (inner IRoomUserData kept mutable — don't deep-clone) |
| `SoundManager` | `getVolumesSnapshot(): Readonly<ISoundVolumesSnapshot>` | `SOUND_VOLUMES_UPDATED` (only when a volume actually changes) |

Snapshot interface contracts live under `packages/api/src/nitro/session/`
and `packages/api/src/nitro/sound/`. When adding a new snapshot, the
checklist is:
1. Define the `Ixxx Snapshot` interface in `packages/api/src/nitro/...`
   and export it from the matching `index.ts`.
2. Add a `XXX_UPDATED` member to `packages/events/src/NitroEventType.ts`.
3. Add `getXxxSnapshot()` to the interface AND impl; cache + invalidate
   on every mutation path (don't forget batch operations like queue
   truncation — invalidate AFTER the full batch, not mid-way).

Adding snapshots here is the preferred way to unblock new React
widgets — prefer it over exposing raw event-listener APIs on the
client side.

## Recent renderer changes (`feat/react19-event-bus`)

Tracked separately from the v2.1.0 batch above; all are
non-breaking additions or align-with-Arcturus fixes:

### RoomEnterComposer: optional `spawnX` / `spawnY`

`new RoomEnterComposer(roomId, password?, spawnX?, spawnY?)`. The
Arcturus `RequestRoomLoadEvent` handler reads the two extra ints only
when `packet.remaining >= 8`, so the same composer header serves both
the legacy 2-arg form (door spawn) and the 4-arg form (reconnect /
respawn at a specific tile). RoomSession + RoomSessionManager use the
4-arg variant in their `enterRoom` / reconnect paths.

### RoomSettingsData: `allowUnderpass` field

`RoomSettingsData` (and its parser) now exposes `allowUnderpass:
boolean`. Arcturus' `RoomSettingsComposer` already appends one
trailing int for this flag, and the new parser reads it via
`if(wrapper.bytesAvailable) … readInt() === 1` so older servers that
don't emit the field still parse cleanly. `SaveRoomSettingsComposer`
accepts an optional `allowUnderpass` arg at the end of its parameter
list; the server-side `RoomSettingsSaveEvent` reads it under

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duckietm/Octane-Renderer](https://github.com/duckietm/Octane-Renderer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
