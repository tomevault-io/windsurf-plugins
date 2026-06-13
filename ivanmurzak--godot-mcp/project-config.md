---
trigger: always_on
description: Godot-MCP bridges LLMs (Claude, Cursor, Copilot, …) with the [Godot](https://godotengine.org/) editor
---

# CLAUDE.md — Godot-MCP

Godot-MCP bridges LLMs (Claude, Cursor, Copilot, …) with the [Godot](https://godotengine.org/) editor
via the [Model Context Protocol](https://modelcontextprotocol.io/) — the Godot-engine sibling of Unity-MCP.
Sub-projects: `addons/godot_mcp/` (the Godot **editor addon**, a `Godot.NET.Sdk` C# project), `cli/` (the
`godot-cli` npm package — the Godot analog of `unity-mcp-cli`), and `Godot-MCP.Tests/` (the CI xUnit
suite). The MCP server is NOT in this repo — the addon consumes the shared
[GameDev-MCP-Server](https://github.com/IvanMurzak/GameDev-MCP-Server). See **CLI (godot-cli)** and
**Server (shared GameDev-MCP-Server)** below.

The addon is the heart of the repo. A `[Tool]` `EditorPlugin` (`GodotMcpPlugin`) boots on editor load,
installs the main-thread dispatcher, builds a `Reflector` with Godot type converters, and connects to an
MCP server (cloud `ai-game.dev` by default, or a custom local server) over the reused
`com.IvanMurzak.McpPlugin` SignalR client. The MCP/reflection stack is **not forked** — it is consumed
from nuget.org as `PackageReference`s and the pins are owned by the upstream release pipelines (never bump
them here). The reused pins are frozen at `com.IvanMurzak.ReflectorNet` **5.3.1** and
`com.IvanMurzak.McpPlugin` **6.7.0** (in `Godot-MCP.csproj`, mirrored by `Godot-MCP.Tests/` and the infra
testbed) — keep all three in lockstep; never bump here.

## Tool families

Tools live in `addons/godot_mcp/Tools/` — one `[AiToolType]` `partial class Tool_<Family>` per family,
with each tool method (`[AiTool("<name>", ...)]` + `[Description]`) in its own partial-class file. Tool
names mirror Unity-MCP where sensible. The 10 families:

| Family (class) | Tools | `#if TOOLS`? |
| --- | --- | --- |
| `Tool_Ping` | `ping` | no (pure-managed) |
| `Tool_Node` | `node-find`/`-create`/`-modify`/`-set-parent`/`-duplicate`/`-delete` | yes (editor) |
| `Tool_Scene` | `scene-open`/`-save`/`-create`/`-list-opened`/`-get-data` | yes (editor) |
| `Tool_Resource` | `resource-find`/`-get-data`/`-modify`/`-create`/`-move`/`-delete` | yes (editor) |
| `Tool_FileSystem` | `filesystem-list`/`-reimport` | yes (editor) |
| `Tool_Script` | `script-read`/`-create`/`-update`/`-delete`/`-attach-to-node` | yes (editor) |
| `Tool_Screenshot` | `screenshot-viewport`/`-camera`/`-isolated` | yes (editor) |
| `Tool_Editor` | `editor-application-get-state`/`-set-state`, `editor-selection-get`/`-set` | yes (editor) |
| `Tool_Console` | `console-get-logs`/`-clear-logs` | no (pure-managed collector) |
| `Tool_Reflection` | `reflection-method-find`/`-call` | no (engine-agnostic ReflectorNet) |

Editor-driving families live behind `#if TOOLS` (they touch `EditorInterface`/live `Node`/`Resource`);
their pure-managed result models + helpers live OUTSIDE the guard so they are CI-unit-testable. The
ping/console/reflection families have no editor-API surface and stay fully outside `#if TOOLS`.

## Build / test

Three suites (mirrors the `godot-mcp` implement-task profile `test.md`):

```bash
# Suite 1 — build (CI gate; 0 errors required)
dotnet restore Godot-MCP.sln
dotnet build  Godot-MCP.sln --configuration Debug --no-restore

# Suite 2 — unit tests (xUnit; 0 failures required, also runs in CI)
dotnet test   Godot-MCP.Tests/Godot-MCP.Tests.csproj --configuration Debug --no-build

# Suite 3 — headless live-editor smoke (operator/reviewer check, NOT a CI gate; see testbed runbook below)
```

`Godot.NET.Sdk` is a NuGet SDK, so **no Godot binary is needed for Suite 1 or 2**. The xUnit
project compiles a CI-friendly subset of the addon sources directly (it never `ProjectReference`s the
game project; the game `.csproj` excludes `Godot-MCP.Tests/**` so its xUnit files don't leak into the
game assembly). Only **pure-managed** Godot types are unit-testable here — Godot types that wrap a native
object (`NodePath`, `Node`, `Resource`, `SceneTree`) call `godotsharp_*` P/Invoke on construction and
crash the test host with `AccessViolationException` when no Godot native lib is loaded; verify those via
the headless Godot smoke (Suite 3) instead.

## CLI (godot-cli)

`cli/` publishes the **`godot-cli`** npm package — the Godot analog of `unity-mcp-cli`. It is a Node
(`^20.19.0 || >=22.12.0`) TypeScript CLI that resolves and launches the Godot editor with the right
`GODOT_MCP_*` connection env vars, runs MCP/system tools over the server's HTTP API (`POST
<url>/api/tools/<tool>`), probes server health, writes AI-agent MCP-client config, and enables/disables the
`godot_mcp` addon in a project. It also exports a side-effect-free library (`import { openProject, runTool,
… } from 'godot-cli'`) whose functions return a `{ kind: 'success' | 'failure' }` discriminated union and
never throw past the public boundary. Unlike the Unity CLI there is **no `setup-skills` command** — Godot
skills are generated addon-side on plugin boot, so there is nothing for the CLI to invoke.

```bash
cd cli
npm install
npm run build   # tsc → dist/ (ESM)
npm test        # vitest
```

**Find detail in** [`cli/README.md`](cli/README.md) — full command table, editor-resolution order,
connection env vars, and server-URL resolution.

## Server (shared GameDev-MCP-Server)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IvanMurzak/Godot-MCP](https://github.com/IvanMurzak/Godot-MCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-13 -->
