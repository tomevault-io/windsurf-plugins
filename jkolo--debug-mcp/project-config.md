---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

debug-mcp is an MCP server that exposes .NET debugging as 40 structured tools. It interfaces directly with the .NET runtime via ICorDebug APIs (through the ClrDebug NuGet wrapper) — the same approach JetBrains Rider uses. AI agents connect via Model Context Protocol to launch processes, set breakpoints, step through code, inspect variables, and analyze exceptions.

## Build & Test Commands

```bash
dotnet build                    # Build everything (0 errors, 0 warnings expected)
dotnet build -c Release         # Release build

# Reliable tests only (unit + contract — use this for development)
dotnet test tests/DebugMcp.Tests --no-build --filter "FullyQualifiedName~Unit|FullyQualifiedName~Contract"

# Run a single test class
dotnet test tests/DebugMcp.Tests --no-build --filter "FullyQualifiedName~ToolAnnotationTests"

# Run a single test method
dotnet test tests/DebugMcp.Tests --no-build --filter "FullyQualifiedName~ToolAnnotationTests.Tool_Title_MatchesSpec"

# Run the tool locally
dotnet run --project DebugMcp
```

**Do NOT run** integration or performance tests without reason — they require a live debugger and are timing-sensitive:
- `ReattachmentTests.ReattachAfterProcessTermination_ShouldSucceed` — flaky
- `TerminateLaunchedProcessTests.GetLoadedModules_OnRunningLaunchedProcess_ShouldNotHang` — timing-dependent
- `ModulePerformanceTests.GetTypesAsync_WithPagination_ReturnsWithin2Seconds` — performance flaky

## Tech Stack

- **Runtime**: .NET 10.0 (pinned in `global.json`), C#, cross-platform (Windows/macOS/Linux, x64/arm64)
- **Debugging**: ClrDebug 0.3.4 (ICorDebug wrappers), DbgShim 9.0
- **MCP SDK**: ModelContextProtocol 1.2.0
- **Code Analysis**: Roslyn (Microsoft.CodeAnalysis 5.3.0)
- **Tests**: xUnit + FluentAssertions + Moq
- **E2E**: Reqnroll (Gherkin BDD) in `tests/DebugMcp.E2E/`
- **Docs site**: Docusaurus 3.9.2 in `website/`

## Architecture

### Layer Diagram

```
MCP Client (Claude, GPT, etc.)
    ↓ stdio (JSON-RPC)
Program.cs — DI container, MCP server setup, CLI options
    ↓
Tools/ (40 tools)          Resources/ (4 resources)       Completions/
    ↓                           ↓                              ↓
Services/
├── DebugSessionManager    — Session lifecycle (launch/attach/disconnect)
├── ProcessDebugger        — Low-level ICorDebug, callbacks, locking
├── Breakpoints/
│   ├── BreakpointManager  — Hit logic, condition eval, exception matching
│   ├── BreakpointRegistry — In-memory store (thread-safe)
│   ├── BreakpointNotifier — Channel<T> async notification queue
│   └── PdbSymbolReader    — Source location resolution from PDB
├── ExceptionAutopsyService — Full exception chain analysis
├── CodeAnalysis/          — Roslyn workspace (go-to-def, find usages)
└── Symbols/               — SSQP symbol server client + disk cache
    ↓
ClrDebug (ICorDebug) → .NET Runtime
```

### ICorDebug Callback Threading (Critical)

ProcessDebugger uses **two locks** — getting this wrong causes deadlocks:

- **`_lock`** — protects user API calls (Launch, Continue, Stop, etc.)
- **`_stateLock`** — protects state fields updated by ICorDebug callbacks

**Lock ordering invariant**: `_lock` → `_stateLock` is OK. **Reverse is FORBIDDEN.** ICorDebug callbacks fire synchronously on the ICorDebug thread while other code may hold `_lock` during `Stop()` or `Continue()`. Callbacks must never acquire `_lock`.

### Event Flow: Breakpoint Hit

```
ICorDebug thread → OnBreakpoint callback
  → UpdateState(Paused) under _stateLock only
  → Fire BreakpointHit event
    → BreakpointManager.OnBreakpointHit
      → Resolve location from PDB
      → Evaluate condition (if any)
      → If tracepoint: queue notification via Channel<T>
      → Set ShouldContinue flag on EventArgs
  → If ShouldContinue: call Continue(false) and UpdateState(Running)
  → If !ShouldContinue: stay paused (session manager signals waiters)
```

### Tool Registration

Tools are discovered via reflection: classes with `[McpServerToolType]`, methods with `[McpServerTool]`. Each tool receives dependencies via constructor DI. Tools return JSON strings with `{success: true/false, ...}` structure.

### Key Model Conventions

- **Positional records** for all models: `record Breakpoint(string Id, BreakpointLocation Location, ...)`
- **Immutable state** via `with` expressions: `var updated = bp with { HitCount = bp.HitCount + 1 };`
- **DateTimeOffset everywhere** — never use `DateTime`
- **ID prefixes**: breakpoints `bp-{guid}`, tracepoints `tp-{guid}`, exception breakpoints `ebp-{guid}`

## Project Layout

```
DebugMcp/                        # Main project (packaged as dotnet tool)
├── Program.cs                   # Entry point, DI, MCP server config
├── Tools/                       # 36 MCP tool classes
├── Services/                    # Core business logic
├── Models/                      # Positional records (Breakpoints/, Inspection/, Memory/, Modules/)
└── Infrastructure/              # Logging, MCP logger provider

tests/
├── DebugMcp.Tests/              # Main test project
│   ├── Unit/                    # Unit tests (mock-based)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jkolo/debug-mcp](https://github.com/jkolo/debug-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
