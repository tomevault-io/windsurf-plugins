---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run build        # compile TypeScript → Lua via rbxtsc
npm run watch        # compile in watch mode (rbxtsc -w)
npm run dev          # compile as a game place, watch mode
npx eslint src/      # lint — currently broken, see note below
```

> **Known gap:** `npx eslint src/` fails outright (`eslint.config.js` not found). `package.json` pins ESLint 9, but the repo still ships the legacy `.eslintrc` format, which ESLint 9 can't read without a compat shim. Needs either a flat-config migration or a pin back to ESLint 8 before lint is usable again.

There is no automated test suite. Development and verification happen inside Roblox Studio: build, sync with Rojo (`default.project.json`), then play.

- **UI-Labs storybook** — `src/tests/storybook/` (`backpack.storybook.ts` plus `*.story.tsx`) is the component dev loop.
- **Runtime harnesses** — `src/tests/client/runtime.client.tsx` and `src/tests/server/runtime.server.ts` are demo entry points that exercise the library end to end. They are harnesses, not assertions.

## Architecture

**backpack-plus** (`@rbxts/backpack-plus`, v2.0.0-rc.1) is a published npm package that replaces the default Roblox inventory/backpack UI. It compiles TypeScript to Lua via roblox-ts and runs inside Roblox. Inspired by `ryanlua/satchel`.

### Module layout

```
src/lib/
├── client/
│   ├── core.ts         # initializeBackpackClient() — syncer, observer, input wiring (idempotent)
│   ├── charm.ts        # Charm signals: getClientBackpack/setClientBackpack, getClientEquipped (computed), getClientHotbar, getClientBackpackOrder, getDraggingState, getInventoryVisibility, getBackpackSelection, getBackpackFilters, getConsoleSwap
│   ├── icon.ts         # TopBarPlus inventory topbar icon, bound to the togglekey setting
│   ├── inputs/         # keyboard.ts (0–9 equip), console.ts (L1/R1 cycling), gamepad.ts (B/X + focus helpers), index.ts barrel
│   ├── settings/       # setting modules (configs/: device, viewport, slots, dimensions, inputtype, togglekey), types.ts (SettingModule), index.ts assembles getBackpackSettings
│   ├── tools.ts        # dragTool(), undragTool(), swapSlots(), swapSlotsHotbar(), equipTool(), getTool(), cancelDrag(), findTool* helpers
│   ├── filter.ts       # addFilter(), removeFilter(), getFilter(), clearFilter()
│   ├── hooks.ts        # onToolEquipped/Unequipped, onToolAdded/Removed, onInventoryToggled, onBackpackLoaded, onHotbarChanged, onSlotChanged
│   ├── networking.luau / networking.d.ts  # Zap-generated client remotes (SyncState, RequestState, RequestEquip)
│   ├── decorating/     # Plugin-style UI extension points (see "Decorating" below)
│   │   ├── slot.ts         # registerSlotDecorator(), SlotDecorator, ToolContext
│   │   ├── hotbar.ts       # registerHotbarDecorator(), HotbarDecorator
│   │   ├── inventory.tsx   # registerInventoryDecorator(), InventoryDecorator, default header/search
│   │   └── draggingslot.ts # dragging-slot decorators
│   └── ui/
│       ├── App.tsx     # BackpackPlusApp — StrictMode + ErrorBoundary, mounts Hotbar/Inventory/DraggingSlot
│       ├── components/  # hotbar/, inventory/, slot/, searchbox, styleprovider
│       ├── error/       # errorboundary.tsx, errorhandler.tsx
│       └── hooks/       # useStyle, useTags
├── server/
│   ├── core.ts         # initializeBackpackServer() — charm-sync server + equip handling
│   ├── charm.ts        # getClientBackpacks / setClientBackpacks (source of truth)
│   ├── clients.ts      # registerPlayer(), unregisterPlayer(), modifyPlayer(), getBackpack(), getClientOwnership()
│   ├── tools.ts        # giveTool(), removeTool(), updateTool(), holdTool(), getTool()
│   ├── hooks.ts        # onToolEquipped(), onToolUnequipped()
│   ├── data.ts         # toolMap, toolClientMap, toolRegistry (server-only tool bookkeeping)
│   └── networking.luau / networking.d.ts  # Zap-generated server remotes
└── shared/
    ├── types.ts        # ToolPlus, ToolId, ClientBackpack, ClientBackpacks, SyncBackpackGetter, BackpackNormalizedGetter
    └── utils/
        ├── id.ts        # generateId() — counter-based unique IDs
        └── fuzzyscore.luau / .d.ts  # fuzzy search scoring helper
```

### State: Charm signals, not atoms

State lives in `client/charm.ts` and `server/charm.ts` as **Charm v11 `signal()` pairs**, not `atom()`. Each signal destructures into a getter and a setter, both exported:

```ts
const [clientHotbar, updateClientHotbar] = signal(new Map<number, ToolId | "Drag" | "Empty">());
export const getClientHotbar = clientHotbar;
export const setClientHotbar = updateClientHotbar;
```

So the convention throughout is `getX()` to read and `setX(value | updater)` to write — there is no `xAtom` symbol anywhere. Setters accept either a new value or an updater function receiving the current one.

The one exception is `client/settings/`, which still uses `Atom` internally (`SettingModule.atom`) and exposes the assembled result through the `computed` getter `getBackpackSettings()`.

### State flow


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [teakzc/backpack-plus](https://github.com/teakzc/backpack-plus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
