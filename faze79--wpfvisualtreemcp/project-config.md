---
trigger: always_on
description: transforms, works when the window is covered, but cannot see Popups/menus);
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**WpfVisualTreeMcp** is an MCP (Model Context Protocol) server that enables AI agents to inspect running WPF applications in real-time. It exposes WPF visual tree inspection capabilities similar to Snoop WPF or Visual Studio's Live Visual Tree through the MCP protocol.

## Build Commands

```bash
# Build solution
dotnet build WpfVisualTreeMcp.sln

# Build for Release
dotnet build -c Release WpfVisualTreeMcp.sln

# Run tests
dotnet test WpfVisualTreeMcp.sln

# Run sample WPF app for testing
dotnet run --project samples/SampleWpfApp

# Publish MCP Server executable (same exe also runs as the CLI)
dotnet publish src/WpfVisualTreeMcp.Server/WpfVisualTreeMcp.Server.csproj -c Release -o ./publish

# Run as CLI (any subcommand switches from MCP server mode to CLI mode)
dotnet run --project src/WpfVisualTreeMcp.Server -- list
./publish/WpfVisualTreeMcp.Server.exe help
```

## Architecture

```
AI Agent (Claude Code)
    ↓ [MCP Protocol - JSON-RPC over stdio]
MCP Server (.NET 8.0)
    ├─ WpfTools (28 tools)
    ├─ ProcessManager (discovers WPF processes)
    └─ NamedPipeBridge (IPC)
        ↓ [Named Pipes: wpf_inspector_{pid}]
Target WPF Application (.NET Framework)
    └─ Inspector DLL (.NET 4.8)
        ├─ TreeWalker (visual tree navigation + adorner/popup traversal)
        ├─ ScreenshotCapture (RenderTargetBitmap element capture)
        ├─ PropertyReader (dependency properties)
        ├─ BindingAnalyzer (binding inspection)
        ├─ ControlInteractor (UI Automation / physical click)
        └─ IpcServer (named pipe communication)
```

**Key Design:** Multi-process architecture for safety. The server runs separately and communicates via named pipes. All operations are read-only **except `wpf_click_element`, `wpf_select_item`, `wpf_set_text`, `wpf_send_keys`, `wpf_set_property`, and `wpf_revert_property`**, which drive controls or edit property values and change application state. `wpf_set_property` is reversible via `wpf_revert_property` (per-session undo stack that restores the prior binding, local value, or default).

## Key Source Locations

| Component | Location | Purpose |
|-----------|----------|---------|
| MCP Server Entry | `src/WpfVisualTreeMcp.Server/Program.cs` | Server init with MCP SDK; routes to CLI mode if args present |
| Tool Definitions | `src/WpfVisualTreeMcp.Server/WpfTools.cs` | 28 tools with `[McpServerTool]` attributes |
| CLI Front-End | `src/WpfVisualTreeMcp.Server/Cli/CliRunner.cs` | Command-line front-end over the same services (28 commands) |
| Trigger / Style Diagnostics | `ResourceInspector.ExplainTriggers` | Evaluate Style + ControlTemplate triggers vs current state (active/inactive + setters); attribute a property value to its style setter / active trigger |
| Control Interactor | `src/WpfVisualTreeMcp.Inspector/ControlInteractor.cs` | Clicks, text input, and keyboard shortcuts (UI Automation + SendInput physical fallback) |
| Property Writer | `src/WpfVisualTreeMcp.Inspector/PropertyWriter.cs` | Live-edits dependency properties (TypeConverter coercion) with a per-session undo stack (restores prior binding/local/default) |
| Snapshot / Diff | `TreeWalker.CaptureSnapshot` + `InspectorService.ComputeDiff` | Capture a subtree's curated state keyed by element handle; diff two snapshots (handle stable → changed/added/removed) |
| Injector Helper | `src/WpfVisualTreeMcp.InjectorHelper/Program.cs` | 32-bit .NET 8 helper exe spawned by `ProcessInjector` for cross-arch injection (v0.6.0) |
| IPC Bridge | `src/WpfVisualTreeMcp.Server/Services/NamedPipeBridge.cs` | Named pipe communication to Inspector |
| Process Manager | `src/WpfVisualTreeMcp.Server/Services/ProcessManager.cs` | WPF process discovery and attachment |
| Inspector Entry | `src/WpfVisualTreeMcp.Inspector/InspectorService.cs` | Injected DLL main entry point |
| Screenshot Capture | `src/WpfVisualTreeMcp.Inspector/ScreenshotCapture.cs` | RenderTargetBitmap element/window capture |
| IPC Server | `src/WpfVisualTreeMcp.Inspector/IpcServer.cs` | Named pipe server in target process |
| IPC Messages | `src/WpfVisualTreeMcp.Shared/Ipc/IpcMessages.cs` | Request/response contracts |
| Native Bootstrapper | `src/WpfVisualTreeMcp.Bootstrapper/WpfInspectorBootstrapper.cpp` | CLR hosting for DLL injection |
| Process Injector | `src/WpfVisualTreeMcp.Injector/ProcessInjector.cs` | CreateRemoteThread + LoadLibrary injection |

## Threading Model

- WPF apps are single-threaded (STA). All visual tree operations must run on the UI Dispatcher thread.
- Inspector wraps Dispatcher.Invoke in `Task.Run()` to avoid blocking the IPC thread.
- 10-second timeout for UI operations.
- `IpcServer` accepts **concurrent** pipe connections (each client handled on its own task);
  requests still serialize on the UI Dispatcher, so a long `wpf_wait_for` no longer blocks
  other commands.
- `wpf_wait_for` polls on the background thread (short `Dispatcher.Invoke` per check +
  `Task.Delay` between) — it is handled in `HandleRequestAsync` *before* the blocking
  Dispatcher.Invoke path, so the UI thread stays free and the awaited condition can change.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [faze79/WPFVisualTreeMcp](https://github.com/faze79/WPFVisualTreeMcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
