---
trigger: always_on
description: Arbor is a distributed AI agent orchestration system built on Elixir/OTP. Umbrella project with capability-based security and contract-first design.
---

# CLAUDE.md

Arbor is a distributed AI agent orchestration system built on Elixir/OTP. Umbrella project with capability-based security and contract-first design.

## Core Concepts

- **Agent**: A supervised entity with a cryptographic Ed25519 identity, trust profile, and granted capabilities. Created via `Arbor.Agent.Lifecycle.create/2`. Runs as a `BranchSupervisor` (rest_for_one) with children: APIAgent host, Executor, and Session.
- **Session**: GenServer in `arbor_orchestrator` that drives agent turns (user messages → LLM responses) and heartbeats (autonomous cycles) by executing DOT graph pipelines via the Engine.
- **Heartbeat**: An autonomous ~30-second cycle where the agent runs a DOT pipeline to check goals, select a cognitive mode (goal pursuit / reflection / plan execution / consolidation), optionally call an LLM, update memory, and execute pending actions — all without human input.
- **DOT Pipeline**: A directed graph written in DOT/Graphviz syntax defining a workflow of typed nodes connected by edges. Node types:
  - `exec` — runs a Jido Action (pure business logic) via ExecHandler
  - `compute` — makes an LLM call via LlmHandler/ComputeHandler
  - `diamond` (shape=diamond) — conditional routing based on a context key
  - `start` (shape=Mdiamond) / `done` (shape=Msquare) — entry/exit sentinels
  - `compose` / `invoke` — embed or call sub-pipelines via SubgraphHandler
  Edges can be conditional (`condition="context.key=value"`), enabling branching and retry loops. A shared key-value **context** flows through the graph — nodes read from and write to it. The graph is a static definition; the Engine provides dynamic execution.
- **Engine** (`Arbor.Orchestrator.Engine`): Executes DOT pipeline graphs by traversing nodes in topological order, dispatching each to the appropriate handler, managing checkpoints for resume, and emitting lifecycle events. The Engine is the core execution loop — it calls handlers, collects results, evaluates conditional edges to pick the next node, and tracks node durations. The execution model itself is sound; the lifecycle tracking infrastructure around it (how completed/failed status is recorded) is being redesigned — see `.arbor/roadmap/2-planned/engine-lifecycle-redesign.md`.
- **CRC Pattern (Construct-Reduce-Convert)**: Pure functional modules that separate business logic from side effects. `new/1` constructs from input, operations transform state, `show/1` formats for output. All functions are pure — no DB, no GenServer calls, no IO. Used extensively in dashboard cores. See [`.claude/skills/functional-core.md`](.claude/skills/functional-core.md).
- **Socket-First Component**: Dashboard pattern replacing LiveComponent with plain Phoenix.Component modules that manage state on the parent LiveView's socket via delegate functions. Events namespaced as `"component:action"`. See [`.claude/skills/socket-component.md`](.claude/skills/socket-component.md).
- **Trust Tier**: Graduated trust level governing agent capabilities: `untrusted` (0-19) → `probationary` (20-49) → `trusted` (50-74) → `veteran` (75-89) → `autonomous` (90-100). Managed by `arbor_trust`.
- **Capability**: An unforgeable, signed token granting a specific permission on a resource URI (e.g., `arbor://fs/read/`, `arbor://shell/exec/git`). Granted via `Arbor.Security.grant/1`, checked via `Arbor.Security.authorize/4`. Supports delegation, expiry, constraints (rate limits), and revocation.
- **Identity**: Ed25519 + X25519 keypair. Agent ID is `"agent_" <> hex(SHA-256(public_key))` — deterministically derived, unforgeable. Private keys stored encrypted at rest via `SigningKeyStore`. External agents authenticate via per-request `SignedRequest` signatures verified by `Arbor.Gateway.SignedRequestAuth`.
- **Signal**: Fire-and-forget pub/sub event for observability. Emitted via `Arbor.Signals`, consumed by dashboards, event stores, and monitoring. NOT used for lifecycle tracking or execution control — observability only.
- **Memory**: Per-agent working memory (ETS-backed `MemoryStore`), knowledge graph, and background health checks. Managed by `arbor_memory`. Goals and intents persisted via `BufferedStore` (ETS + optional Postgres backend).

## Fix the Root Cause

Don't perform actions just to unblock something immediately so you can move on. Always fix the root cause.

## Security Bug Fixes Need Regression Tests

When you fix a security bug — anything where a check failed open, an authorization gate was bypassed, an invariant was silently violated — the diff MUST include a committed test that:

- **fails on `git checkout HEAD~1`** (proves the bug existed)
- **passes on `HEAD`** (proves the fix closes it)

A live verification via tidewave / iex is not enough. It tells you the gate is closed *today*. It does not tell you the gate stays closed when someone refactors the auth pipeline six months from now. Without a committed test, every security fix is a one-shot — the next refactor can silently re-open the hole and nothing will catch it.

The test should:
- Live in the same library as the fix (so it runs when that library's tests run)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [trust-arbor/arbor](https://github.com/trust-arbor/arbor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
