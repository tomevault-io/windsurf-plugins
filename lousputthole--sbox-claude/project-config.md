---
trigger: always_on
description: > Let non-coders build s&box games through conversation with Claude Code.
---

# s&box + Claude Code MCP Integration

> Let non-coders build s&box games through conversation with Claude Code.

## Status: 100 of 109 Tools Implemented

**Last updated:** 2026-04-26
**Bridge:** File-based IPC ✅ working on main thread
**Handlers:** 100 compiled and registered (of 109 MCP tools defined)
**Not implementable:** 9 tools (no s&box API exists — see "Known Issues")

### What's new in this update

- **Generic component-button driver** — `invoke_button` + `list_component_buttons` work on any component with a `[Button]` attribute. Plus `set_prefab_ref` for assigning prefab GameObjects to component properties.
- **Map editing primitives** — `add_terrain_hill`, `add_terrain_clearing`, `add_terrain_trail`, `clear_terrain_features` drive a `MapBuilder` component by mutating its editable `Hills`/`Clearings`/`Trails`/`CavePath` lists and rebuilding.
- **Sculpt brushes** — `sculpt_terrain` modifies the heightmap directly with raise/lower/flatten/smooth modes.
- **Cave + forest editing** — `add_cave_waypoint`, `clear_cave_path`, `add_forest_poi`, `add_forest_trail`, `set_forest_seed`, `clear_forest_pois`, `paint_forest_density`.
- **Path placement** — `place_along_path` drops asset instances along a curve.
- **Heightmap raycast** — `raycast_terrain` returns surface height at a world XY.
- **Type discovery** — `describe_type`, `search_types`, `get_method_signature`, `find_in_project` expose `Game.TypeLibrary` reflection so Claude can look up real APIs instead of guessing.
- **Better play-mode tracking** — `start_play` uses `EditorScene.Play` with `SetPlaying` fallback; `is_playing` tracks state via `PlayState` static + Game flag + active-scene divergence (fixes the prior "start_play triggers but is_playing returns false" issue).

---

## Architecture

```
Claude Code → (stdio) → MCP Server → (file IPC) → Bridge Addon → s&box Editor
                          Node.js        %TEMP%/sbox-bridge-ipc/     C# in Editor
```

**NOT WebSocket.** s&box's sandboxed C# environment does not allow `System.Net` (HttpListener, WebSocket, TcpListener). Communication uses **file-based IPC**:

1. MCP Server writes `req_<id>.json` to the temp directory
2. Bridge addon polls for request files via `System.Threading.Timer` (50ms)
3. Requests are queued and processed on the **main editor thread** (required for scene APIs)
4. Bridge writes `res_<id>.json` back
5. MCP Server polls for response files

IPC directory: `%TEMP%/sbox-bridge-ipc/` (typically `C:\Users\<user>\AppData\Local\Temp\sbox-bridge-ipc\`)

Two components:
1. **MCP Server** (`sbox-mcp-server/`) — TypeScript/Node.js, stdio transport, talks to Claude Code
2. **Bridge Addon** — C# editor library, lives in the s&box **project's Libraries folder**

---

## Critical Lessons Learned

### Addon Location
- **DO NOT** put addons in the global `sbox/addons/` folder — those are built-in only and won't compile custom code
- **DO** put addons in the project's `Libraries/` folder (e.g., `bigfoot/Libraries/claudebridge/`)
- s&box auto-scaffolds `Editor/`, `Code/`, `UnitTests/` with proper `.csproj` files when you create a library through the editor

### Compilation
- s&box compiles addons silently — if there are errors, **no log output appears** unless you check the full log
- Check `logs/sbox-dev.log` or the console for `Compile of 'local.X.editor' Failed:` messages
- The `.csproj` file is required and must reference s&box DLLs with absolute paths
- s&box generates the `.csproj` automatically when you create a library through the Library Manager

### Main Thread Requirement
- All scene manipulation APIs (`CreateObject`, `AddComponent`, `Destroy`, etc.) **must run on the main editor thread**
- `System.Threading.Timer` callbacks run on thread pool threads — NOT safe for scene APIs
- Solution: Timer reads files from disk (thread-safe), queues them, and a `[EditorEvent.Frame]` handler on a `[Dock]` widget processes them on the main thread

### Class Discovery
- s&box discovers classes via attributes like `[Menu]`, `[Dock]`, `[EditorEvent.Frame]`
- Static constructors fire when the type scanner discovers the class
- `[Event("editor.created")]` fires BEFORE custom addons load — don't rely on it
- `[Event("editor.loaded")]` does NOT exist

### s&box API Key Differences (vs. what was originally coded)
- `SceneEditorSession.Active.Scene` — the editor scene (NOT `Game.ActiveScene` which is for play mode)
- `go.AddComponent<T>()` — add component (NOT `go.Components.Create<T>()`, though ComponentList.Create also exists)
- `go.GetOrAddComponent<T>()` — get existing or add
- `go.GetComponent<T>()` — get existing
- `SceneEditorSession.Active.Selection` — editor selection (NOT `EditorScene.Selection`)
- `SceneEditorSession.Active.SetPlaying(scene)` / `.StopPlaying()` — play mode
- `SceneEditorSession.Active.FrameTo(bbox)` — focus camera on object
- `SceneEditorSession.Active.Save()` — save scene
- `Game.TypeLibrary.GetType("name")` — find types
- `Game.TypeLibrary.GetTypes<Component>()` — list all component types
- `MeshCollider` does NOT exist — use `HullCollider` instead
- `Rotation.Pitch()`, `.Yaw()`, `.Roll()` are methods, not properties

### Math & Events (s&box sandbox specifics)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LouSputthole/Sbox-Claude](https://github.com/LouSputthole/Sbox-Claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
