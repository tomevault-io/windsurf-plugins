---
trigger: always_on
description: Unreal-MCP bridges LLMs (Claude, Cursor, Copilot, …) with [Unreal Engine](https://www.unrealengine.com/)
---

# CLAUDE.md — Unreal-MCP

Unreal-MCP bridges LLMs (Claude, Cursor, Copilot, …) with [Unreal Engine](https://www.unrealengine.com/)
via the [Model Context Protocol](https://modelcontextprotocol.io/) — the Unreal-engine sibling of
Unity-MCP and Godot-MCP. It works in **two modes**: an **editor** mode (the default — drive the Unreal
**Editor**: spawn actors, edit levels, author/compile Blueprints, edit/compile C++, capture screenshots,
…) and an opt-in **in-game runtime** mode (drive a running PIE / Standalone / packaged **Development**
build so an AI can operate your live game). **Status: beta** — the plugin, the .NET sidecar, the
`unreal-mcp-cli`, the AI Game Developer editor UI, and **61 built-in editor tools across 7 families**
(plus 3 §2.4 system tools)
have shipped and are covered by CI. The `unreal-mcp-cli` is **published on npm** (the Fab / Epic
Marketplace listing for the precompiled plugin is coming soon). The local MCP server is the shared,
engine-agnostic [GameDev-MCP-Server](https://github.com/IvanMurzak/GameDev-MCP-Server)
(binary `gamedev-mcp-server`) — no server source lives in this repo.

**The authoritative design is [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md)** (defer to it on any
conflict). Read the relevant section before implementing anything non-trivial — it locks the IPC
protocol (§1), dynamic tool registration (§2), schema generation (§3), the game-thread dispatcher (§4),
extensions (§5), sidecar lifecycle (§6), UI (§7), config (§8), repo/versioning/CI (§9), the tool-family
roadmap (§10), risks (§11), **runtime (in-game) support (§12)**, and custom prompts & resources (§A).
The user-facing entry point is [`README.md`](README.md); the release runbook is
[`docs/RELEASING.md`](docs/RELEASING.md); the extension author guide is
[`docs/EXTENSIONS.md`](docs/EXTENSIONS.md); the in-game operator walkthrough is
[`docs/RUNTIME-E2E.md`](docs/RUNTIME-E2E.md).

## Architecture in one screen

Unlike Unity/Godot (C# engines that host the .NET `McpPlugin`/ReflectorNet/SignalR stack **in-process**),
Unreal's editor and runtime are **C++** and cannot host that .NET stack. So the McpPlugin host runs as an
auto-managed **.NET 9 sidecar process** (`unreal-mcp-bridge`, `bridge/`) that is a **local child of
whichever host loads the plugin** — the **Editor** in dev, or the **game process itself** in an opt-in
packaged build. It is never remote: the C++ plugin **listens** on a loopback TCP port and the sidecar
**dials** it, authenticating with a one-shot token delivered over **stdin** (never argv — §1.4). The
sidecar relays IPC ⇄ SignalR to the MCP server (cloud `ai-game.dev` by default, or a local
`gamedev-mcp-server`). See the §0 system-overview diagram.

The plugin is split into two modules (`UnrealMCP/UnrealMCP.uplugin`, runtime FIRST so it loads before
the editor module that depends on it):

- **`UnrealMcpRuntime`** (Type **Runtime**, LoadingPhase Default) — the dependency-lean, engine-agnostic
  machinery that cooks into packaged games: the tool **registry**, the game-thread **dispatcher**, the
  loopback-TCP **bridge server** + NDJSON framing, the **sidecar manager**, the world provider, the
  **extension manager**, config, the `IUnrealMcp*Provider` contracts + builders, and the runtime built-in
  `ping`. NO UnrealEd / Slate / asset / blueprint / capture deps (deps: Core/CoreUObject/Engine/
  DeveloperSettings public; Networking/Sockets/Json/JsonUtilities/Projects private). Also home to the
  runtime bootstrap (`UUnrealMcpRuntimeSubsystem`) + kill-switch settings (`UUnrealMcpRuntimeSettings`).
- **`UnrealMcpEditor`** (Type **Editor**, LoadingPhase Default) — depends on `UnrealMcpRuntime` and layers
  the Slate UI + the **61 editor-only tools** (the 7 families below, several RCE-class) + the
  `unreal-skill-create` system tool (§2.4) on the SAME registry, plus the local-server manager and the
  dev-control bridge.

> Three locked decisions (ARCHITECTURE §12): (a) reuse the .NET sidecar — no in-process C++ server;
> (b) the `UnrealMcpRuntime` module holds the engine-agnostic infra + the runtime-safe `ping`, and
> `UnrealMcpEditor` depends on it; (c) a runtime connection is **explicit opt-in** — a shipped game NEVER
> auto-connects. The 61 engine-development tools are all **editor-only**; a game brings its OWN gameplay
> tools via `IUnrealMcpToolProvider` (author guide: [`docs/EXTENSIONS.md`](docs/EXTENSIONS.md)).

## Layout

| Path | What it is |
| --- | --- |
| `UnrealMCP/` | The UE **plugin**. `UnrealMCP/UnrealMCP.uplugin` `VersionName` is the version single-source (currently **`0.6.1`**). **No `EngineVersion` pin** (UE treats that field as an exact-build match, not a floor, and would refuse to load on newer engines); the **5.5+ floor** is a CI/doc claim (CI-tested against **5.7** and **5.8**). Declared modules: **`UnrealMcpRuntime` (Type `Runtime`)** + **`UnrealMcpEditor` (Type `Editor`)**, both LoadingPhase `Default`, runtime first |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IvanMurzak/Unreal-MCP](https://github.com/IvanMurzak/Unreal-MCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
