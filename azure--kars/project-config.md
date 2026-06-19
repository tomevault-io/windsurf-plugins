---
trigger: always_on
description: A secure AI agent runtime on Azure AKS. OpenClaw agents run in isolated K8s sandbox pods with E2E encrypted inter-agent communication (Signal Protocol via AgentMesh). Each agent gets its own namespace, NetworkPolicy, seccomp profile, and inference router.
---

# Kars — Copilot Instructions

## What is Kars?

A secure AI agent runtime on Azure AKS. OpenClaw agents run in isolated K8s sandbox pods with E2E encrypted inter-agent communication (Signal Protocol via AgentMesh). Each agent gets its own namespace, NetworkPolicy, seccomp profile, and inference router.

## Architecture

Four components, two languages:

| Component | Language | Package Name | Role |
|-----------|----------|-------------|------|
| **Controller** | Rust (kube-rs) | `kars-controller` | K8s operator — reconciles `KarsSandbox` CRDs into isolated sandboxes (namespace, deployment, service, NetworkPolicy, ConfigMap) |
| **Inference Router** | Rust (axum) | `kars-inference-router` | Per-sandbox proxy — the **only** network path for agents. Handles IMDS auth, Content Safety, token budgets, the full Foundry data-plane API surface, AGT governance, sub-agent spawn |
| **CLI** | TypeScript | `@kars/cli` | 18 CLI commands (`kars up/add/dev/connect/handoff/mesh/...`) + OpenClaw plugin + 10 Foundry skills |
| **Policy Engine** | YAML profiles | — | AGT governance policy profiles (allow/deny/approval/rate-limit) |

**External dependencies:** [OpenClaw](https://openclaw.ai) (agent framework), [Azure AI Foundry](https://learn.microsoft.com/azure/ai-studio/) (managed AI services), [AGT](https://github.com/microsoft/agent-governance-toolkit) (governance layer).

### Sandbox pod structure

Each sandbox pod has 2 containers + 1 init container:
- **init: egress-guard** — iptables rules restricting UID 1000 to localhost + DNS only
- **openclaw** (UID 1000) — runs the OpenClaw agent with the Kars plugin
- **inference-router** (UID 1001) — Rust router on port 8443, all agent traffic flows through it

Agents never see API keys. The router authenticates via IMDS/Workload Identity.

### AgentMesh provider

Kars uses Microsoft AGT AgentMesh exclusively. TypeScript transport is provided by `@microsoft/agent-governance-sdk` through the `@kars/mesh` package, and AGT relay/registry are deployed via `deploy/agentmesh-agt.yaml`. The historical AgentMesh npm package and vendored relay/registry forks were removed in Phase 5.2 after AGT upstreamed Kars's gap-closing patches.

## Build, Test, and Lint

### Rust (edition 2024, MSRV 1.88)

```bash
make build                # builds controller + router + CLI
cargo build --release     # Rust only (both crates)
cargo test --all          # all Rust tests (74 controller + 105 router + 26 integration)

# Single crate:
cargo build --release --package kars-controller
cargo build --release --package kars-inference-router

# Single test:
cargo test --package kars-controller -- test_name
cargo test --package kars-inference-router -- test_name

# Lint:
cargo clippy --all-targets -- -D warnings
cargo fmt --all           # format
```

### TypeScript CLI (Node.js 22+)

```bash
cd cli
npm ci && npm run build    # compile + copy policy profiles to dist/
npm test                        # vitest
npm run lint                    # oxlint
npm run typecheck               # tsc --noEmit
```

### Docker images

```bash
make images               # builds controller + router images
make push                 # pushes to ACR

# Sandbox image (must use repo root as context):
docker build -f sandbox-images/openclaw/Dockerfile .
```

### E2E tests

```bash
make test-e2e             # requires Docker + Kind
```

## Key Conventions

### Image tags: always use `:latest`

Never hardcode version tags. The controller defaults to `:latest` (reconciler.rs ~line 945). Previous version tag drift (v11–v25) caused hard-to-debug issues. Don't manually set `SANDBOX_IMAGE`/`INFERENCE_ROUTER_IMAGE` env vars or CRD `openclaw.image` fields — let the controller use defaults.

### Plugin singleton guard

OpenClaw loads the plugin in multiple parallel contexts (gateway + tool registry + agent session — up to 5 contexts). A process-level singleton lock keyed off `Symbol.for("agt-mesh-client")` / `Symbol.for("agt-init-lock")` / `Symbol.for("agt-init-promise")` (see `runtimes/openclaw/src/index.ts` → `initAGT`) ensures only the first caller creates the AGT client; subsequent contexts reuse it. Don't remove this guard or weaken the synchronous lock — without it the plugin races and you get duplicate inbox messages.

### Sub-agent container lifecycle

`entrypoint.sh` starts the OpenClaw gateway (port 18789) in the background, then starts a persistent `openclaw agent --local` session. This background session loads the plugin → connects to AGT relay → receives/replies to E2E messages. Without it, the sub-agent can't receive relay messages.

### AGT mesh stack

Do not add a second mesh provider or restore the removed vendored AgentMesh fork. Mesh transport changes should target `mesh-plugin/src/agt-transport.ts` and, when broadly applicable, be proposed upstream to Microsoft AGT first. The Rust crate named `agentmesh` is from Microsoft AGT and remains a valid dependency.

### Rust workspace


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure/kars](https://github.com/Azure/kars) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
