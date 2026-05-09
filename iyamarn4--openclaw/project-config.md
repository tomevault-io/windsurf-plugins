---
trigger: always_on
description: This file provides AI assistants with essential context about the OpenClaw codebase: its structure, build system, conventions, and development workflows.
---

# CLAUDE.md — OpenClaw

This file provides AI assistants with essential context about the OpenClaw codebase: its structure, build system, conventions, and development workflows.

---

## Project Overview

**OpenClaw** is a multiplatform C++ reimplementation of *Captain Claw* (1997), the classic Monolith platformer. The entire engine is written from scratch; it reuses only the original game assets from the `CLAW.REZ` archive (not redistributed). The project targets Windows, Linux, macOS, Android, and WebAssembly (Emscripten).

- **Language:** C++11
- **Renderer/Input/Audio:** SDL2, SDL2_image, SDL2_mixer, SDL2_ttf, SDL2_gfx
- **Physics:** Box2D
- **Data/Config:** TinyXML-2 (XML parsing)
- **Signals:** libsigc++3
- **Build system:** CMake (primary) + Visual Studio 2017 solution (`OpenClaw.sln`)
- **CI:** Travis CI (`.travis.yml`) and AppVeyor (`appveyor.yml`)

---

## Repository Layout

```
OpenClaw/                        # Root
├── Box2D/                       # Bundled Box2D physics library
├── Build_Release/               # CMake output directory (executables + assets)
├── ClawLauncher/                # GUI launcher (Windows/Linux via Mono)
├── libsigc++3/                  # Bundled signal/slot library
├── libwap/                      # Custom library for reading .REZ / .WAP assets
├── OpenClaw/                    # Main game source code
│   ├── Engine/                  # Core engine subsystems
│   │   ├── Actor/               # Actor + component system
│   │   │   └── Components/      # Individual actor components
│   │   ├── Audio/               # Audio manager
│   │   ├── Events/              # Event system
│   │   ├── GameApp/             # Application framework & main loop
│   │   ├── Graphics2D/          # 2D rendering
│   │   ├── Logger/              # Logging utilities
│   │   ├── Physics/             # Box2D integration
│   │   ├── Process/             # Process manager (game loop tasks)
│   │   ├── Resource/            # Resource cache & loaders
│   │   ├── Scene/               # Scene graph
│   │   ├── UserInterface/       # Menus, HUD, screens
│   │   └── Util/                # General utilities
│   ├── ActorController.cpp/h    # Player/actor input controller
│   ├── ClawEvents.cpp/h         # Game-specific event definitions
│   ├── ClawGameApp.cpp/h        # Top-level application class
│   ├── ClawGameLogic.cpp/h      # Game logic layer
│   ├── ClawHumanView.cpp/h      # Human (rendering) view
│   └── main.cpp                 # Entry point
├── Scripts/                     # Build/utility scripts
├── ThirdParty/                  # Additional third-party deps (e.g. TinyXML)
├── CMakeLists.txt               # Root CMake configuration
├── OpenClaw.sln                 # Visual Studio 2017 solution
├── .travis.yml                  # Travis CI config
└── appveyor.yml                 # AppVeyor CI config
```

---

## Architecture

### Application Layer

`ClawGameApp` inherits from `BaseGameApp` (in `Engine/GameApp/`) and overrides:
- `VGetGameTitle()` → `"Captain Claw"`
- `VGetGameAppDirectory()` → SDL2 `SDL_GetBasePath()`
- `VCreateGameAndView()` → instantiates `ClawGameLogic` + `ClawHumanView`

Entry point (`main.cpp`) calls `RunGameEngine(argc, argv)` which drives the main loop.

### Component-Entity System (Actor Model)

Actors are composed of components. Key files:

| File | Role |
|------|------|
| `Engine/Actor/Actor.h` | Base `Actor` class, holds components by type ID |
| `Engine/Actor/ActorComponent.h` | `ActorComponent` interface |
| `Engine/Actor/ActorFactory.h/.cpp` | Creates actors from XML definitions |
| `Engine/Actor/ActorTemplates.h/.cpp` | Predefined actor configs |
| `Engine/Actor/Components/` | Concrete component implementations |

Actor creation flow: XML element / resource path → `ActorFactory::CreateActor()` → component construction via `GenericObjectFactory` → unique `ActorId` (GUID counter).

### Interfaces

`Engine/Interfaces.h` defines the three main engine interfaces all major subsystems implement:
- `IGameLogic` — actor lifecycle, level/menu loading, game-state transitions
- `IGameView` — render tick, input routing, actor attachment
- `IGamePhysics` — add/remove bodies, apply forces, raycast/AABB queries

### Event System

Events are declared in `Engine/Events/` and game-specific ones in `ClawEvents.h`. The system uses `IEventData` base objects and a listener/dispatcher pattern.

### Physics

Box2D is bundled under `Box2D/` and integrated in `Engine/Physics/`. Collision categories are defined as `CollisionType`/`CollisionFlag`/`FixtureType` enums in `Interfaces.h`.

---

## Coding Conventions

### Naming
| Element | Convention | Example |
|---------|-----------|---------|
| Classes | `PascalCase` | `ClawGameLogic` |
| Virtual methods | `V` prefix + `PascalCase` | `VGetGameTitle()` |
| Member variables | `_camelCase` (underscore prefix) | `_lastActorGUID` |
| Constants / enums | `PascalCase` or `ALL_CAPS` | `ActorPrototype`, `SAFE_DELETE` |
| Files | `PascalCase` matching class name | `ActorFactory.h` |

### Pointers & Memory
- Raw owning pointers are wrapped with `SAFE_DELETE` / `SAFE_DELETE_ARRAY` macros (defined in `SharedDefines.h`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iyamarn4/OpenClaw](https://github.com/iyamarn4/OpenClaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
