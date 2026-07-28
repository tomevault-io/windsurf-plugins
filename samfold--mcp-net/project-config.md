---
trigger: always_on
description: - Revisit `IMcpClient` ergonomics around tool-call cancellation and async disposal.
---

# Roadmap: Mcp.Net.Agent

## Current focus

- Revisit `IMcpClient` ergonomics around tool-call cancellation and async disposal.
- Keep the local filesystem and process surfaces coherent now that `ReadFileTool` metadata, `WriteFileTool`, `GrepTool`, `GlobTool`, `EditFileTool`, `run_shell_command`, and the bounded-vs-unbounded filesystem policy are in place.
- Preserve the now-completed continue/resume, per-turn summary, guarded event-dispatch, async compaction, and transcript lifecycle-notification surfaces while tool coverage expands.

## What

- Revisit the `IMcpClient` seam so callers can eventually get cleaner cancellation and ownership/disposal behavior for remote tool execution.
- Keep `run_shell_command` as the bounded local process tool for host CLI workflows, with host-shell resolution, root-bounded working directories, timeout caps, output truncation, and concurrency limits.
- Later, replace the entry-count-only compaction trigger with token-aware context budgeting that can target provider max-context limits and reserve output budget explicitly.

## Why

- The runtime and factory seams are now in place and the dead model layer is gone.
- Continue/resume, per-turn summaries, transcript lifecycle notifications, and the dead session-start seam are now in place.
- The next highest-value gap is not another runtime seam; it is proving the library with concrete tools that real consumers can use.
- The first built-in tools should establish a public authoring pattern for outside consumers instead of relying on internal-only helpers.
- `Mcp.Net.WebUi` is an older adapter layer and should not drive `Mcp.Net.Agent` design; the runtime can move first and Web UI can be rebuilt around it if necessary.
- Bounded filesystem discovery, content search, and surgical mutation tools are now in place.
- `ReadFileTool` now exposes mutation-oriented metadata, `GlobTool` enables deterministic candidate discovery, `GrepTool` enables bounded content search, and `EditFileTool` enables bounded edits to existing text files.
- The first bounded process seam is now in place for real CLI workflows (`git`, `dotnet`, `npm`, `cargo`).
- The local file/process tool stack now covers discovery, content search, reads, writes, edits, and bounded shell execution, so the next runtime gap is the remote-client seam.
- The current entry-count compactor is a good MVP, but it does not track real context-window pressure or leave deliberate room for model output.

## How

### Next runtime slice

- Revisit `IMcpClient` so tool-call cancellation and ownership/disposal behavior are explicit instead of leaking through host-specific composition.
- Keep the landed `ReadFileTool` / `WriteFileTool` / `EditFileTool` / `GrepTool` / `GlobTool` / `ListFilesTool` / `RunShellCommandTool` surface stable while validating that next runtime seam.
- Preserve the snapshot-based provider boundary and avoid inventing provider-specific escape hatches while the local-tool surface grows.

### Verification

- Add focused tool coverage for containment checks, truncation, typed argument binding, and error paths.
- Add executor/session coverage as needed to prove the tools flow through the current runtime seams.
- Keep the completed `ChatSession` lifecycle tests and broader agent/runtime coverage green.

## Near-term sequence

1. Revisit `IMcpClient` ergonomics when a real caller needs `CallTool` cancellation or async disposal.
2. Revisit session-owned transcript persistence when non-Web UI consumers need durable session state.
3. Consider hook/extension and conversation-branching surfaces only after the core loop is robust.
4. Revisit context-window management with token-aware compaction driven by provider context limits, reserved output budget, and a stronger summarizer path once real conversation pressure justifies it.

## Recently completed

- Removed the obsolete `AgentDefinition` / manager / store / registry model and agent-oriented DI/extensions from `Mcp.Net.Agent`.
- Removed the corresponding agent-driven controllers, DTOs, startup hooks, and chat-factory branches from `Mcp.Net.WebUi`.
- Narrowed the remaining registration story to `AddChatRuntimeServices()` plus `AddChatSessionFactory()`.
- `ChatSession` now flows caller cancellation through provider requests and tool execution.
- Abort behavior is now deterministic for provider waits and tool execution, including partial tool-result persistence when some tool work finished before cancellation.
- `ChatSession` now validates tool execution against its own configured tool catalog and no longer depends on `IToolRegistry` at runtime.
- `Mcp.Net.Agent.Tools` now includes `ILocalTool`, `LocalToolExecutor`, and `CompositeToolExecutor`.
- Local tools can now create results through public `ToolInvocation` / `ToolInvocationResults` helpers instead of relying on the raw `ToolInvocationResult` constructor.
- Local tools can now bind invocation arguments through `ToolInvocation.BindArguments<TArgs>()` or derive from `LocalToolBase<TArgs>` for typed authoring plus generated input schema from a transport-neutral local-tool generator.
- `AddChatRuntimeServices()` no longer registers the disconnected tool-registry seam; `ToolRegistry` is now explicit opt-in through `AddToolRegistry()`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SamFold/Mcp.Net](https://github.com/SamFold/Mcp.Net) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
