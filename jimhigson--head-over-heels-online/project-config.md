---
trigger: always_on
description: My name is Jim Higson and my tag on github is jimhigson
---

# Me
My name is Jim Higson and my tag on github is jimhigson

### Project Summary: Isometric Game Engine (PixiJS v8, Preact 11)
 *	Framework & Version: Built using PixiJS v8 with modern WebGL2 features.
 *	Graphics:
 *	Uses classic isometric projection with a 2:1 pixel ratio.
 * although using 2d sprites for rendering, the game takes place in a 3d coordinate space, with x and y defining width and depth, and z being height
 *	Sprites are mostly 24×24 pixels, although some are larger such as wall tiles and larger blocks, using two colours plus transparent background.
 *	The world origin (0,0,0) is at the bottom-front corner from the player’s view.
 *	Xyz vectors are used for all 3D positions, with Xy used for 2D screen-space.
 * the game runs at a variable frame rate - anything from 30fps to 240fps is possible

## Level Format:
 *	Game levels are stored as JSON files.
 *	Each level’s items field is an object (not array) mapping IDs to item data.
 *	All items have a type, position: Xyz, and config.
 *	Walls may have a times property (x/y) for repetition/length.
 * rooms for the original game are converted from xml to json via scripts in the src/campaignXml2Json dir
 * x and y co-ordinates are on the horizontal plane. The z dimension represents height/altitude in the game

## Room json patches
* the conversion from xml is often patched to change things from the original game using jsondiff
* if editing original campaign rooms, regenerate the patches with `pnpm gen:roomsPatch` 

## Item config
* per-item-type config comes from ItemConfigMap
* after making changes to config, run gen:roomSchema so the jsonschema used by the editor is also updated, then check that the result looks ok
* optional new properties mean that old rooms created in the editor will still comply
## item State
* the item state is the values in an item that can change during gameplay
* new properties in state should usually be optional, or at least the code should work if they are omitted, so that old save games still work

## Campaigns:
* the player can play the original campaign, a sequel, or
a community-contributed campagin made using the level editor
* the original campaign is 'burnt' into the game's source via codegen
* all other campaigns are loaded from the db via supabase's codegen
* all are loaded dynamically - either using the db or using import
* therefore, only the original campaign is playable offline
* the original campagin is converted from xml using the script at src/campaignXml2Json/scripts/xml2json.ts

## Preact / React

The runtime is Preact 11 (beta), but the codebase retains React import paths and type definitions:

* `@preact/preset-vite` aliases `react` → `preact/compat` and `react-dom` → `preact/compat` at build time, so `import { useState } from "react"` resolves to Preact at runtime
* Type-only imports (`import type { ReactNode } from "react"`) use `@types/react` v19 for type-checking — this is intentional, not a migration leftover
* Hooks should be imported from `preact/hooks` (e.g. `import { useEffect } from "preact/hooks"`), not from `"react"`
* Entry points use `import { render } from "preact"` directly
* Third-party React libraries (`react-redux`, `@floating-ui/react`, `@monaco-editor/react`, etc.) work through the compat layer
* No Preact-specific APIs like signals are used

## Rendering:
 *	Layers use Pixi’s RenderLayer, specifically to emulate colour clash of the zx spectrum

## Game engine
### Iterating room items
When iterating over `RoomStateItems` (the `room.items` object), use the typed helpers from `src/model/RoomState.ts` instead of `objectValues()` directly:
- `roomItemsIterable(roomItems)` - returns a raw `IterableIterator` with correct item types, for `for...of` loops or passing to constructors like `new Map()`
- `iterateRoomItems(roomItems)` - wraps in `iterate()` to provide iterator helper methods (`.map()`, `.filter()`, etc.)
- `iterateRoomItemEntries(roomItems)` - when you need both the item id and item value as `[id, item]` tuples

```ts
// Good - proper type inference
for (const item of roomItemsIterable(room.items)) {
  item.state.position; // correctly typed
}

// Good - with iterator helpers
iterateRoomItems(room.items).filter(isFreeItem).map(i => i.state.position);

// Bad - loses type information
for (const item of objectValues(room.items)) { ... }
```

Similarly, when iterating over `RoomJsonItems` (editor/JSON room items), use the helpers from `src/model/RoomJson.ts`:
- `roomJsonItemsIterable(roomJson)` - takes the whole `RoomJson` object, returns `IterableIterator` of items (no ids)
- `roomJsonItemsEntriesIterable(roomJson.items)` - returns `[id, item]` tuples
- `iterateRoomJsonItemsWithIds(roomJson.items, ...types?)` - returns iterator helper with `[id, item]` tuples, optionally filtered by item type

Prefer `roomJsonItemsIterable` when you only need items without ids. Use `iterateRoomJsonItemsWithIds` when you need the id or want built-in type filtering.

## Editor Features:
 * there is a level editor also built for creating new rooms for the game
 * this is for creating new non-original levels which are specific to the remake
 *	A level editor supports door placement which removes or splits walls where the door appears.

### Schema

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jimhigson/head-over-heels-online](https://github.com/jimhigson/head-over-heels-online) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
