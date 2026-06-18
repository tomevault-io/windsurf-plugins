---
trigger: always_on
description: > This file provides context for AI coding assistants working on this project.
---

# MosAIc Companion — AI Agent Instructions

> This file provides context for AI coding assistants working on this project.
> Read this before making changes to the codebase.

## Project Overview

MosAIc is an Electron desktop app (React + TypeScript) that serves as an AI companion with tool use, wallet integration, and secure sandboxed tool execution.

**Stack:** Electron (main process) + React (renderer) + TypeScript. Vite for bundling.

## Architecture — MUST READ Before Modifying

The project follows a **Core (trusted) vs Sandbox (untrusted)** architecture for tool execution.

**Full architecture docs:** [`/docs/architecture/`](docs/architecture/README.md)

### Key Rules

1. **WASM is the primary runtime for sandboxed tools.** Docker is optional for heavy workloads (GPU, databases). WASM tools have zero network/filesystem/OS access by default — all capabilities come through host functions.
2. **The Gatekeeper is host functions, not a container.** For WASM tools, the Gatekeeper logic runs directly inside Electron as host functions gated by `GatekeeperPolicy`. No separate proxy or container.
3. **Tools are always low-trust.** Even tools written by us get the same restrictions in the Sandbox. Trust is architectural, not reputational.
4. **Every boundary crossing must be:** explicit, Core-mediated, and logged.
5. **No runtime permission escalation** — tools declare permissions in their manifest upfront.
6. **`GatekeeperPolicy` is the base interface** for filtering decisions. WASM host functions call it directly now; a future Docker proxy would call the same policy. Same rules, different plumbing.

### Key Architecture Docs

| Doc                                                                          | When to read                                              |
| ---------------------------------------------------------------------------- | --------------------------------------------------------- |
| [`overview.md`](docs/architecture/overview.md)                               | Before modifying any Core or Sandbox code                 |
| [`manifest.md`](docs/architecture/manifest.md)                               | Before changing tool manifests, permissions, or UI panels |
| [`gatekeeper.md`](docs/architecture/gatekeeper.md)                           | Before touching outbound network filtering                |
| [`container-communication.md`](docs/architecture/container-communication.md) | Before changing how MosAIc talks to tools                 |
| [`permissions.md`](docs/architecture/permissions.md)                         | Before changing the permission model                      |
| [`data-model.md`](docs/architecture/data-model.md)                           | Before modifying Chronicle, Vault, or data flow           |
| [`tool-lifecycle.md`](docs/architecture/tool-lifecycle.md)                   | Before changing tool install, launch, or execution flow   |
| [`tool-ui.md`](docs/architecture/tool-ui.md)                                 | Before changing how tools render UI blocks                |
| [`execution-plan.md`](docs/architecture/execution-plan.md)                   | For the Phase 1 implementation sequence                   |
| [`implementation-status.md`](docs/architecture/implementation-status.md)     | To check what's built vs planned                          |
| [`glossary.md`](docs/architecture/glossary.md)                               | For term definitions                                      |

## Project Structure

```
electron/
  main.ts                          # Electron main process entry point
  preload.ts                       # IPC bridge (renderer ↔ main)
  settings.ts                      # Persistent app settings (~/.config/mosaic-companion/)
  updater.ts                       # Electron autoUpdater configuration
  utils/
    index.ts                       # File/folder utilities, JSON helpers
  integrations/
    sandbox/                       # Tool sandbox subsystem (WASM-first)
      types.ts                     # ToolManifest, ToolLauncher, GatekeeperPolicy interfaces
      gatekeeper.ts                # ManifestGatekeeperPolicy + WASM host functions
      wasm-launcher.ts             # WasmLauncher (Extism) — ONLY file that knows WASM/Extism
      tool-bridge.ts               # Bridges WASM tools into ToolModule pattern
      chronicle.ts                 # Append-only JSONL activity logs per tool
      index.ts                     # ToolManager — orchestration + IPC
    tools/                         # Tool system
      types.ts                     # ToolModule, ToolDefinition, ExecutionContext
      registry.ts                  # ToolRegistry — central tool manager
      index.ts                     # Registry singleton + lifecycle
      modules/                     # Built-in tool modules
        gmail.ts                   # Gmail integration (8 tools)
        web3.ts                    # Web3/crypto tools (17 tools)
        vault-tools.ts             # Vault read-only access for agents
    vault/                         # Vault data storage (boxes + entries)
      index.ts                     # Vault CRUD (boxes, entries, content)
      types.ts                     # VaultBox, VaultEntry, BoxContent

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hypercycle-development/mosaic-companion](https://github.com/hypercycle-development/mosaic-companion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
