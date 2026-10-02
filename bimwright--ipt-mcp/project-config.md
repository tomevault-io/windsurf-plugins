---
trigger: always_on
description: Open-source (Apache-2.0) MCP gateway that lets Claude Code (and any MCP-capable client) drive Autodesk Inventor 2022-2027.
---

# ipt-mcp

Open-source (Apache-2.0) MCP gateway that lets Claude Code (and any MCP-capable client) drive Autodesk Inventor 2022-2027.

## Architecture

Full C# stack. No TypeScript. Single language, single build system. Multi-version: one in-process add-in per Inventor 2022-2027, all compiled from the same `src/shared/**` source glob.

```
MCP client (Claude Code / Cursor / Cline / …) → stdio → C# MCP Server (.NET 8 console)
   → TCP (2022-2024) or Named Pipe (2025-2027) → per-version Inventor add-in → Inventor API
```

Two processes:
- **Bimwright.Ipt.Server.exe** — MCP server, separate process, stdio transport (`ModelContextProtocol` 1.1.0). Has NO Inventor reference; it only compiles the API-agnostic contract files.
- **Bimwright.Ipt.Plugin.InvNN.dll** — `ApplicationAddInServer` add-in, loads inside `Inventor.exe`, runs a TCP or Named-Pipe listener and marshals every command onto Inventor's STA thread.

Communication: newline-delimited JSON (NDJSON). Per-version discovery files are written to `%LOCALAPPDATA%\Bimwright\ipt-mcp\inventor-<year>-<pid>.json`.

Discovery / target descriptor (`TargetDescriptor`):
```json
{
  "target_id": "inventor-2025-12345",
  "inventor_year": 2025,
  "process_id": 12345,
  "host_app": "Inventor",
  "transport": "pipe",
  "port": 0,
  "pipe_name": "BimwrightInventor-12345",
  "auth_token": "…",
  "document_title": "Part1.ipt",
  "document_path": "C:\\…\\Part1.ipt",
  "last_heartbeat_utc": "2026-05-29T12:00:00Z"
}
```

The server scans these files (`TargetRegistry.List()`), dropping any whose `process_id` is dead, whose `last_heartbeat_utc` is older than the max age (~120 s), whose `host_app != "Inventor"`, or whose year is outside 2022-2027. Use 4-digit calendar years (2022..2027), never legacy version codes. Agents should call `inventor_list_available_targets` to enumerate live instances and `inventor_get_current_target` to inspect the pinned one rather than guessing.

## Project Structure

```
src/
├── IptMcp.sln                 # Solution (server + tests; 6 add-ins added in Phase 2)
├── server/                         # MCP server (console, net8.0). NO Inventor reference.
│   ├── Bimwright.Ipt.Server.csproj   # explicit-includes shared/Contracts + Security (+ ToolBaker)
│   ├── Program.cs                  # boot + ResolveToolTypesForRegistration → WithTools(...)
│   ├── InventorMcpConfig.cs        # CLI/env/JSON config (toolsets, read-only, send-code, …)
│   ├── ToolsetFilter.cs            # KnownToolsets / DefaultOn / WriteCapable + Resolve()
│   ├── PluginClient.cs             # transport client (tcp+pipe), target selection
│   └── Tools/                      # [McpServerToolType] classes, one per domain
│       ├── MetaTools.cs            # 3 server-side target tools (list/get/switch)
│       ├── QueryTools.cs  DocumentTools.cs
│       ├── ParameterTools.cs  PropertyTools.cs  SketchTools.cs  FeatureTools.cs
│       ├── ExportTools.cs  CodeTools.cs  ToolBakerTools.cs  ToolBakerWriteTools.cs
├── shared/
│   ├── Contracts/                  # API-AGNOSTIC — server explicit-includes; add-ins glob too
│   │   ├── InventorCommandEnvelope.cs  InventorCommandResult.cs  InventorErrorCodes.cs
│   │   ├── TargetDescriptor.cs  TargetRegistry.cs  ResponseSizeGuard.cs
│   ├── Security/                   # ErrorSanitizer, SecretMasker
│   ├── Infrastructure/             # PLUGIN-ONLY glob: IInventorCommand, InventorCommandContext,
│   │                               #   CommandDispatcher, IsExternalInit polyfill (net48)
│   ├── Transport/                  # ITransportServer, TcpTransportServer, PipeTransportServer, AuthToken
│   ├── Plugin/                     # InventorAddInServer, InventorStaDispatcher, registry, descriptor writer
│   └── Handlers/                   # one file per wire command (Phase 2-3)
├── plugin-inv22/ inv23/ inv24/     # net48, TCP transport
├── plugin-inv25/ inv26/            # net8.0-windows7.0, Named Pipe
└── plugin-inv27/                   # net10.0-windows7.0, Named Pipe
tests/Bimwright.Ipt.Tests/     # net8.0, xUnit. Server-only — no Inventor needed.
```

The **server** compiles only `shared/Contracts/*` + `shared/Security/*` (and ToolBaker) explicitly, so it builds with no Inventor SDK present. Each **add-in** uses `<Compile Include="..\shared\**\*.cs" />` to pull in everything, including the API-touching `Infrastructure`/`Plugin`/`Handlers`.

## Build & Test

```bash
# Server + tests (server-only; no Inventor required, works on any machine with the .NET 8 SDK):
dotnet build src/IptMcp.sln -c Debug
dotnet test  tests/Bimwright.Ipt.Tests -c Debug

# Legacy TFM compatibility check using the installed 2027 interop reference:
dotnet build src/plugin-inv24 -c Debug /p:InventorInteropDir="C:\Program Files\Common Files\Autodesk Shared\Extensions 2027\Framework\Interop"

# Real 2027 add-in compile (needs the .NET 10 SDK and installed Inventor 2027 interop):
dotnet build src/plugin-inv27 -c Debug
```

Inventor must be CLOSED before deploying add-in DLLs it would otherwise lock. The Inventor-API handlers have been smoke-tested against a live Inventor session; hold new handler bodies to the same verification bar before calling them done.

## Multi-Version Matrix


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bimwright/ipt-mcp](https://github.com/bimwright/ipt-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
