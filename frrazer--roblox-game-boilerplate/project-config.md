---
trigger: always_on
description: - Do not add comments to code. Luau directives such as `--!strict` are allowed.
---

# Game Boilerplate — agent instructions

## Code style

- Do not add comments to code. Luau directives such as `--!strict` are allowed.
- Keep `--!strict` at the top of every Luau file. Run `./scripts/check.ps1` before handing off code changes.
- Format with StyLua (tabs, 110 columns, double quotes). Never edit generated `Packages/`, `ServerPackages/`, `build/` or `sourcemap.json`.

### Naming

| Thing | Convention | Example |
| --- | --- | --- |
| Files, modules, folders | PascalCase, matching the returned table | `PurchaseService.luau`, `RateLimit.luau` |
| Services and controllers | PascalCase ending in `Service` or `Controller`; `Name` matches the file | `DataService`, `UIController` |
| Types | PascalCase; export shared ones with `export type` | `PlayerData.Data`, `Types.Screen` |
| Public functions, methods, fields and signals | PascalCase | `DataService:GetData`, `DataService.PlayerLoaded` |
| Private fields | `_camelCase` | `self._trove` |
| Local variables and local functions | camelCase | `local activeSessions`, `local function refresh()` |
| Constants | camelCase locals, or PascalCase keys in a frozen `Config` table | `local maxReceipts = 100`, `GameConfig.Name` |
| Unused parameters | `_` prefix | `_self`, `_player` |
| Packets | PascalCase verb or event name | `Buy`, `RoundStarted` |
| Tags and attributes | PascalCase | `Spinner`, `SpinSpeed`, `DataLoaded` |
| Services and packages | Local named after the service or package | `local Players = game:GetService("Players")`, `local Trove = require(...Trove)` |

- Name things for what they are, not how they are used: `Catalog`, not `ShopHelper`.
- Avoid abbreviations except universally understood ones (`id`, `ui`, `dt`).
- Booleans read as questions: `isOpen`, `hasLoaded`, `StartsOpen`.

## Scope

- This is a generic game boilerplate. Keep it game-agnostic: no genre-specific systems, assets or design decisions unless the user asks. Ask before making material design decisions.
- Explicit user instructions take precedence.

## Setup and checks

Tools are pinned in `aftman.toml`: Rojo, Wally, StyLua, wally-package-types and luau-lsp. The only prerequisite is [Aftman](https://github.com/LPGhatguy/aftman).

```powershell
./scripts/install.ps1
rojo serve default.project.json --address 127.0.0.1
```

- `scripts/install.ps1` installs the tools, downloads the Roblox type definitions (`globalTypes.d.luau`, Git-ignored) if missing, runs `wally install`, then `wally-package-types` so package types (such as `Signal.Signal<T...>`) are visible to strict code. Run it on a fresh clone and after changing `wally.toml`; never run bare `wally install`. On first run Aftman may ask the user to trust each tool.
- Connect the Rojo plugin in Studio to `localhost:34873`. Once the game has a place, add `"servePlaceIds": [<placeId>]` to `default.project.json`.
- In Studio, enable **Game Settings > Security > Enable Studio Access to API Services** (the place must be published). Without it (or in an unpublished place), ProfileStore falls back to in-memory mock data: everything works, but data resets when Play stops, and `DataService` warns once. Live servers always have DataStore access.
- In Play, output shows `[Game Boilerplate] Server services ready`, `Client controllers ready` and a Packet round trip. Both Bootstraps set a `Ready` attribute.
- `./scripts/check.ps1` is the one command to run after every change. It formats `src` with StyLua, regenerates the sourcemap, runs luau-lsp type and lint analysis, and does a Rojo build. Fix every error it reports and rerun it until it passes.
- `rojo build` output is code-only, not a replacement for the authored place.

## Layout

| Files | Studio destination | Contents |
| --- | --- | --- |
| `src/server` | `ServerScriptService` | `Bootstrap.server.luau`, `Registry.luau`, `Services/`, `Data/`, `Components/` (create when needed) |
| `src/client` | `StarterPlayer.StarterPlayerScripts` | `Bootstrap.client.luau`, `Registry.luau`, `Controllers/`, `Components/` |
| `src/shared` | `ReplicatedStorage` | `Lifecycle`, `Packet/`, `Packets`, `Config/`, `Utils/` |
| `Packages` | `ReplicatedStorage.Packages` | Generated Wally dependencies |
| `ServerPackages` | `ServerScriptService.ServerPackages` | Generated server-only Wally dependencies |

- Edit code on disk; Rojo syncs it. Code lives directly in the service roots, with no Client/Server/Shared wrapper folders.
- Rojo owns mapped folders. Unknown children at service roots are preserved. Workspace, lighting, GUI and other art are edited and saved in Studio, not on disk.
- Server-only code (secrets, rules, data) belongs in `src/server`. Everything in `src/shared` is visible to clients.
- Put tunable constants in `src/shared/Config` as frozen tables, or in server modules if clients must not see them.
- Put pure logic in plain modules (such as `src/server/Data`) with no service dependencies.
- Keep `src/shared` tidy: its root holds only the framework (`Lifecycle`, `Packet`, `Packets`) and folders. Reusable, game-agnostic helpers used by more than one system go in `src/shared/Utils` (such as `Utils/RateLimit`), one module per helper. Server-only helpers go in `src/server/Utils`. Helpers used by a single system stay inside that system's folder.

## Systems


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [frrazer/roblox-game-boilerplate](https://github.com/frrazer/roblox-game-boilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
