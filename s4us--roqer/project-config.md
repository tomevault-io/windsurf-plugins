---
trigger: always_on
description: ﻿# Roblox MCP Engineering Guide
---

﻿# Roblox MCP Engineering Guide

## Purpose

This open-source monorepo contains two connected products:

1. the Roblox Studio MCP bridge and Studio plugin, including the full and
   inspector editions; and
2. Roqer, a local desktop agent that consumes the MCP.

Roqer is a harness with no account and no hosted service: runs are driven by
the user's own ChatGPT or Claude subscription through the official Codex or
Claude Code client on their machine, or by a model endpoint the user configures
with their own key (the `custom` provider, driven by Roqer's own agent loop in
the desktop). Nothing Roqer does depends on a server the project runs.

Build them as one system. The MCP owns Roblox and Studio capabilities. Roqer
owns the user-facing agent experience, approvals, persistence, and presentation.
Do not duplicate Studio behavior in the desktop app or desktop policy in the
MCP.

Prioritize, in order:

1. correctness and user safety;
2. truthful, verifiable behavior;
3. clear ownership and maintainable interfaces;
4. backward compatibility where practical;
5. agent usability and token efficiency;
6. implementation convenience.

## Sources of truth

Use the repository as it exists now, not a roadmap copied into this file.

- The user's request defines the current task and priority.
- Tests and running code define current behavior.
- `package.json` files define supported commands.
- `README.md`, `docs/`, and `tests/README.md` describe public behavior and test
  operation.
- Design studies are context, not implementation authority, unless the task
  explicitly adopts them.

Do not add a changing milestone checklist to this file. Update the relevant plan
or documentation when product status changes.

## Repository map

### MCP and server

- `packages/core/` contains shared MCP definitions, routing, transport, Studio
  instance management, and most server-side behavior.
- `packages/core/src/tools/definitions.ts` is the public tool schema catalog.
- `packages/core/src/tools/index.ts` contains the main tool implementation layer.
- `packages/core/src/http-server.ts` exposes the authenticated HTTP bridge used
  by Studio and Roqer.
- `packages/robloxstudio-mcp/` is the full-edition package entry point.
- `packages/robloxstudio-mcp-inspector/` is the inspector-edition entry point.

Keep package entry points thin. Shared behavior belongs in `packages/core` unless
there is a real edition-specific reason to separate it.

### Studio plugin

- `studio-plugin/src/` contains the Roblox-TS plugin source.
- `studio-plugin/src/modules/handlers/` owns Studio-privileged operations.
- `studio-plugin/MCPPlugin.rbxmx` and `MCPInspectorPlugin.rbxmx` are generated
  artifacts. Do not edit or commit them.

### Desktop Roqer

- `apps/desktop/electron/` owns Electron startup, preload, native integration,
  persistence wiring, build, and smoke scripts.
- `apps/desktop/runtime/` owns main-process logic that should remain testable
  without importing Electron: MCP access, run execution, planning, and result
  compaction. `agent-loop.ts` is Roqer's own model loop, used by the Custom
  provider; `model-api/` holds its turn contract and the OpenAI-compatible,
  OpenAI Responses and Anthropic transports.
- `apps/desktop/shared/` contains contracts shared by the main process and
  renderer: run events, policy, tool risk, and Studio status.
- `apps/desktop/src/` owns the React UI and renderer-side state.

### Tests and documentation

- `packages/core/src/__tests__/` contains core unit and integration-style tests.
- `apps/desktop/**/*.test.ts` contains desktop unit tests.
- `tests/` contains subprocess and live Studio tests. Read `tests/README.md`
  before running a live or destructive suite.
- `docs/` documents public setup and behavior.

## Working method

1. Inspect the relevant code, tests, and current plan before choosing a design.
2. Define acceptance criteria for behavior, safety, compatibility, and evidence.
3. Make one focused, PR-sized change. Avoid unrelated cleanup.
4. Extend existing ownership boundaries instead of creating parallel systems.
5. Add or update the narrowest useful test alongside the implementation.
6. Run targeted checks first, then the applicable completion gates below.
7. Inspect the actual diff before reporting completion.

Preserve user changes in a dirty worktree. Do not discard or overwrite unrelated
work. Never weaken a test or silently change public behavior just to make a gate
pass.

For genuinely separable work, use bounded subagents with explicit objectives,
file ownership, and acceptance criteria. Keep writes to shared registries and
interfaces under one owner. Treat subagent reports as leads; the integrating
agent must inspect the diff and verify the result.

## Architecture boundaries

### MCP public surface

The MCP wire surface is a public, budgeted API. Do not add a tool merely because
it is convenient internally. First consider whether the capability belongs in:

- an existing tool or operation;
- an MCP resource;
- server-side orchestration;
- a Studio plugin implementation detail; or
- Roqer orchestration over existing tools.

When a public tool changes, keep these layers synchronized:

1. schema and description;
2. MCP/public handler;
3. `RobloxStudioTools` implementation;
4. HTTP routing where applicable;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [S4US/Roqer](https://github.com/S4US/Roqer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->
