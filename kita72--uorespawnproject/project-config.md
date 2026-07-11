---
trigger: always_on
description: This repo contains TWO distinct codebases. **Never cross them when answering app questions.**
---

# Copilot Instructions

## ⚠️ Server Scripts Boundary — Read Before Searching

This repo contains TWO distinct codebases. **Never cross them when answering app questions.**

| Area | Path | Purpose |
|------|------|---------|
| **App** ✅ | `UORespawnApp/Scripts/` | .NET 10 MAUI Blazor app — always reference this for app work |
| **App UI** ✅ | `UORespawnApp/Components/` | Blazor components, layouts, pages |
| **Server scripts** 🚫 | `UORespawnApp/Data/SERVER/` | ServUO / MUO C# scripts — deployed to game server, never edited for app work |

### Rules for all AI tools
- When `code_search` or any workspace search returns results, **discard any match whose path contains `Data/SERVER/`** before acting on results.
- When looking for app utilities, version strings, services, or entities — **only look under `UORespawnApp/Scripts/`**.
- Never suggest edits to files under `Data/SERVER/` unless the user explicitly asks about server-side script work.
- The version string lives in **`UORespawnApp/Scripts/Utilities/Utility.cs`** — not in any server utility file.

---

## Project Overview
UORespawn is a professional spawn management system for Ultima Online servers running ServUO. Built with .NET 10 MAUI Blazor Hybrid, it provides a **desktop-only** editor (Windows/macOS) for creating and managing creature spawns.

**Platform Support: Desktop Only (Windows, macOS)**
- We do NOT support mobile platforms (iOS, Android)
- We do NOT support web deployment
- Blazor Hybrid is used for its UI benefits, not cross-platform mobile/web capabilities

---

## Project Structure Mind Map
UORespawnProject/
├── UORespawnApp/                    # Main MAUI Blazor Hybrid application
│   ├── Components/
│   │   ├── Controls/                # Blazor UI components (.razor + .razor.css)
│   │   ├── Layout/                  # MainLayout.razor, NavMenu
│   │   └── Pages/                   # Page-level components
│   ├── Scripts/
│   │   ├── Constants/               # PathConstants, AppConstants
│   │   ├── Entities/                # Data entities (ServUO-style, not DTOs)
│   │   ├── Services/                # Business logic services
│   │   └── Utilities/               # Static helper utilities
│   ├── Data/                        # Runtime data folder (user accessible)
│   │   ├── UOR_DATA/                # Active spawn binary files (.bin)
│   │   ├── PACKS/                   # Spawn packs (Approved, Created, Imported)
│   │   └── MAPS/                    # Map images (Map0.bmp - Map255.bmp)
│   ├── Resources/Raw/               # Bundled resources (default reference data)
│   └── wwwroot/
│       ├── css/                     # Global styles
│       └── js/map.js                # Canvas rendering & mouse interactions
│
└── Data/SERVER/                     # Server-side scripts (copied to ServUO)
    └── UORespawnSystem/
        ├── Commands/                # In-game admin commands
        ├── Core/                    # Core spawn engine
        ├── Entities/                # Server entity definitions
        └── Mobiles/                 # Custom mobile behaviors
---

## Architecture

### Server Directory Structure (v2.0+)
When linked to ServUO, the server uses this folder structure under `Data/UORespawn/`:UORespawn/
├── INPUT/       # Editor writes .bin files here (server reads)
├── OUTPUT/      # Server writes .txt files here (editor reads)
├── STATS/       # Heatmap/player spawn statistics
└── SYS/         # Internal tracking files
### Data Flow
1. **Editor → Server**: Editor saves `.bin` files to `Data/UOR_DATA/`, DataWatcher syncs to server's `INPUT/`
2. **Server → Editor**: Server auto-generates `.txt` files in `OUTPUT/` on startup (bestiary, regions, spawners, vendors)
3. **Spawn Packs**: `Data/PACKS/` organizes packs by category (Approved/Created/Imported); applying a pack copies files to `UOR_DATA/`

### Binary Files (Editor Writes)
| File | Purpose |
|------|---------|
| `UOR_BoxSpawn.bin` | Box spawn definitions |
| `UOR_TileSpawn.bin` | Tile/point spawn definitions |
| `UOR_RegionSpawn.bin` | Region-based spawn definitions |
| `UOR_VendorSpawn.bin` | Vendor spawn definitions |
| `UOR_Settings.bin` | Editor settings |

### Text Files (Server Generates - Auto on Startup)
| File | Purpose |
|------|---------|
| `UOR_Bestiary.txt` | All spawnable creature types |
| `UOR_RegionList.txt` | All region names and bounds |
| `UOR_SpawnerList.txt` | XML spawner locations (for overlay) |
| `UOR_VendorList.txt` | Vendor types available |

---

## Key Services (Scripts/Services/)

| Service | Purpose |
|---------|---------|
| `ViewService` | View navigation, shared state (current map, XML/Spawns toggles) |
| `DataWatcher` | Monitors server folder for file changes (Windows/macOS) |
| `SpawnPackService` | Load, validate, import, export, apply spawn packs |
| `BinarySerializationService` | All .bin file read/write operations |
| `ToastService` | Warning toasts only (no success/info toasts) |
| `SettingsService` | App settings persistence |

---

## Key Utilities (Scripts/Utilities/)

| Utility | Purpose |
|---------|---------|
| `MapUtility` | Map ID ↔ name conversion, available maps scanning |
| `BestiaryUtility` | Load/parse bestiary creature list |
| `RegionUtility` | Load/parse region definitions |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kita72/UORespawnProject](https://github.com/Kita72/UORespawnProject) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
