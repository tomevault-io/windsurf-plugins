---
trigger: always_on
description: This document provides instructions for AI agents working on this codebase.
---

# AI Agent Instructions for MCP Starter Template

This document provides instructions for AI agents working on this codebase.

## Project Vision
A minimal, highly structured, and modular MCP server template in TypeScript.

**Dual Role**:
1. **Editable Boilerplate**: Fork or clone this to scaffold new MCP servers rapidly.
2. **Reference Library**: Keep this template alongside your other MCP servers. It serves as a 2026 "Gold Standard" that agents can read anytime they need to build, edit, or improve an MCP server's tool orchestration and metadata.

## Mandatory Reading

> Before creating or modifying **any** tool, you **MUST** read and internalize [Tool Description Prompt Engineering](./docs/descriptions_prompt_engineering.md). It contains the canonical examples, patterns, and rules for 2026-grade tool metadata.

## Architecture Guidelines (Agent-First & Tools-First)
In 2026, **Autonomous AI Agents** are the primary consumers of MCP. Follow these principles:

- **Tools-First Strategy**: Focus on creating simple, efficient, and robust **Tools**. They are the most reliable primitive for agentic workflows.
- **Discouraged Primitives**: In 99% of cases, **Resources**, **Prompts**, and **Sampling** should be considered legacy or specialized edge cases.
- **Long-Running Processes**: Use tools with **Progress Logging** (`server.server.sendLoggingMessage`) for tasks that take time.
- **Transport & Registration**: Exclusively `stdio`. Use `registerTool` config-based registration for strict typing and runtime flexibility.
- **Sampling Fallbacks**: If sampling is absolutely necessary, **mandate** a manual tool fallback. Never assume client support.

## SDK v1.27.1 Native Features — Use Them, Don't Reinvent Them
The SDK provides first-class support for metadata and flow control that agents use to reason about tools. **Always** use these over custom approximations:

| Feature | Where | Purpose |
|---|---|---|
| `title` | `registerTool`, `registerPrompt`, `registerResource` | Human-readable display name. Distinct from the programmatic `name`. |
| `annotations` | `registerTool` config | Behavioral hints: `readOnlyHint`, `destructiveHint`, `idempotentHint`, `openWorldHint`. Agents use these to assess risk and cacheability. |
| `outputSchema` | `registerTool` config | Zod schema for structured output validation. Use when returning typed data. |
| `ctx.signal` | Tool Handler Context | `AbortSignal` for standard cancellation. Inherit into fetchers or loops (`signal: ctx.signal`). |
| `server.server.sendLoggingMessage` | Tool Logic | Emit structured logs back to the client during long operations. |
| `.enable()` / `.disable()` | `RegisteredTool`, `RegisteredResource`, `RegisteredPrompt` | Toggle visibility at runtime. Call `server.server.sendToolListChanged()` after. |
| `.update()` | All registered objects | Modify name, description, schema, or callback after registration. |
| `.remove()` | All registered objects | Permanently unregister the primitive. |

**Every tool MUST have**: `title`, `description`, `annotations`, and fully `.describe()`'d `inputSchema`.

## Dynamic Tool Registration
**Dynamic Registration** is the single most efficient way to leverage MCP servers in 2026:

- **Context Rot Prevention**: Enable/disable tools based on session state to prevent irrelevant tools from polluting the agent's context.
- **Precision Orchestration**: Push the "right tool at the right time" (e.g., enable analysis tools only after data retrieval).
- **Implementation**: Use `RegisteredTool.enable()` / `.disable()` at runtime. Always call `server.sendToolListChanged()` to notify the client.

## Graduated Complexity: Tool Orchestration Levels
Always aim for the lowest complexity that achieves the objective. See `src/tools/examples.ts` for blueprints:

1. **Level 1: Atomic** — Single-purpose tools like `ping_host`. High reliability, no state.
2. **Level 2: Sequential** — Tools like `extract_keywords` designed to pipe into `search_knowledge_base`. Use descriptions to guide the dependency chain.
3. **Level 3: State-Aware** — Tools like `toggle_workspace_lock` that manage the agent's operating environment.

## Quick Start for New Servers

1. **Prune Primitives**: If Resources, Prompts, or Sampling aren't needed, **delete them immediately**.
   - Remove: `src/resources/`, `src/prompts/`.
   - Cleanup `src/server.ts` imports and registration calls.
2. **Focus on Tools**: Add your logic to `src/tools/`.
3. **Minimalist Architecture**: Prefer a small set of high-impact, versatile tools over a large library of niche ones.
4. **Agentic Trust**: Trust that modern agents are smart enough to choose the right tool from high-quality metadata. Design "Golden Path" tool sequences (Discovery → Analysis → Action).

## Crafting Tool Descriptions

Autonomous agents rely entirely on descriptions to know **when** and **how** to use a tool. The main description and Zod argument descriptions form a **unified interface** — they must be complementary, never redundant.

**Main Description** (the `description` field):
- Start with a clear intent verb (e.g., "Analyze", "Retrieve", "Toggle").
- Include "When to use" heuristics and "Returns" expectations.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Git-Fg) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
