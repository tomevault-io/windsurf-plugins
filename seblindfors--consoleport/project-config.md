---
trigger: always_on
description: > Reference document for AI agents and contributors working on the ConsolePort addon suite.
---

# ConsolePort Coding Standards

> Reference document for AI agents and contributors working on the ConsolePort addon suite.
> ConsolePort is a World of Warcraft gamepad addon written in Lua/XML against the WoW API.

---

## Table of Contents

1. [Project Architecture](#1-project-architecture)
2. [File Organization](#2-file-organization)
3. [Naming Conventions](#3-naming-conventions)
4. [Formatting & Style](#4-formatting--style)
5. [The `db` Object & Data Access](#5-the-db-object--data-access)
6. [Event System](#6-event-system)
7. [Mixin & OOP Patterns](#7-mixin--oop-patterns)
8. [Variable / Settings Definitions](#8-variable--settings-definitions)
9. [Secure Environment Code](#9-secure-environment-code)
10. [Error Handling](#10-error-handling)
11. [Idioms & Patterns](#11-idioms--patterns)
12. [Localization](#12-localization)
13. [XML Templates](#13-xml-templates)
14. [Multi-Version Support](#14-multi-version-support)

---

## 1. Project Architecture

### Addon Suite Structure

ConsolePort is a **suite of addons** sharing a single core. Each sub-addon is a separate WoW addon with its own `.toc` file.

| Addon | Loads | Purpose |
|---|---|---|
| `ConsolePort` | Always | Core: API, database, input, controller, widget library |
| `ConsolePort_Bar` | Always | Action bar replacement |
| `ConsolePort_Config` | On Demand (LOD) | Settings UI |
| `ConsolePort_Cursor` | On Demand (LOD) | Interface cursor for gamepad navigation |
| `ConsolePort_Keyboard` | On Demand (LOD) | Virtual keyboard |
| `ConsolePort_Menu` | Always | Game menu replacement |
| `ConsolePort_Rings` | Always | Ring menus / utility rings |
| `ConsolePort_World` | Always | World interaction features |

### Three Global Entry Points

All code communicates through three globals:

- **`ConsolePort`** — The public API frame. External addons call methods on this object (e.g. `ConsolePort:GetBindings()`). Defined in `ConsolePort/API.lua`.
- **`db`** — The internal database (a `RelaTable` instance). Accessed in every file via `local _, db = ...;`. Provides path-based get/set, event callbacks, table utilities, and subsystem registration.
- **`CPAPI`** — Global utility table of functions, mixins, and constants. Available before any addon file loads. Contains helpers like `CreateEventHandler`, `LinkEnv`, `Popup`, `Prop`, `Bool`, etc.

### MVC-ish Sub-Addon Layout

Each sub-addon follows a loose Model–Controller–View structure, plus optional `Widget/` and `Assets/` directories:

```
ConsolePort_Bar/
├── Model/          # Data types, database registration, interface definitions
├── Controller/     # Logic, state management, Blizzard hooks
├── View/           # UI frames, layout, display
├── Widget/         # Reusable widget components
└── Assets/         # Textures, atlases
```

### Core Addon Load Order

Defined in `ConsolePort.toc` — order matters:

```
Libs → XML → API.lua → Utils → Model → Controller → Widget → View
```

Sub-addon internal order is declared directly in the `.toc` file (files listed top-to-bottom). Within the core addon, each directory uses a `__manifest.xml` to declare its own load order.

---

## 2. File Organization

### Manifests

The core addon uses `__manifest.xml` files (double-underscore prefix) in each directory to declare load order:

```xml
<Ui>
    <Script file="Database.lua"/>
    <Script file="Utils.lua"/>
    <Script file="Const.lua"/>
    <Include file="SubDir/__manifest.xml"/>
</Ui>
```

Sub-addons list files directly in their `.toc`, sometimes with nested `.xml` files for component subdirectories (e.g. `Widget/Button/Button.xml`).

### File Naming

- **PascalCase** for Lua and XML files: `Database.lua`, `Button.lua`, `Templates.xml`
- **PascalCase** for directories: `Controller/`, `Model/`, `View/`, `Widget/`
- **Locale files** use locale codes: `enUS.lua`, `zhCN.lua`
- **Manifest files** use `__manifest.xml` (double underscore)
- **Sub-addon directories** match their TOC name: `ConsolePort_Bar/`, `ConsolePort_Cursor/`

### File Preamble

Every Lua file begins by destructuring the addon varargs:

```lua
local _, db = ...;
```

Or with CPAPI:
```lua
local CPAPI, _, db = CPAPI, ...;
```

Frequently followed by upvalue caching of globals:
```lua
local getmetatable, setmetatable = getmetatable, setmetatable;
local CreateFrame, Mixin = CreateFrame, Mixin;
```

---

## 3. Naming Conventions

### Variables

| Context | Convention | Examples |
|---|---|---|
| Local variables | `camelCase` | `activeDevice`, `classColor`, `totalFree`, `freeSlots` |
| Local references to classes/handlers | `PascalCase` | `local Cursor`, `local HotkeyHandler`, `local PowerLevel` |
| Constants | `UPPER_SNAKE_CASE` | `MOUSEOVER_THROTTLE`, `LCLICK_BINDING` |
| Hex constants | Hex literals | `0x0`, `0x1`, `0x2` |

### Functions

| Context | Convention | Examples |
|---|---|---|
| Methods on objects/mixins | `PascalCase` | `SetIcon`, `OnDataLoaded`, `GetBindings` |
| CPAPI functions | `CPAPI.PascalCase` | `CPAPI.CreateEventHandler`, `CPAPI.LinkEnv` |
| WoW event handlers | Match event name | `function Handler:UPDATE_BINDINGS()` |
| Private/internal helpers | `camelCase` or `local function` | `local function clearLockedState()` |

### Frames

| Context | Convention | Examples |
|---|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [seblindfors/ConsolePort](https://github.com/seblindfors/ConsolePort) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
