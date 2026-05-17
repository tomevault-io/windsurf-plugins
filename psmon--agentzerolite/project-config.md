---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & test

Target framework is **.NET 10** (preview). The WPF host is `net10.0-windows`; shared logic (`ZeroCommon`) is `net10.0` and must remain WPF/Win32-free so its headless xUnit suite can run.

```bash
# Build the WPF app (pulls ZeroCommon via ProjectReference)
dotnet build Project/AgentZeroWpf/AgentZeroWpf.csproj -c Debug

# Release build — auto-bumps version.txt (patch+1, 9→minor+1) in a post-build target
dotnet build Project/AgentZeroWpf/AgentZeroWpf.csproj -c Release

# Headless tests — fast, no desktop session needed
dotnet test Project/ZeroCommon.Tests/ZeroCommon.Tests.csproj

# WPF-dependent tests (actors, ConPTY session, approval parser) — needs desktop session
dotnet test Project/AgentTest/AgentTest.csproj

# Run a single test
dotnet test Project/ZeroCommon.Tests/ZeroCommon.Tests.csproj --filter "FullyQualifiedName~ApprovalParserTests.ParsesBashPrompt"

# Launch the GUI
Project/AgentZeroWpf/bin/Debug/net10.0-windows/AgentZeroLite.exe

# Drive a running GUI from any shell (exe double-dispatches on `-cli`)
Project/AgentZeroWpf/bin/Release/net10.0-windows/AgentZeroLite.exe -cli status
```

There is a third build configuration `AgentCLI` declared alongside Debug/Release — leave it in the csproj even though the runtime CLI/GUI split is decided inside `App.OnStartup` by checking `-cli` in args, not by configuration.

## ⚠️ IDE debugging — disable integrated terminal attachment

The app hosts real ConPTY terminals inside WPF tabs. If the IDE attaches its own console to the process's stdio, it intercepts the console events ConPTY needs to own and tabs either refuse to start or render garbled output.

- **Rider**: Run/Debug config → *Use external console = ON*
- **Visual Studio**: Debug properties → uncheck *Use the standard console* / *Redirect standard output*
- **VS Code**: `launch.json` → `"console": "externalTerminal"` (not `internalConsole`)

`dotnet run` from a shell works because it doesn't steal stdio.

## Architecture

Two C# projects with a strict dependency rule: **`AgentZeroWpf → ZeroCommon`, never reversed.** Anything that doesn't need WPF/Win32 APIs belongs in `ZeroCommon` so it stays testable headlessly. `AgentTest` references both; `ZeroCommon.Tests` references only `ZeroCommon`.

### Single exe, two modes
`AgentZeroLite.exe` is a WinExe (assembly name set in `AgentZeroWpf.csproj`; project/namespace stay `AgentZeroWpf.*`). In `App.OnStartup` (`Project/AgentZeroWpf/App.xaml.cs`):
- If args contain `-cli`, `CliHandler.Run` takes over, calls `Environment.Exit`, and the WPF message loop never starts.
- Otherwise a named mutex `Local\AgentZeroLite.SingleInstance` guards single-instance GUI, then `ActorSystemManager.Initialize()` builds the Akka system before `MainWindow` shows.

### Actor topology (Akka.NET)
All message types live in one file: `Project/ZeroCommon/Actors/Messages.cs`. The hierarchy is:

```
/user/stage                       StageActor      — supervisor + message broker
    /bot                          AgentBotActor   — UI gateway: chat/key mode, peer routing,
                                                    intro tracking. Spawns AgentLoopActor lazily.
        /loop                     AgentLoopActor  — THE agent: owns one IAgentLoop, drives
                                                    Idle→Thinking→Generating→Acting→Done FSM.
    /ws-<name>                    WorkspaceActor  — one per workspace (folder)
        /term-<id>                TerminalActor   — wraps one ITerminalSession
```

`ITerminalSession` (`Project/ZeroCommon/Services/ITerminalSession.cs`) is the seam between actors (logic, testable) and `ConPtyTerminalSession` (WPF-side, in `AgentZeroWpf/Services/`). Actor-layer code must not import WPF; if you need to reach the UI, go through `AgentEventStream` / `SetBotUiCallback`.

**Agent vocabulary (M0013)** — the agent loop layer (`Project/ZeroCommon/Llm/Tools/`) uses canonical "agent loop" naming aligned with the Anthropic *Building effective agents* post + Claude Agent SDK: `IAgentLoop` (backend-agnostic contract), `LocalAgentLoop` / `ExternalAgentLoop` (LLamaSharp+GBNF / OpenAI-compat REST), `IAgentToolbelt` (side-effect surface), `AgentLoopRun` (one RunAsync result), `AgentLoopGuards` (repeat / hard-stop / transient-retry defenses). The actor wraps one `IAgentLoop` per session — **`AgentBotActor` is the UI gateway, `AgentLoopActor` is the agent**. Full vocabulary table at `harness/knowledge/_shared/agent-architecture.md`.

Actor names sometimes contain user input (workspace names, terminal IDs). Route them through `ActorNameSanitizer` before constructing paths — Akka rejects characters like `/`, `:`, spaces.

### Akka shutdown quirk
`ActorSystemManager.Shutdown()` is fire-and-forget by design. Previously `ShutdownAsync().GetAwaiter().GetResult()` on the UI thread deadlocked the `synchronized-dispatcher` (which needs to post back to the UI thread), leaving the process alive and the single-instance mutex held. The Akka config sets `coordinated-shutdown.exit-clr = on` so CLR termination happens from the shutdown phases, not from user code.

### CLI ↔ GUI IPC

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [psmon/AgentZeroLite](https://github.com/psmon/AgentZeroLite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
