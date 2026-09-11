---
trigger: always_on
description: MCP (Model Context Protocol) tooling for **vvvv gamma** (visual programming on .NET, .vl files).
---

# AGENTS.md — vvvv-mcp

MCP (Model Context Protocol) tooling for **vvvv gamma** (visual programming on .NET, .vl files).
Goal: a professional AI patching assistant that builds whole connected subgraphs in one call,
verified against the live vvvv instance.

> Terminology: "vvvv gamma" = current product. "VL" = its visual language. "vvvv beta" = the
> old product — never apply its concepts.

## Architecture (two processes)

1. **`src/VvvvMcp` + `src/VvvvMcp.Core`** — the MCP server (stdio, .NET 8 global tool).
   Used by MCP clients (Kilo/VS Code, Claude, Cursor, Open WebUI). Reads/writes .vl files
   directly (XML), talks to the bridge over HTTP when vvvv runs.
2. **`VL.MCP.HDE`** — vvvv editor extension (HDE package) hosting the bridge:
   `HttpListener` REST on `:7123` + MCP-over-SSE on `/sse` + chat host (Open WebUI, `:7125`).
   Bridge starts automatically with the extension (always on, `:7123`); **Alt+C** toggles chat.

Key services (VvvvMcp.Core/Services):
- `PatchBuilderService` — `build_patch`: one-call subgraph builder (resolve → deps → pins →
  layout → links → save → reload → verify). THE primary write path.
- `SpecValidator` — extensible pre-validation rules that run BEFORE build_patch writes anything.
  Catches common mistakes (missing RootScene, type mismatches) and returns
  actionable fix suggestions. Add new `ISpecValidationRule` implementations as patterns are found.
- `NodeResolutionService` — live registry first (bridge `/api/nodes`), offline catalog fallback.
- `PatchWriterService` / `PatchReaderService` — .vl XML read/write.
- `SearchIndexService` — SQLite FTS5 (two-phase: AND then OR fallback). Schema version via
  `PRAGMA user_version` — bump `SchemaVersion` when FTS tables change.
- `BridgeClientService` — HTTP client for the bridge. Env override `VVVV_MCP_BRIDGE_PORT`.

Bridge (VL.MCP.HDE/src):
- `MCPBridgeServer.cs` — ProcessNode, REST routes, `IDisposable` (MUST release the listener
  on dispose or the port stays hostage across recompiles).
- `McpSseServer.cs` — MCP JSON-RPC 2.0 server (Streamable HTTP POST `/mcp` + legacy SSE
  `/mcp/sse`). Sends `BridgeInstructions.Text` in the `initialize` response — this is the
  Tier-0 knowledge that bridge-connected LLMs (Open WebUI/kimi) need to produce working patches.
  Keep `BridgeInstructions.cs` in sync with `ServerInstructions.cs` (the stdio equivalent).
- `LiveNodeCatalog.cs` — live node snapshot from `NodeFactoryRegistry.Factories` (.NET nodes)
  + `LatestCompilation.DocumentsAndPackages → DefinedSymbols` (VL-defined nodes). Reflection
  over VL.Lang (no compile-time ref). Auto-rebuilds when the factory set changes.
- `BridgeState.cs` — documents/errors/packages via reflection. Errors carry
  `DocumentId`/`ElementId` (== .vl XML Id attributes) from `VL.Lang.Message.Location`.
- `McpChatHost.cs` — Open WebUI lifecycle: **named mutex `Global\vvvv-mcp-chat-start`**
  serializes startups across Alt+C presses AND HDE reloads; adopt healthy running instance
  (MCP-registration failure is non-fatal); never cancel/kill on disable (the "Open Chat"
  pin is a one-frame bang); server dies only on Dispose (vvvv exit). `/chat` 302-redirects
  to OWUI when up, else serves the placeholder page (polls same-origin `/api/chat/status`,
  reloads on ready — never navigate cross-origin from CEF client-side). Chat host sets
  `WEBUI_AUTH=False`/`ENABLE_SIGNUP=False` (else OWUI shows an admin-setup prompt).
  Loading `/chat` (a chat window opening) sets `_chatWanted` → auto-starts OWUI even
  without Alt+C (handles vvvv-restart-with-chat-open). OWUI console output: only error-ish
  lines forward to the vvvv console; all lines buffered → `GET /api/chat/log`.

## Ground-truth rules learned the hard way

- **`build_patch` auto-layout** (`ComputeLayout` in `PatchBuilderService.cs`): bottom-up Sugiyama-
  style layout. Sources (depth=0) at top, sinks (depth=max) at bottom. Y rows spaced by nodeHeight
  + 20px gap (≈39px top-to-top for standard nodes). X positions determined by processing deepest
  layers first; each node's preferred center X = average of `(successor.left + 5 + 20*pinIndex)`
  over placed successors. Nodes within a layer sorted by preferred X then packed left-to-right
  without overlap. StartX=43, StartY=40. ColGap=20. Pads get type-specific 4-value bounds
  (Float32/Int=35×15, Vector3=85×15, Vector2=65×15, RGBA=65×15, Boolean=35×35, String=150×20).
- **Live catalog symbol sources** (`ExtractFromCompilation`): iterates `DocumentsAndPackages` which
  has two fundamentally different source types:
  - `DocSymbols` (open documents) → VL source, always current, contains `PatchedXxx` symbols
  - `CompiledSymbols` (binary packages like VL.CoreLib.dll) → Roslyn analysis of the binary,
    may contain stale symbols removed from the .vl source but still in old binary metadata
  - `CompiledSymbols.DefinedSymbols` groups contain `ImportedConcreteTypeSymbol` items. Each
    such type has `OriginalSymbolSource.IsForeign`:
    - **`IsForeign = false`**: VL-compiled type (from VL.CoreLib.vl source) → operations are valid
      patchable nodes (e.g. `Split (BezierSegment)` on BezierSegment record)
    - **`IsForeign = true`**: C# type from a foreign assembly (e.g. Matrix from Stride.Core.Mathematics)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [digitalwannabe/vvvv-mcp](https://github.com/digitalwannabe/vvvv-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-11 -->
