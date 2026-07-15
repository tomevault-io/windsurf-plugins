---
trigger: always_on
description: > 📖 **New here?** Read [AI.md](AI.md) first for project context.
---

> 📖 **New here?** Read [AI.md](AI.md) first for project context.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

```bash
# Initialize submodules (W3C test data — required for dotnet test)
./tools/update-submodules.sh   # or .\tools\update-submodules.ps1

# Build entire solution
dotnet build SkyOmega.sln

# Build specific project
dotnet build src/Mercury/Mercury.csproj

# Release build (enables optimizations)
dotnet build -c Release

# Run tests (xUnit)
dotnet test

# Run specific test
dotnet test --filter "FullyQualifiedName~BasicSelect"

# Run benchmarks (BenchmarkDotNet)
dotnet run --project benchmarks/Mercury.Benchmarks -c Release

# Run specific benchmark class
dotnet run --project benchmarks/Mercury.Benchmarks -c Release -- --filter "*Storage*"

# List available benchmarks
dotnet run --project benchmarks/Mercury.Benchmarks -c Release -- --list

# Run examples
dotnet run --project examples/Mercury.Examples
dotnet run --project examples/Mercury.Examples -- storage
dotnet run --project examples/Mercury.Examples -- temporal
dotnet run --project examples/Mercury.Examples -- demo
```

## File-Based Apps (.NET 10)

For throwaway scripts, one-off debugging, test data generation, or quick repro cases, use file-based apps instead of creating a full project. Write a single `.cs` file and run it directly:

```csharp
#!/usr/bin/env -S dotnet
#:project ../src/Mercury/Mercury.csproj

// your code here
```

```bash
chmod +x script.cs
./script.cs          # or: dotnet script.cs
```

Use `#:package Name@version` for NuGet references, `#:project path` for project references, `#:sdk Microsoft.NET.Sdk.Web` for web apps. Do not add file-based scripts to the solution — they are standalone by design.

## Global Tools

Sky Omega tools are packaged as .NET global tools for use from any directory.

```bash
# Install all tools from local source
./tools/install-tools.sh

# Or install individually from local nupkg
dotnet pack SkyOmega.sln -c Release -o ./nupkg
dotnet tool install -g SkyOmega.Mercury.Mcp --add-source ./nupkg
```

| Command | Description |
|---------|-------------|
| `mercury` | SPARQL CLI with persistent store at `~/Library/SkyOmega/stores/cli/` |
| `mercury-mcp` | MCP server for Claude with persistent store at `~/Library/SkyOmega/stores/mcp/` |
| `mercury-sparql` | SPARQL query engine demo |
| `mercury-turtle` | Turtle parser demo |
| `drhook-mcp` | MCP server for .NET runtime inspection (EventPipe + ICorDebug via `DrHook.Engine`, BCL + P/Invoke + per-RID `libdbgshim`) |

All tools support `-v`/`--version`.

### MCP Integration

**Mercury** runs from the **global Release tool** (`mercury-mcp`) by default — substrate hardening closed at 1.7.69, so version skew is no longer the dev-iteration concern it was during the cycle 8 → cycle 10 arc. **DrHook** still runs from the **local source build** (`dotnet run --project src/DrHook.Mcp`) because the DrHook substrate is the active 1.8.x development target — ADR-006 Phase 3 closed at 1.8.2 (substrate-independence reached), and [ADR-007](docs/adrs/drhook/ADR-007-teardown-concurrency-test-debug.md) (Proposed 2026-05-23) sequences production-suitability work (teardown + concurrency hardening, test-runner debugging substrate, integration-test mechanism, cross-platform validation). Iteration speed matters here; the global tool would lag. To iterate on Mercury MCP code itself, manually edit `.mcp.json` to point at `dotnet run --project src/Mercury.Mcp`. The **debug-state visualization** views (ADR-012 Phase 2 — `drhook-viz-console`, run via `dotnet run --project src/DrHook.Viz.Console`) are standalone, **human-launched** processes that connect to the active session's rendezvous socket and render the live debug-state; they are not part of the MCP surface, and terminating a view never affects the LLM-owned debug session.

See **[DRHOOK.md](DRHOOK.md)** for the DrHook debugging workflow (observe-before-fixing), the 23-tool MCP reference, how to run each test kind, and the probe corpus — the runtime-observation counterpart to [MERCURY.md](MERCURY.md). **Before driving DrHook, read its "Lifecycle discipline & common pitfalls" section** — DrHook is the first debugger built for an LLM operator, so there is no training corpus on using one and that section is the training surface: no `Debugger.Break()` crutch (the launch hold-gate arms breakpoints pre-main for `dotnet exec`); end sessions with `drhook_stop`, not `drhook_kill` (kill is the anomaly escape hatch); clean up orphaned targets you can see in `drhook_processes`; **never mix runtime versions in one session** — a DrHook debugger process locks to the first target's runtime version, so debugging a net11 file-based app (`dotnet run x.cs` builds net11 under SDK 11) and then a net10 target (the substrate targets net10.0) fails with `0x80131C3C` (`CORDBG_E_DEBUG_COMPONENT_MISSING`); pin probes with `#:property TargetFramework=net10.0` or reconnect the MCP when switching runtime version (finding 86).

**Dev-time** (this repo): `.mcp.json` at repo root auto-configures Claude Code. After `tools/install-tools.sh` updates the Mercury global tool, restart Claude Code to spawn a fresh MCP process against the new version.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bemafred/sky-omega](https://github.com/bemafred/sky-omega) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
