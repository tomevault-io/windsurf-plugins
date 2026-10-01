---
trigger: always_on
description: This file holds the design rules every change follows. [Working in this repository](#working-in-this-repository) links to everything else.
---

# OpenAgentCore development

This file holds the design rules every change follows. [Working in this repository](#working-in-this-repository) links to everything else.

## Design principles

OpenAgentCore is protocol-first and modular. Core orchestrates operations that protocols define; Sandbox Providers, Runtimes, Harnesses and model providers are replaceable implementations of those protocols. [Architecture](docs/architecture.md) describes each component's responsibilities.

### Protocols at every boundary

- Each boundary between components has exactly one protocol: one code file (interface, wire types and validators) and one document. A protocol change edits both and every implementation in one change, reviewed on its own.
- Protocols are deterministic. Every operation is declared and every outcome is typed. Implementations declare what they support, and callers validate each selected combination against those declarations; they never discover support through type assertions, name checks or implicit fallbacks. An unsupported operation or combination returns a typed error. Core never substitutes another implementation, and a capability means the same for every implementation.
- A component joins the system only by implementing a protocol, never through a private entry point, side channel or path selected by its name.
- Each rule has one authored definition. Generate cross-language projections from it or check them against shared fixtures.

| Boundary | Protocol code | Protocol doc |
| --- | --- | --- |
| Application–Core (`/v1`) | Types in `contracts/agents-api/v1/` and route annotations in `services/core/internal/api/`; `make openapi` generates `contracts/agents-api/openapi.yaml` | [Agents API guide](docs/api/public-agent-api.md) |
| Web and operators–Core (`/core/v1`) | Route annotations in `services/core/internal/api/`; `make openapi` generates `contracts/agents-api/core.openapi.yaml` | [Core administration API](contracts/agents-api/admin-api.md) |
| Nodes and daemons–Core (`/api/v1` HTTP routes; the node and daemon wire protocols are separate rows) | Route annotations in `services/core/internal/api/`; `make openapi` generates `contracts/agents-api/runtime.openapi.yaml` | [Machine connection API](contracts/agents-api/machine-api.md) |
| Core–Sandbox Provider | `services/core/internal/sandbox/sandbox_provider.go` | [Sandbox Provider guide](docs/sandbox-provider.md) |
| Core–sandbox node | `services/core/internal/sandbox/node/wire.go` | [Sandbox node protocol](contracts/agents-api/node-generation-protocol.md) |
| Provider–Runtime startup | `internal/runtimebootstrap/bootstrap.go` | [Runtime bootstrap](docs/runtime-bootstrap.md) |
| Core–Runtime wire | `internal/agentdaemon/proto/` | [Core–Runtime protocol](docs/runtime-protocol.md) |
| Runtime–Harness | `apps/daemon/internal/agent/harness.go` | [Harness onboarding](contracts/agents-api/harness-onboarding.md) |
| Harness–Model provider | `internal/modelprovider/config.go` | [Model execution](contracts/agents-api/model-execution.md) |

Each row names the protocol's code entry point and its document.

### Complexity stays in the adapter

- New complexity lives in the adapter that needs it and never spreads outward. A new Sandbox Provider, Harness, model provider or vendor feature changes only its adapter. It adds no Core execution path, store table or column, migration, deployment or configuration field, API field or Web UI specific to one vendor or Harness.
- The [Sandbox Provider guide](docs/sandbox-provider.md) and [Harness onboarding](contracts/agents-api/harness-onboarding.md) describe how to add an adapter.
- When the protocol cannot express what an adapter needs, change the protocol. Never add an optional side interface for one implementation.
- Example: implementing the complete declared `CheckpointProvider` lifecycle in one vendor's Provider is an adapter change. A vendor-only pause interface, a Core path for that vendor, vendor receipts in the store or a vendor idle setting in the deployment is not.
- Fix shared lifecycle, admission, cancellation, reuse and performance problems in the common flow, never in branches selected by a Harness, Runtime or vendor name. Core preparation and execution never branch on operating system or Environment source; platform support requires native CI builds and automated tests.
- Each Harness runs its own model and tool loop through a maintained upstream SDK or native protocol, in the Environment's declared workspace directory; its native history or configuration directory is never the workspace. Never build a second executor, a hand-written model/tool loop or a general-purpose compatibility framework to fabricate parity. The public API and persistence never depend on one engine's native item types.

### Public API


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MiniMax-AI/OpenAgentCore](https://github.com/MiniMax-AI/OpenAgentCore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
