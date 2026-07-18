---
trigger: always_on
description: **Target Project:** mcp-ts-template
---

# Agent Protocol & Architectural Mandate

**Version:** 2.4.7
**Target Project:** mcp-ts-template
**Last Updated:** 2024-10-15

This document defines the operational rules for contributing to this codebase. Follow it exactly.

> **Note on File Synchronization**: `AGENTS.md` is symlinked to CLAUDE.md & `.clinerules/AGENTS.md` for consistency. Only edit the root `AGENTS.md` file. You do not have permission edit, touch, or change in any way the `CLAUDE.md` or `.clinerules/AGENTS.md` files.

> **Note for Developer**: Never assume anything. Always review related files, search for documentation, etc. when making changes. Always prefer reading the full file content to understand the full context. NEVER attempt to edit a file before reading the current content.

---

## I. Core Principles (Non‑Negotiable)

1.  **The Logic Throws, The Handler Catches**
    - Implement pure, stateless logic in `ToolDefinition`/`ResourceDefinition` `logic` functions. No `try...catch` in logic.
    - Throw `new McpError(...)` with appropriate `JsonRpcErrorCode` on failure.
    - Handlers (`createMcpToolHandler`, `resourceHandlerFactory`) create `RequestContext`, measure execution, format responses, and catch errors.

2.  **Full‑Stack Observability**
    - OpenTelemetry preconfigured. Logs/errors auto-correlated to traces. `measureToolExecution` records duration, success, payload sizes, error codes.
    - No manual instrumentation. Use provided utilities and structured logging. No direct console calls - use our logger.

3.  **Structured, Traceable Operations**
    - Logic receives `appContext` (logging/tracing) and `sdkContext` (Elicitation, Sampling, Roots operations).
    - Pass same `appContext` through call stack. Use global `logger` with `appContext` in every log.

4.  **Decoupled Storage**
    - Never access persistence backends directly. Always use DI-injected `StorageService`.
    - `StorageService` provides built-in validation, opaque cursor pagination, and parallel batch operations.
    - All inputs (tenant IDs, keys, prefixes) are validated before reaching providers.

5.  **Local ↔ Edge Runtime Parity**
    - All features work with local transports (`stdio`/`http`) and Worker bundle (`build:worker` + `wrangler`).
    - Guard non-portable deps. Prefer runtime-agnostic abstractions (Hono + `@hono/mcp`, Fetch APIs).

6.  **Use Elicitation for Missing Input**
    - Use `sdkContext.elicitInput()` for missing params. See `template_madlibs_elicitation.tool.ts`.

---

## II. Architectural Overview & Directory Structure

> **📁 Repository Structure Reference**: For a complete visual tree of the codebase, see [docs/tree.md](docs/tree.md). This will help you understand the full directory layout and where to place your code.
>
> **⚠️ Architectural Discipline**: ALWAYS respect the established directory structure. New services go in `src/services/`, new tools in `src/mcp-server/tools/definitions/`, etc. Do not create top-level directories or place code in non-standard locations.

Separation of concerns maps directly to the filesystem. Always place files in their designated locations.

| Directory                                   | Purpose & Guidance                                                                                                                                                                                                                                                                                                                |
| :------------------------------------------ | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`src/mcp-server/tools/definitions/`**     | **MCP Tool definitions.** Add new capabilities here as `[tool-name].tool.ts`. Follow the **Tool Development Workflow**.                                                                                                                                                                                                           |
| **`src/mcp-server/resources/definitions/`** | **MCP Resource definitions.** Add data sources or contexts as `[resource-name].resource.ts`. Follow the **Resource Development Workflow**.                                                                                                                                                                                        |
| **`src/mcp-server/tools/utils/`**           | **Shared tool utilities:** Core tool infrastructure (`ToolDefinition`, `toolHandlerFactory`)                                                                                                                                                                                                                                      |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [polvallverdu/base-ui-mcp](https://github.com/polvallverdu/base-ui-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
