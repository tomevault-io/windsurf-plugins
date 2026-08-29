---
trigger: always_on
description: **ALWAYS read when working on code:**
---

# Project Instructions — PowerPointMcp

## Critical Files (Read These First)

**ALWAYS read when working on code:**
- [Critical Rules](instructions/critical-rules.instructions.md) - mandatory rules (Success flag, COM cleanup, TDD, session lifecycle)
- [Architecture Patterns](instructions/architecture-patterns.instructions.md) - layered architecture, command pattern, resource management

**Read based on task type:**
- Adding/fixing Core commands or COM interop → [Architecture Patterns](instructions/architecture-patterns.instructions.md)
- Writing tests → [Testing Strategy](instructions/testing-strategy.instructions.md)
- MCP Server tool work → [MCP Server Guide](instructions/mcp-server-guide.instructions.md)

## Sister Projects

PowerPointMcp is one of a family of Windows-only, COM-interop MCP-server repos maintained by the
same author (`sbroenne`). The family currently includes:

- **`mcp-server-excel`** (`sbroenne/mcp-server-excel`) — **the authoritative architectural
  template for this entire family.** Its layering (ComInterop → Core → Service → CLI/MCP Server),
  Unified Service Architecture, generator design, hand-written vs. generated tool split, COM
  cleanup discipline (Rule 22: every `dynamic` COM object released via `ComUtilities.Release` in a
  `finally` block), support/audit scripts (`scripts/*.ps1`), and CI Gate workflow are the reference
  design. When PowerPointMcp diverges from Excel, treat the divergence as a bug to fix **unless**
  there is a concrete, documented reason the underlying Office object model differs (e.g. Excel's
  `Application.ScreenUpdating` has no PowerPoint equivalent — that divergence is legitimate and
  intentionally NOT ported).
- **`mcp-server-powerpoint`** (this repo) — PowerPoint automation, structurally mirrors Excel's
  10-project layout and Unified Service Architecture.
- **`mcp-windows`** — a sister Windows-automation MCP server project.

**When making architectural changes here, check whether Excel already solved the same problem —
port its pattern rather than inventing a new one.** Likewise, if you fix a real bug or close a gap
here that also exists in Excel or `mcp-windows`, flag it so the same fix can be ported there.

## What is PowerPointMcp?

**PowerPointMcp** is a Windows-only toolset for programmatic PowerPoint automation via COM
interop, designed for coding agents and automation scripts. It drives a **live PowerPoint desktop
instance** through the official `Microsoft.Office.Interop.PowerPoint` PIA (embedded via
`ForceEmbedPowerPointInteropTypes` — no runtime assembly-resolver needed, unlike Excel).

The project ships **two equal, first-class entry points** — an MCP Server and a CLI — that share
one `Core` codebase and are kept in parity by source generators, mirroring `mcp-server-excel`'s
Unified Service Architecture.

**Core Layers:**
1. **ComInterop** (`src/PowerPointMcp.ComInterop`) - STA thread + OLE message filter + channel-based
   work queue (`PresentationBatch`), ported from `mcp-server-excel`'s `ExcelBatch` pattern.
2. **Core** (`src/PowerPointMcp.Core`) - PowerPoint domain commands, one folder per domain
   (Presentation, Slide, Shape, TextFrame, Table, Notes, Layout, Master, Animation, Image, Media, Chart,
   Export). Domains other than Presentation carry a `[ServiceCategory]` marker attribute that the
   generators discover.
3. **Generators** (`src/PowerPointMcp.Generators.Mcp`, `src/PowerPointMcp.Generators.Cli`,
   `src/PowerPointMcp.Generators.Shared`) - Roslyn source generators that read `[ServiceCategory]`
   Core interfaces and emit one **action-dispatch tool per domain** for the MCP surface (e.g.
   `slide`, `shape`, `chart`, each taking an `action` parameter like `"add-chart"`) and one
   `pptcli {category} {action}` command per operation for the CLI. `Presentation` is also a
   hand-written action-dispatch tool (`PresentationTools.cs`) so session lifecycle, template work,
   and document properties live under the single `presentation` MCP tool.
4. **Service** (`src/PowerPointMcp.Service`) - `PowerPointMcpService`: the shared session registry
   + dispatch layer both entry points call into. **McpServer** hosts it **in-process** (no pipe,
   via `ServiceBridge.ForwardToService`); **CLI** (`pptcli`) talks to it via a **separate
   background daemon process** over a named pipe (`ServiceClient`/`IPowerPointDaemonRpc`,
   auto-started on first `session open`/`session create`), so sessions persist across CLI
   invocations without paying PowerPoint's ~90-150s launch cost every command.
5. **McpServer** (`src/PowerPointMcp.McpServer`) - Model Context Protocol stdio host. 16 tools
   total: one hand-written `presentation` action-dispatch tool plus 15 generated action-dispatch
   tools (`slide`, `shape`, `textframe`, `table`, `notes`, `layout`, `master`, `animation`,
   `image`, `media`, `chart`, `smartart`, `export`, `pagesetup`, `accessibility`), covering 186 operations
   across 16 domains. See
   `tests/PowerPointMcp.McpServer.Tests/Integration/McpProtocolTests.cs`'s `ExpectedToolNames` for
   the ground-truth tool list.
6. **CLI** (`src/PowerPointMcp.CLI`) - `pptcli`, built on the same generators as the MCP surface
   (`CliCommandRegistration.RegisterCommands`) plus hand-written `session`/`service` command

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sbroenne/mcp-server-powerpoint](https://github.com/sbroenne/mcp-server-powerpoint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
