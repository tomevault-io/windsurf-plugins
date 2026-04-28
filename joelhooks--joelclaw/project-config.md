---
trigger: always_on
description: Personal AI infrastructure monorepo. Event-driven pipelines, always-on gateway, CLI tooling, and the joelclaw.com website. Built by Joel Hooks, operated by agents.
---

# joelclaw — Agent Instructions

Personal AI infrastructure monorepo. Event-driven pipelines, always-on gateway, CLI tooling, and the joelclaw.com website. Built by Joel Hooks, operated by agents.

## Architecture

```
┌─ Mac Mini "Panda" (M4 Pro, 64GB, always-on) ──────────────────────┐
│                                                                     │
│  Colima VM (VZ framework, aarch64)                                  │
│    └─ Talos v1.12.4 container → k8s v1.35.0 (single node)         │
│        └─ namespace: joelclaw                                       │
│            ├─ inngest-0          (StatefulSet, ports 8288/8289)     │
│            ├─ redis-0            (StatefulSet, port 6379)          │
│            ├─ typesense-0        (StatefulSet, port 8108)          │
│            ├─ restate-0          (StatefulSet, ports 8080/9070/9071) │
│            ├─ system-bus-worker  (Deployment, port 3111)           │
│            ├─ restate-worker     (Deployment, port 9080)           │
│            ├─ dkron-0            (StatefulSet, port 8080)          │
│            ├─ minio-0            (StatefulSet, ports 9000/9001)    │
│            ├─ docs-api           (Deployment, port 3838)           │
│            ├─ livekit-server     (Deployment, ports 7880/7881)     │
│            └─ bluesky-pds        (Deployment, port 3000)           │
│                                                                     │
│  Gateway daemon (pi session, always-on, Redis event bridge)         │
│  NAS "three-body" (ASUSTOR, 10GbE NFS, 64TB RAID5 + 1.9TB NVMe)  │
│  Tailscale mesh (all devices)                                       │
└─────────────────────────────────────────────────────────────────────┘
```

Load the `system-architecture` skill before any cross-cutting architectural work, debugging event flow, or tracing why something ran/didn't run.

## Monorepo Layout

```
joelclaw/
├── apps/
│   ├── web/                    # joelclaw.com (Next.js 16, RSC, Vercel)
│   ├── docs/                   # Documentation site
│   └── docs-api/               # PDF/docs REST API (k8s)
├── packages/
│   ├── cli/                    # @joelclaw/cli — `joelclaw` command
│   ├── sdk/                    # @joelclaw/sdk — programmatic wrapper for CLI contracts
│   ├── system-bus/             # @joelclaw/system-bus — 110+ Inngest functions
│   ├── gateway/                # @joelclaw/gateway — multi-channel message routing
│   ├── restate/                # @joelclaw/restate — durable DAG runtime + workers
│   ├── inference-router/       # @joelclaw/inference-router — model selection catalog
│   ├── model-fallback/         # @joelclaw/model-fallback — provider fallback chains
│   ├── message-store/          # @joelclaw/message-store — Redis message queue
│   ├── vault-reader/           # @joelclaw/vault-reader — Vault context injection
│   ├── markdown-formatter/     # @joelclaw/markdown-formatter — per-platform formatting
│   ├── telemetry/              # @joelclaw/telemetry — OTEL emission interface
│   ├── pi-extensions/          # @joelclaw/pi-extensions — pi agent extensions
│   ├── email/                  # @joelclaw/email — email processing
│   ├── mdx-pipeline/           # @joelclaw/mdx-pipeline — MDX content processing
│   ├── lexicons/               # @joelclaw/lexicons — AT Protocol lexicons
│   ├── discord-ui/             # @joelclaw/discord-ui — Discord components
│   ├── things-cloud/           # @joelclaw/things-cloud — Things integration
│   └── ui/                     # @repo/ui — shared UI components
├── k8s/                        # Kubernetes manifests + deploy scripts
├── skills/                     # Agent skills (canonical, 76 skills)
├── scripts/                    # Utility scripts
├── infra/                      # Infrastructure config
│   └── firecracker/            # Firecracker microVM infrastructure
└── AGENTS.md                   # This file
```

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Runtime | Bun |
| Package manager | pnpm workspaces |
| Language | TypeScript (strict) |
| Web framework | Next.js 16 (App Router, RSC, PPR) |
| Effect system | Effect, @effect/cli, @effect/schema, @effect/platform |
| Workflows | Inngest + Restate DAG runtime |
| State | Redis (k8s StatefulSet) |
| Search | Typesense (full-text + vector) |
| Video | Mux |
| Sandbox substrate | Firecracker microVMs in k8s pods |
| Observability | OTEL → Typesense + Langfuse |
| Infrastructure | Talos Linux on Colima, k8s v1.35.0 |
| Deployment | Vercel (web), GHCR + k8s (worker) |
| Storage | NAS (NFS over 10GbE), Vault (Obsidian) |

## Hexagonal Architecture (ADR-0144) — MANDATORY

Heavy logic lives in standalone `@joelclaw/*` packages behind interfaces. Consumers (gateway, CLI) are thin composition roots that wire adapters together.

### Package Boundary Rules

- **Import via `@joelclaw/*`**, never via relative paths across package boundaries.
- **DI via interfaces** in library packages. Only composition roots do concrete wiring.
- **New heavy logic → new package** if it's >100 lines and reusable.
- **One telemetry interface** — `TelemetryEmitter` from `@joelclaw/telemetry`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joelhooks/joelclaw](https://github.com/joelhooks/joelclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
