---
trigger: always_on
description: This is the **C# server** for the easy-dotnet Neovim plugin. It is the brain of the system — all business logic, project analysis, build orchestration, and IDE intelligence lives here. It communicates with the Neovim Lua client (`easy-dotnet.nvim`) over **JSON-RPC via stdin/stdout**, spawned as a child process.
---

# easy-dotnet-server — Copilot Instructions

## What This Repo Is

This is the **C# server** for the easy-dotnet Neovim plugin. It is the brain of the system — all business logic, project analysis, build orchestration, and IDE intelligence lives here. It communicates with the Neovim Lua client (`easy-dotnet.nvim`) over **JSON-RPC via stdin/stdout**, spawned as a child process.

> **If you are about to make changes that affect the JSON-RPC wire contract** (adding, removing, or changing any method name, parameter shape, or return type), you must read the client repo's instructions first:
> `$HOME/repo/easy-dotnet.nvim/.github/copilot-instructions.md`
> Both sides must be updated atomically. The protocol is the contract.

### Reference Repositories (read-only)

| Path | Purpose |
|---|---|
| `$HOME/repo/sdk` | .NET SDK source — reference for SDK/MSBuild behavior |
| `$HOME/repo/roslyn` | Roslyn source — reference for compiler/analysis APIs |
| `$HOME/repo/msbuild` | MsBuild source — reference for MSBuild APIs |
| `$HOME/repo/netcoredbg` | Debugger source — reference for debug adapter behavior |

Always consult these when implementing features that touch build, analysis, or debugging. **Behavior must match or closely approximate Visual Studio and Rider.** MSBuild is a complex beast — when in doubt, check what the SDK does.

When working with Microsoft-specific APIs and tooling (MSBuild, MTP, Roslyn, NuGet, StreamJsonRpc, etc.), **searching the internet is encouraged**. Official documentation, GitHub issues on the relevant repos, and source-browser links often contain critical behavioral details that aren't obvious from the API surface alone.

---

## Architecture Principles

### The Server Owns Complexity

> **Lua is hard to maintain. C# is not.** Every non-trivial decision, computation, and state belongs here. The client renders and collects; the server thinks.

The client has no knowledge of MSBuild, project files, solution structure, NuGet, Roslyn, or .NET tooling. The server decides what to show, when to show it, and what data to ask for.

### Feature Slices

Features are organized as **vertical slices**, not DDD layers. `EasyDotnet.IDE/TestRunner/` is the canonical example of a well-designed feature — read it before starting any new feature. There are older remnants of DDD structure in the codebase; do not follow those patterns for new work.

### Reverse Requests and User Feedback

The primary pattern for complex workflows is **server-initiated requests**. The server doesn't just respond to client calls — it also sends requests to the client mid-operation to collect input, present choices, or report progress.

This is an **editor**. User feedback during long-running operations is critical. Always wrap long-running operations in a `ProgressScope` — see `./EasyDotnet.IDE/Editor/ProgressScope.cs` for the pattern. Never leave the user staring at a silent editor.

**Canonical references — read these before implementing any new feature:**
- `EasyDotnet.IDE/TestRunner/` — complete end-to-end example of a feature slice with reverse requests and progress reporting
- `EditorService.cs` — service layer structure and how the server orchestrates client interaction

The typical flow:

```
Client calls server  →  server opens a ProgressScope, starts async work
  →  server sends reverse request to client ("pick a framework")
  →  client responds
  →  server continues, sends progress notifications via ProgressScope
  →  server sends final result or structured error, closes ProgressScope
```

### JSON-RPC Contract

- **JSON-RPC 2.0**: `method` + `params` + `id` for requests; `result`/`error` for responses; omit `id` for notifications.
- Method names follow a **REST-style path convention**: `testrunner/run`, `project/restore`, `editor/navigate`.
- Use **notifications** for progress and telemetry. Use **requests** (with `id`) only when the client must reply.
- `rpcDoc.md` is the hand-maintained wire contract. **Never modify it.** Read it to understand existing signatures.

---

## Project Structure

### `EasyDotnet.IDE`

The main server process. Hosts the StreamJsonRpc endpoint, all feature slices, and owns the lifecycle of the BuildServer. Run with:

```bash
dotnet run --project EasyDotnet.IDE
```

### `EasyDotnet.BuildServer`

A **standalone executable** that the IDE project owns through `BuildHost` / `BuildHostFactory`. All MSBuild work happens exclusively in this process — never in `EasyDotnet.IDE` directly. It is generally fine to make changes here, but keep it strictly MSBuild-centric. If something isn't about evaluating or building projects via MSBuild, it does not belong in `BuildServer`.


### `EasyDotnet.StartupHook`
A .NET startup hook injected into user processes at launch. It emits the process PID and conditionally pauses execution to wait for a debugger to attach — pausing only when launched in debug mode, not for regular runs. Both run and debug go through this hook; the pause behaviour is the only difference.
Keep this project minimal and conservative:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GustavEikaas/easy-dotnet-server](https://github.com/GustavEikaas/easy-dotnet-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
