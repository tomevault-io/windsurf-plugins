---
trigger: always_on
description: [中文](AGENTS-CN.md) | **English**
---

[中文](AGENTS-CN.md) | **English**

# AGENTS.md

OpenBitFun is a Rust workspace plus React frontends.

Repository rule: **keep product logic platform-agnostic, then expose it through platform adapters**.

## Quick start

1. Read `README.md` and `CONTRIBUTING.md` before architecture-sensitive changes.
2. Use the primary product loop below for normal development. Surface-specific
   alternatives belong in the nearest app guide.
3. After Rust file changes, prefer `pnpm run fmt:rs` to format only changed or staged `.rs` files. Use `cargo fmt` only when you intentionally want broader formatting coverage.
4. After changes, use the nearest local `AGENTS.md` for the focused verification
   command. The repository-level verification section below only covers
   cross-cutting checks.
5. Workspace Rust dependencies own compatible versions, not broad capability
   unions. Each crate must select the dependency features it actually uses;
   keep test-only features in dev-dependencies and attach feature-gated service
   capabilities to the owning crate feature. Disable third-party defaults in
   `[workspace.dependencies]` when they are not part of every consumer's
   contract; members inherit that policy and add only their needed slices. For
   internal crates whose guarded `default` is empty, do not repeat
   `default-features = false` on every edge. Narrow consumers of an intentional
   compatibility default, such as ACP, must still disable it explicitly.
   Manifests copied into a standalone Docker build context must keep explicit
   versions and default policy because they cannot inherit the workspace root.
   `tokio/full` is forbidden in the root workspace and workspace members.

## Layered Module Index

Dependencies flow top to bottom. This table is the physical crate layout, not
the full conceptual architecture. For Product Surface / Product Assembly /
Product Feature / Agent Kernel / Execution / Extension / Cross-platform Adapter /
Stable Contracts and Security Control Plane boundaries, read
[`docs/architecture/product-architecture.md`](docs/architecture/product-architecture.md).
Keep crate dependencies inside each layer to the smallest set needed.

| # | Layer | Path | Owns | Modules / entries | Layer doc |
|---|---|---|---|---|---|
| 1 | Interfaces and entrypoints | `src/apps/*`, `src/web-ui`, `src/mobile-web`, `OpenBitFun-Installer`, `tests/e2e`, `src/crates/interfaces` | Product hosts, commands, UI entrypoints, protocol interfaces, and cross-surface tests | desktop, CLI, server, relay, Web UI, mobile web, installer, E2E, `acp`, `app-server`, `sdk-host` | nearest local `AGENTS.md`; [interfaces](src/crates/interfaces/AGENTS.md) |
| 2 | Product assembly | `src/crates/assembly` | Compatibility exports, product capability selection, product-full wiring, immutable built-in Agent content, adapter/service registration, and ecosystem-neutral source coordination | `agent-content`, `core`, `external-sources`, `product-capabilities` | [AGENTS.md](src/crates/assembly/AGENTS.md) |
| 3 | Adapters | `src/crates/adapters` | AI/transport/WebDriver protocol adapters, external AI work source adapters (OpenCode/Claude Code/Codex), and external-provider translation | `agent-runtime-ipc`, `ai-adapters`, `opencode-adapter`, `claude-code-adapter`, `codex-adapter`, `static-hook-support`, `transport`, `webdriver` | [AGENTS.md](src/crates/adapters/AGENTS.md) |
| 4 | Services | `src/crates/services` | Reusable OS, filesystem, terminal, MCP, remote, git, watch, process, session persistence primitives, MiniApp runtime IO, and network implementations | `services-core`, `services-integrations`, `miniapp-market-service`, `relay-service`, `page-function-runtime`, `terminal` | [AGENTS.md](src/crates/services/AGENTS.md) |
| 5 | Execution primitives | `src/crates/execution` | Portable Agent Runtime, named-workflow policy, stream, plugin runtime client, typed-service, tool-contract, tool-group, and tool-execution building blocks | `agent-runtime`, `agent-workflows`, `agent-stream`, `tool-contracts`, `plugin-runtime-client`, `runtime-services`, `tool-provider-groups`, `tool-execution`, `tool-call-jsonrepair` | [AGENTS.md](src/crates/execution/AGENTS.md) |
| 6 | Stable contracts and product domains | `src/crates/contracts` | Shared DTOs, event shapes, runtime ports, and product domain contracts/policies | `core-types`, `events`, `runtime-ports`, `product-domains` | [AGENTS.md](src/crates/contracts/AGENTS.md) |

Boundary rules:

- Interfaces and app entrypoints expose selected product behavior; reusable behavior moves down.
- Assembly wires lower layers and selects product capability facts; it must not implement concrete adapter, OS, or service details.
- Product features assemble user-facing commands, UI contributions, settings, and default policy on top of kernel capabilities; long-running task, scheduler, permission, session/workspace, memory, DFX, hook, and event facts stay in Agent Kernel owners.
- Adapters translate protocols and external-provider shapes; they should not own product capability selection or reusable OS service behavior.
- Services implement reusable concrete OS, process, terminal, MCP, remote, git, filesystem, and MiniApp runtime IO capabilities.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GCWing/OpenBitFun](https://github.com/GCWing/OpenBitFun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
