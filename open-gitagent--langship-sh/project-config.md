---
trigger: always_on
description: Langship is a framework-agnostic deployment, governance, and operations layer for agent applications. It supports LangChain/LangGraph and other agent frameworks (LlamaIndex, CrewAI, AutoGen, Pydantic AI, raw SDK agents, etc.) — not tied to any single framework.
---

# Langship

Langship is a framework-agnostic deployment, governance, and operations layer for agent applications. It supports LangChain/LangGraph and other agent frameworks (LlamaIndex, CrewAI, AutoGen, Pydantic AI, raw SDK agents, etc.) — not tied to any single framework.

## Three pillars

- **Deployment** — packaging, versioning, rollouts, env/secret management for agent apps
- **Governance** — policies, budgets, approvals, audit logs, tenant isolation, safety filters
- **Operations** — monitoring, replay/debug, incident response, cost & performance management

## Deployment targets

Langship is multi-runtime. Supported deployment targets include:

- **Kubernetes** (self-hosted / any cloud)
- **AWS Bedrock AgentCore Runtime**
- **GCP Vertex AI Agent Engine**

The deployment layer abstracts over these runtimes so the same agent app definition can ship to any of them. Governance and operations policies must apply uniformly across runtimes.

## Positioning

Closer to a "platform for agents" (Kubernetes/Datadog-style) than a single-framework tool like LangSmith or LangGraph Platform.

## Top-level scope — Project

Langship's top-level scoping primitive is a **Project**. A Project owns: users + roles, environments, workflows, secrets, deployment configs, audit log. Self-hosted Langship typically serves several teams, so Projects isolate one team/agent from another from day one (rather than retrofitting tenancy later). API paths are scoped: `/projects/:id/...`. Note: "Project" in this doc always refers to the Langship Project; cloud-provider projects (e.g., GCP project) are always qualified with the provider name.

## CI/CD — configurable workflow builder

CI/CD is **not** a fixed pipeline. It is a **configurable, drag-and-drop workflow builder** where pipelines are graphs of nodes (triggers, build, test, eval, policy, approval, deploy, promote, rollback). The visual canvas is the UI; YAML in git is the source of truth (GitOps).

- **Two modes** — replace CI entirely (Langship runs build/test/eval/deploy) or CD + release-gates only (external CI hands off an artifact, Langship picks up at eval/governance/deploy).
- **Role-based views** — agent devs, platform/DevOps, governance owners see different node palettes on the same underlying graph.
- **Environments are first-class** — dev, staging, prod each have their own pipeline. Per-env config (secrets, runtime, scaling) is separate from the graph. Different runtimes per env are expected (e.g., dev on K8s, prod on Vertex Agent Engine).
- **Promotion is gated and branching-strategy-driven** — evals must pass → approval gate (human, automated policy, or quorum) → `Promote` node executes per project's branching strategy (trunk-based, env-branches, release branches, or custom). Promotion and rollback are auditable events.
- **Governance is a node, not a wrapper** — policies/approvals/budget gates are first-class, visible, reorderable steps in the graph.
 

## Deployment model — self-hosted

Langship is **self-hosted by the customer**. The customer runs the whole stack (API server, Restate, Postgres, workers, secrets manager) in their own infrastructure. Distribution is via Helm chart / installer / Docker Compose for local dev.

**Why self-hosted:**
- Customer's cloud credentials, agent code, eval data, and audit logs never leave their network — strong fit for the governance-focused positioning
- Clear regulatory story for finance/healthcare/gov buyers who can't adopt hosted control planes
- Simpler security model: no cross-tenant credential storage, no proxy of LLM traffic
- Customer's compliance team monitors logs in systems they already operate

**Trade-offs accepted:**
- Higher friction to adopt vs. hosted SaaS — installer/upgrade UX matters more
- Support is harder — no production access by default; need good telemetry-with-consent + clear runbooks
- Distribution: ship a Helm chart as primary path; Docker Compose for local dev / small teams

**Hosted offering may come later** as a managed deployment of the same stack, but the product is designed self-hosted-first. CLI/UI/API contracts assume the server is something the customer operates.

## Server architecture — three layers

The "server" is actually three layers, all running in the customer's infrastructure:

1. **API / control plane** — REST or gRPC. CLI and UI call this. Handles auth, RBAC, workflow CRUD, run triggers, approvals, audit queries. Stateless app servers.
2. **Orchestration layer** — Restate (primary) cluster + worker pool. Workers execute node logic (build, eval, deploy, etc.). Long-lived and durable.
3. **Data layer** — MongoDB (runs, approvals, audit, users, projects, workflows index); S3-compatible object store (artifacts, large eval outputs, trace blobs); secrets manager (Vault or cloud-native KMS) for cloud credentials. Postgres runs alongside, but **only** as Restate's required persistence backend — never accessed by Langship app code.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [open-gitagent/langship.sh](https://github.com/open-gitagent/langship.sh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
