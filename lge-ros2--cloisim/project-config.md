---
trigger: always_on
description: This context file was built from direct source analysis of the workspace. When project documentation and checked-in code disagree, this file follows the code.
---

# Claude Code Assistant Context: CLOiSim

This context file was built from direct source analysis of the workspace. When project documentation and checked-in code disagree, this file follows the code.

## 1. What this project is

CLOiSim is a Unity-based multi-robot simulator. It reads SDF (Simulation Description Format) world and model files, dynamically builds a Unity scene from them, and exposes simulated sensor data and control endpoints over a NetMQ-based transport layer. An external ROS 2 bridge package (`cloisim_ros`) connects these endpoints to ROS topics.

## 2. Actual Unity version and package state

Despite README references to Unity 2022.3 LTS, the checked-in `ProjectSettings/ProjectVersion.txt` declares:

```
m_EditorVersion: 6000.3.11f1
```

This is a **Unity 6** project. Key packages from `Packages/manifest.json`:

| Package | Version |
|---------|---------|
| `com.unity.render-pipelines.universal` | 17.3.0 |
| `com.unity.inputsystem` | 1.19.0 |
| `com.unity.ai.navigation` | 2.0.11 |
| `com.unity.mathematics` | 1.3.3 |
| `com.unity.splines` | 2.8.4 |
| `com.unity.terrain-tools` | 5.3.2 |
| `com.unity.robotics.vhacd` | GitHub (Unity-Technologies/VHACD) |

Third-party libraries in `Assets/Plugins/`:

| Library | Purpose |
|---------|---------|
| AssimpNetter 6.0.2.1 | Mesh import (OBJ, DAE, STL) via native Assimp |
| NetMQ 4.0.2.2 + AsyncIO | ZeroMQ transport for sensor/control data |
| protobuf-net 3.2.56 | Protocol Buffers serialization for messages |
| WebSocketSharp 1.0.3-rc11 | WebSocket server for external simulation control |
| Newtonsoft.Json 13.0.4 | JSON handling for WebSocket API |

## 3. Architectural layers (from source analysis)

The simulator is organized into five cooperating layers, all rooted under `Assets/Scripts/`:

### Layer 1: Startup and orchestration — `Main.cs`

`Main` is the singleton `MonoBehaviour` (`[DefaultExecutionOrder(30)]`) that bootstraps everything:

**`Awake()` does:**
- Replaces legacy `StandaloneInputModule` with `InputSystemUIInputModule`
- Redirects `Console.Out`/`Console.Error` to Unity log via `DebugLogWriter`
- Reads resource paths from environment variables (`CLOISIM_FILES_PATH`, `CLOISIM_MODEL_PATH`, `CLOISIM_WORLD_PATH`) — colon-separated, multiple paths supported
- Loads Assimp native library dynamically (platform-conditional paths)
- Forces `AsyncIO.ForceDotNet.Force()` for NetMQ Windows compatibility
- Sets `Application.targetFrameRate = 60`, enables texture streaming (`512MB` budget), reduces shadow distances
- Configures per-layer camera culling distances (Default layer gets 50% of far clip)
- Reads `CLOISIM_QUALITY` env var (0-4, default 3 = "Very High")
- Locates scene root objects by name: `Core`, `Props`, `World`, `Lights`, `Roads`, `UI`
- Creates singletons: `BridgeManager`, `SimulationService`, `SimulationWorld`, `ObjectSpawning`, `ModelImporter`, `Segmentation.Manager`, `MeshProcess.VHACD`

**`Start()` does:**
- Hard-fails if `AsyncGPUReadback` is not supported
- Parses command-line args: `-world` / `-worldFile`, `-capture`
- Builds `SDF.Root`, populates resource paths, calls `UpdateResourceModelTable()`
- Launches `LoadWorld()` coroutine if a world file was specified

### Layer 2: SDF parsing and import pipeline — `Tools/SDF/`

This layer has three sub-stages:

1. **Parser** (`Tools/SDF/Parser/`) — C# classes mirroring SDF XML elements: `Root`, `World`, `Model`, `Link`, `Joint`, `Visual`, `Collision`, `Sensor`, `Plugin`, `Actor`, `Light`, `Material`, `Physics`, etc. `Root.cs` handles XML loading, `<include>` model resolution, and path-to-absolute conversion. Supports SDF versions 1.0–1.9.

2. **Importer** (`Tools/SDF/Import/`) — Partial class `Loader` with files per element type (`Import.World.cs`, `Import.Model.cs`, `Import.Link.cs`, `Import.Joint.cs`, `Import.Sensor.cs`, `Import.Plugin.cs`, etc.). Converts parsed SDF into Unity GameObjects/components. Sensor import is a type-switch dispatching to extension methods like `AddLidar()`, `AddCamera()`, `AddImu()`, etc. Plugin import uses `Type.GetType(pluginLibraryName)` + `AddComponent` — plugins are resolved by class name derived from the SDF `filename` attribute.

3. **Implementer** (`Tools/SDF/Implement/`) — Static extension methods that create the actual Unity components (colliders, renderers, articulation joints, materials).

**Helper classes** (`Tools/SDF/Helper/`) — `Base`, `Model`, `Link`, `Visual`, `Collision`, `Actor` are MonoBehaviours attached to imported objects, holding SDF pose data and enabling reset.

**Utility** (`Tools/SDF/Util/`) — `SDF2Unity` provides coordinate system conversion (SDF right-hand → Unity left-hand: `X→Z`, `Y→-X`, `Z→Y`), material creation, and mesh utilities. `Unity2SDF` provides the reverse for world saving.

### Layer 3: Simulation runtime — `Core/`

| File | Role |
|------|------|
| `SimulationWorld.cs` | `CLOiSimPlugin`-derived world clock + reset signal broadcaster. Registers a Clock TX and Control client channel |
| `PluginStartTracker.cs` | Collects all `CLOiSimPlugin` instances under a loaded root, monitors their `Started` events, emits progress, fires `AllStartedEvent` when all are ready |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lge-ros2/cloisim](https://github.com/lge-ros2/cloisim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
