---
trigger: always_on
description: This file is the operating manual for **every AI coding agent** working in this
---

# AGENTS.md

This file is the operating manual for **every AI coding agent** working in this
repository. Read it fully before making any change. If a task ever conflicts
with this file, this file wins — stop and ask for clarification.

---

## 1. Project mission

This repository is a **declarative platform for building AI agent systems**.

Developers describe an agent system in YAML. The platform validates that YAML,
compiles it into a typed internal graph, and runs it through a long-lived
runtime that renders prompt files, calls resolver/tool plugins and MCP servers,
exposes an API, and produces execution traces.

The repository is in **active development**. The YAML validator, compiler,
runtime engine (orchestrators run as supervisor agents), resolver
plugin system, tool plugin loading, MCP client (local and remote servers,
including authenticated MCP via hooks), the runtime hooks system
(11 lifecycle points — see
[docs/RUNTIME_HOOKS.md](docs/RUNTIME_HOOKS.md)), per-run execution-limit
guardrails (see [docs/EXECUTION_LIMITS.md](docs/EXECUTION_LIMITS.md)), prompt
rendering, and the CLI (`validate`, `inspect`, `generate`, `run`, `serve`,
`chat`) are implemented. Model access supports both Anthropic and Amazon
Bedrock. Two HTTP API layers exist: a thin `agent_engine` API (`/invoke`,
`/stream`), started by `agentctl serve` (default port `8090`) — stateless, no
persistence, no web client — and `agent_manager` — a conversation lifecycle
service built on top of it with SQLite-backed persistence, SSE streaming, and
the official React web client, started by the separate `agent-manager`
console script (default port `8100`). `agentctl chat` is a separate, ephemeral
developer console that persists nothing (see `docs/ARCHITECTURE.md` §14 for
detail). Basic observability
(structured logging plus a Langfuse callback provider) is wired in. A
`Dockerfile`/`entrypoint.sh` provide a basic container image.

The access plugin **is** wired into child filtering, but the request-context
gate that should populate real identity/permissions into it is **not yet
implemented** — access filtering currently runs against an empty context, so
`protected` nodes are not actually enforced today. Treat this as an open
security gap, not a finished feature. See [docs/ROADMAP.md](docs/ROADMAP.md)
for per-phase status. Agents implement the product task-by-task using the
files in `tasks/`, though the package layout has evolved beyond what the
original numbered tasks describe (see §4).

---

## 2. Core architecture pipeline

```
config.yml
  → validate          (schema + semantic validation)
  → compile           (typed internal models)
  → CompiledAgentGraph (immutable, built once)
  → RuntimeEngine     (long-lived, created at startup)
  → ExecutionContext  (created per request)
  → recursive agent execution
  → response + trace
```

Per-request flow:

```
Incoming request
  → Security/Context Gate
  → RuntimeEngine
  → filter protected nodes through access plugin
  → execute root as a supervisor agent (children exposed as tools)
  → resolve prompt variables through resolver plugins
  → render prompts
  → orchestrators synthesise; leaf agents execute their tools
  → call configured tools/MCP as needed
  → return response + trace
```

**Three separated phases (do not collapse them):** (1) **build/compile** —
load, validate, and compile YAML into a `CompiledAgentGraph` *before*
serving requests (never executes requests); (2) **runtime/execution** — per
request, create an `ExecutionContext`, filter protected nodes, route to a node
instance, render prompts, execute, call tools/MCP, return response + trace;
(3) **client extension** — client-specific auth/business logic lives in
plugins, never in the generic runtime.

---

## 3. Non-negotiable architecture rules

These rules are binding. Do not violate them, even if a task description seems
to ask for it. If a rule blocks you, stop and raise it.

1. **YAML is declarative specification, not executable business logic.**
2. **The runtime must never execute raw YAML dictionaries directly.**
3. **YAML must be validated first.**
4. **Validated YAML must be compiled into internal typed models** before use.
5. **RuntimeEngine is created once at application startup.** Never per request.
6. **ExecutionContext is created per request.** Never reused across requests.
7. **Do not store request state on RuntimeEngine** or on the compiled graph.
8. **Prompt files are templates.** They contain placeholders, not final text.
9. **Prompt templates may be cached.** Compiled/parsed templates are reusable.
10. **Prompt values are resolved dynamically per request.** Never cache a
    fully-rendered prompt globally.
11. **Client-specific auth, authorization, and business context are handled
    through plugins**, not baked into the generated runtime.
12. **Nodes declare what they need** (resolvers, tools, MCPs). **The runtime
    resolves and binds** those needs.
13. **Protected nodes must fail closed** through the fixed access plugin.
14. **Prompt text alone is not a security boundary.** Enforcement happens at the
    tool/data layer.
15. **Secrets must never be stored in YAML or prompt files.**
16. **Every meaningful behavior must have tests.**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [extra-org/extra](https://github.com/extra-org/extra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
