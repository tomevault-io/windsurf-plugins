---
trigger: always_on
description: **Target Project:** git-mcp-server
---

# Agent Protocol & Architectural Mandate

**Version:** 2.15.0
**Target Project:** git-mcp-server
**Last Updated:** 2026-04-28

This document defines the operational rules for contributing to this codebase. Follow it exactly.

> **Note on File Synchronization**: `AGENTS.md` is a symlink to `CLAUDE.md`. **Edit only `CLAUDE.md`** — `AGENTS.md` will reflect changes automatically.

---

## I. Core Principles (Non-Negotiable)

1. **The Logic Throws, The Handler Catches**
   - **Tools:** Implement pure, stateless business logic inside tool logic functions. **No `try/catch` blocks.**
   - **Resources:** Same rule — pure read logic, no `try/catch`.
   - **On Failure:** Throw `new McpError(...)` with the appropriate `JsonRpcErrorCode` and context.
   - **Framework's Job:**
     - `createMcpToolHandler` wraps tool logic: creates `RequestContext`, measures execution via `measureToolExecution`, formats the response, catches errors.
     - `createToolHandler` wraps git-specific logic: resolves DI dependencies and working directory before calling your pure logic.
     - Resource handlers (`resourceHandlerFactory`) validate params, invoke logic, apply `responseFormatter`, and catch errors.

2. **Full-Stack Observability**
   - OpenTelemetry is preconfigured. Logs and errors are automatically correlated to traces.
   - `measureToolExecution` automatically records duration, success, payload sizes, and error codes for every tool call.
   - Do not add custom spans in tool/resource logic. The framework handles instrumentation.

3. **Structured, Traceable Operations**
   - Tool logic receives dependencies via `ToolLogicDependencies` (which includes `appContext` and `sdkContext`).
   - `appContext` (`RequestContext`): Internal logging/tracing context with `requestId`, `sessionId`, `tenantId`, `traceId`.
   - `sdkContext` (`SdkContext`): MCP SDK protocol capabilities — `signal`, `sendNotification`, `sendRequest`, `authInfo`.
   - Pass `appContext` through your internal call stack. Use the global `logger` with `appContext` in every log call.

4. **Decoupled Storage**
   - Never directly access persistence backends from tool/resource logic.
   - Use `StorageService` (injected via DI) for session state (working directory persistence).
   - Git operations execute via the `IGitProvider` interface, not direct CLI calls.

5. **Graceful Degradation in Development**
   - When `tenantId` is missing, default to permissive behavior: `const tenantId = appContext.tenantId || 'default-tenant';`
   - Auth/scope checks default to allowed when auth is disabled.
   - Production environments with auth enabled provide real `tenantId` from JWT claims automatically.

---

## II. Directory Structure

| Directory                               | Purpose                                                                                                                                  |
| :-------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------- |
| `src/mcp-server/tools/definitions/`     | MCP Tool definitions. Named `git-[operation].tool.ts`.                                                                                   |
| `src/mcp-server/tools/utils/`           | Shared tool utilities: `toolDefinition.ts`, `toolHandlerFactory.ts`, `git-validators.ts`, `json-response-formatter.ts`.                  |
| `src/mcp-server/tools/schemas/`         | Shared Zod schemas: `PathSchema`, `CommitRefSchema`, `BranchNameSchema`, etc.                                                            |
| `src/mcp-server/resources/definitions/` | MCP Resource definitions. Primary: `git-working-directory.resource.ts`.                                                                  |
| `src/mcp-server/resources/utils/`       | Shared resource utilities: `ResourceDefinition` and handler factory.                                                                     |
| `src/mcp-server/prompts/definitions/`   | MCP Prompt definitions (e.g., `git-wrapup.prompt.ts`).                                                                                   |
| `src/mcp-server/transports/`            | Transport implementations: `http/` (Hono + Streamable HTTP), `stdio/`, `auth/` (JWT/OAuth strategies).                                   |
| `src/services/git/`                     | Git service: `core/` (interfaces, factory), `providers/cli/` (CLI implementation with domain-organized operations).                      |
| `src/storage/`                          | Storage abstractions and providers (in-memory, filesystem, supabase, cloudflare).                                                        |
| `src/container/`                        | Dependency injection (`tsyringe`). Service registration and tokens.                                                                      |
| `src/utils/`                            | Global utilities: `internal/` (logger, requestContext, ErrorHandler, performance), `security/` (sanitization), `telemetry/`, `metrics/`. |
| `tests/`                                | Unit/integration tests mirroring `src/` structure.                                                                                       |

---

## III. Tool Development Workflow


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cyanheads/git-mcp-server](https://github.com/cyanheads/git-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
