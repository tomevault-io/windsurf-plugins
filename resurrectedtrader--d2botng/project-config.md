---
trigger: always_on
description: Modern Diablo II bot manager. Manages D2 game instances, handles CD key rotation, communicates with D2BS scripts via WM_COPYDATA, provides a web UI.
---

# D2BotNG

Modern Diablo II bot manager. Manages D2 game instances, handles CD key rotation, communicates with D2BS scripts via WM_COPYDATA, provides a web UI.

## Stack

| Layer | Tech |
|-------|------|
| Backend | .NET 10, C# 13, ASP.NET Core, gRPC, Serilog |
| Frontend | React 18, TypeScript, Vite 6, Tailwind CSS, HeadlessUI |
| State | Zustand (events), TanStack React Query (mutations) |
| gRPC | `@connectrpc/connect` + `@connectrpc/connect-web` |
| Windows | P/Invoke, WebView2, WM_COPYDATA IPC |

## Layout

```
protos/                  # Protobuf definitions (source of truth for all services)
src/
  D2BotNG/               # .NET backend (x86 Windows)
    Services/            # gRPC implementations (*ServiceImpl.cs)
    Engine/              # Profile lifecycle (ProfileEngine), scheduling (ScheduleEngine)
    Windows/             # Win32 interop: GameLauncher, ProcessManager, Patcher, MessageWindow
    Data/                # Protobuf JSON persistence (FileRepository pattern, data/ng/)
    Legacy/
      Api/               # Legacy D2Bot# HTTP API compatibility layer
      Models/            # Legacy JSONL models + migration
    Rendering/           # DC6 sprite decoding, palette management, item rendering
    Logging/             # Serilog extensions, TrackingLoggerFactory, LoggerRegistry, MessageServiceSink
    UI/                  # WinForms MainForm, WebView2 host, system tray
  D2BotNG.UI/            # React frontend
    src/
      features/          # Page components (profiles, keys, schedules, characters, items, settings)
      components/
        layout/          # Layout, Sidebar, Header, ConsolePanel
        ui/              # Reusable UI library (Button, Card, Dialog, Table, Toast, etc.)
      stores/            # Zustand (event-store, toast-store)
      hooks/             # React Query mutations + useEventStream
      lib/               # gRPC client, auth, DC6 rendering pipeline
        rendering/       # dc6Decoder, paletteManager, itemRenderer, colors
      generated/         # Auto-generated protobuf types (buf generate)
Resources/               # DC6 sprites, palettes (pal.dat, Pal.PL2), fonts
docs/plans/              # Design docs and implementation plans
reference/               # Reference D2Bot implementation for parity
```

## Commands

```bash
# Backend
cd src/D2BotNG
dotnet build                       # Build (also builds UI via MSBuild target)
dotnet build -p:SkipUIBuild=true   # Build backend only (skip npm build)
dotnet build -p:RunFormat=true     # Run dotnet format before build
dotnet build -p:RunInspect=true    # Run ReSharper inspect after build (review src/D2BotNG/obj/inspect.sarif)
dotnet run -- --dev-ui             # Dev mode (proxy to Vite at :4200)
dotnet run -- --headless           # Server only (no GUI window)

# Frontend
cd src/D2BotNG.UI
npm install
npm run dev                        # Vite dev server on port 4200
npm run build                      # Production build to ../D2BotNG/wwwroot
npm run lint                       # ESLint
npm run format                     # Prettier
npm run generate-grpc              # Regenerate protobuf types from protos/

# Publish (single exe)
cd src/D2BotNG
dotnet publish -c Release --self-contained       # Bundles .NET runtime (~60-80MB)
dotnet publish -c Release --no-self-contained    # Requires .NET 10 runtime (~15-25MB)
# Output: bin/Release/net10.0-windows/win-x86/publish/D2BotNG.exe
```

## gRPC Services

All defined in `protos/*.proto`, implemented in `src/D2BotNG/Services/*ServiceImpl.cs`:

| Service | Proto | Methods |
|---------|-------|---------|
| **ProfileService** | profiles.proto | CRUD, Start/Stop/Restart, ShowWindow/HideWindow, ResetStats, RotateKey, ReleaseKey, SetScheduleEnabled, Reorder, TriggerMule |
| **KeyService** | keys.proto | CreateKeyList, UpdateKeyList, DeleteKeyList, HoldKey, ReleaseHeldKey |
| **ScheduleService** | schedules.proto | Create, Update, Delete |
| **EventService** | events.proto | StreamEvents (server stream), ClearMessages |
| **SettingsService** | settings.proto | Update, TestDiscord |
| **FileService** | settings.proto | ListDirectory (file browser for path selection) |
| **ItemService** | items.proto | ListEntities, Search |
| **UpdateService** | updates.proto | CheckForUpdate, StartUpdate |
| **LoggingService** | logging.proto | SetLogLevel, GetLogLevels |

## Event Architecture

Frontend uses a single gRPC server-stream for all real-time state:

1. `useEventStream` hook connects to `EventService.StreamEvents()`
2. Server sends initial snapshots (profiles, key lists, schedules, settings, update status, log levels, console history)
3. Server streams incremental changes (ProfileStatusChanged, Message, SettingsChanged, etc.)
4. Zustand `event-store` processes events and updates state maps
5. Mutations (create/update/delete) return `Empty` - UI updates arrive via the stream
6. Auto-reconnect on disconnect with 5s retry

**Event types:** ProfilesSnapshot, KeyListsSnapshot, SchedulesSnapshot, ProfileStatusChanged, Message, SettingsChanged, UpdateStatusChanged, EntitiesChanged, LogLevelsSnapshot

## Backend Architecture

### Engine Layer

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ResurrectedTrader/D2BotNG](https://github.com/ResurrectedTrader/D2BotNG) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
