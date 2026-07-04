---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

### Setup (one-time)
```powershell
mkdir deps
# Copy these DLLs from a dnSpy installation's bin/ folder:
#   dnSpy.Contracts.DnSpy.dll
#   dnSpy.Contracts.Logic.dll
#   ICSharpCode.Decompiler.dll
#   dnlib.dll
```

### Local Development
```powershell
# Build only (Release default)
dotnet build src/dnSpy.MCP/dnSpy.MCP.csproj -c Release

# Deploy extension (requires dnSpy closed)
pwsh scripts/build.ps1 -DnSpyPath "D:\tools\dnSpy" -Deploy
```

Options: `-Clean`, `-Deploy`, `-DeployDir <path>`, `-Configuration <Debug|Release>`

### CI
GitHub Actions (`build.yml`) auto-downloads dnSpy deps and runs `dotnet build -c Release`. No manual setup needed.

**Tool-count guard**: after adding/removing a tool, run `pwsh scripts/verify-tool-count.ps1`. It cross-checks the count discovered by the same reflection rules `ToolRegistry` uses against the `## Available MCP Tools (NN)` header here — fails on drift so docs and code can't silently diverge.

### Build output
- **Release DLL**: `src/dnSpy.MCP/bin/Release/net10.0-windows/dnSpy.MCP.x.dll`
- **Deploy to dnSpy**: copy `dnSpy.MCP.x.dll`, `.deps.json`, `.pdb` (optional) to `<dnSpy>/bin/Extensions/`

## Project Layout

```
dnspy_mcp/
├── src/dnSpy.MCP/           # Standalone extension project
│   ├── TheExtension.cs       # MEF [ExportExtension] entry point
│   ├── DnSpyContext.cs       # Static service bridge + lazy IServiceLocator resolution
│   ├── MenuCommands.cs        # MCP Server menu items
│   ├── Mcp/
│   │   ├── McpServerHost.cs  # TcpListener + JSON-RPC 2.0 dispatch
│   │   ├── ToolRegistry.cs   # Reflection-based tool discovery
│   │   └── McpLogger.cs     # File + Output Window logging
│   ├── Settings/             # dnSpy Options integration
│   │   ├── McpSettings.cs           # Port, host, auth, timeout, etc.
│   │   ├── McpSettingsPage.cs       # Options dialog integration
│   │   ├── McpSettingsControl.xaml  # Settings UI
│   │   └── McpSettingsControl.xaml.cs
│   ├── Tools/                # 14 tool classes, 38 tools
│   │   ├── IlDisplayTools.cs # IL opcode formatting (read-only)
│   │   ├── IlPatchTools.cs   # IL patching via Roslyn compilation
│   └── Helpers/
│       ├── MethodResolver.cs
│       └── TextDecompilerOutput.cs
├── deps/                     # dnSpy DLL references
└── scripts/build.ps1
```

## Architecture

### Why HttpListener Instead of MCP SDK?
The official MCP SDK 1.2.0 pulls `Microsoft.Extensions.*` 10.x which may conflict with dnSpy's transitive dependencies on .NET 10. Solution: custom HTTP transport via `System.Net.HttpListener`.

### Extension Lifecycle
```
dnSpy starts
  → MEF discovers dnSpy.MCP.x.dll
  → TheExtension constructor: [Import] gets services
  → OnEvent(ExtensionEvent.AppLoaded): DnSpyContext.Initialize(...) + EnsureOutputPane()
  → User clicks Start (or AutoStart=true in Settings) → HttpListener starts
```

Server starts on **manual click** (not at launch) so `EnsureOutputPane()` runs on a fully initialized WPF UI thread.

### Tool Discovery
Tools are `public static` methods in `dnSpy.MCP.Tools.*` with `[Description("...")]`. `ToolRegistry.DiscoverTools()` scans via reflection. Tool names auto-convert to `snake_case`.

### Service Access
`DnSpyContext.cs` is a static singleton bridging MEF to MCP tools. Three access patterns:

**Direct services** (always available after init):
```csharp
DnSpyContext.DocumentService
DnSpyContext.DecompilerService
```

**Cached resolver** (shared `MethodResolver`, lazy-initialized):
```csharp
DnSpyContext.Resolver  // returns MethodResolver backed by DocumentService
```
Tools should use `DnSpyContext.Resolver` instead of creating `new MethodResolver()`.

**Lazy services** (resolved via `IServiceLocator.TryResolve<T>()` on first access):
```csharp
DnSpyContext.TabService     // IDocumentTabService
DnSpyContext.TreeView       // IDocumentTreeView
```

`IServiceLocator` is imported via MEF and passed to `DnSpyContext.Initialize()` at `AppLoaded`. The lazy pattern avoids MEF import order issues with these services.

### Method Resolution
All method-accepting tools use `MethodResolver.ResolveMethodFlexible(string identifier)` which tries in order:
1. Hex token (`0x...`)
2. Plain integer token
3. Full name (`Namespace.Class::Method`)
4. Fallback short name search (returns **first** match)

Do NOT duplicate this logic — call `DnSpyContext.Resolver.ResolveMethodFlexible()`.

### Assembly Scoping
dnSpy can open multiple binaries simultaneously. To avoid ambiguous results:
- **`load_assembly`** — load a DLL/EXE into dnSpy programmatically (no manual UI step).
- **`close_assembly`** — unload an assembly by name.
- **`list_loaded_assemblies`** — always call first to know which binaries are loaded.
- **`assembly` parameter** — search tools (`search_types`, `search_methods`, `search_strings`, `grep`, `search_constants`, `get_xrefs_to`) accept an optional `assembly` parameter to scope results to a specific binary. When omitted, all loaded assemblies are searched.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tuna1999/dnspy-mcp](https://github.com/tuna1999/dnspy-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
