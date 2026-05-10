---
trigger: always_on
description: Runtime rule enforcement for AI agent tool calls. Deterministic pipeline: checks, output checks, session rules, principal-aware enforcement. Eight framework adapters (LangChain, CrewAI, Agno, Semantic Kernel, OpenAI Agents SDK, Claude Agent SDK, Nanobot, Google ADK). Zero runtime deps in core.
---

# CLAUDE.md

## What is Edictum

Runtime rule enforcement for AI agent tool calls. Deterministic pipeline: checks, output checks, session rules, principal-aware enforcement. Eight framework adapters (LangChain, CrewAI, Agno, Semantic Kernel, OpenAI Agents SDK, Claude Agent SDK, Nanobot, Google ADK). Zero runtime deps in core.

Current version: 0.18.0 (PyPI: `edictum`)

## Architecture: Core + Server

Two deployment units. One library, one server.

- `src/edictum/` -- MIT core. All rule types (pre, post, session, sandbox), pipeline, 8 adapters, audit to stdout/file/OTel, local approval backend, single-process session tracking.
- `src/edictum/server/` -- Server SDK client (`pip install edictum[server]`). Implements core protocols (`ApprovalBackend`, `AuditSink`, `StorageBackend`) over HTTP to connect agents to the server.
- Hosted control plane (`edictum-api` + `edictum-app`) -- Separate deployment for centralized approval workflows, audit dashboards, distributed sessions, and hot-reload rules.

## THE ONE RULE

**Core code (src/edictum/) runs fully standalone. The server SDK (src/edictum/server/) imports from core. The control plane itself is a separate deployment.**

Core provides protocols and interfaces. The server SDK provides HTTP-backed implementations. The control plane provides the coordination infrastructure.

## Core (MIT)

- CheckPipeline (evaluation engine)
- ToolCall, Principal model, Session (MemoryBackend)
- YAML rule parsing + validation + templates + composition
- All 8 framework adapters
- Sandbox rules (`type: sandbox`) — allowlist-based governance for file paths, commands, and domains
- Observe mode
- on_postcondition_warn callbacks
- AuditEvent dataclass + StdoutAuditSink + FileAuditSink (.jsonl) + RedactionPolicy
- OTel span instrumentation + GovernanceTelemetry
- Violation classification (`findings.py`, `Finding`) with pii_detected, secret_detected, policy_violation types

## Server

The control plane is a separate deployment. It provides:

- Production approval workflows (ServerApprovalBackend connects to Telegram, Slack, Discord, webhook, and review UI)
- Centralized audit ingestion and governance dashboard (block rates, rule drift, sandbox violations)
- Distributed session state for multi-agent tracking across processes
- Hot-reload rules via SSE push (ServerContractSource) without restarting agents
- RBAC for rule management (who can create/modify/deploy rules)
- Cross-agent session tracking (correlate tool calls across agents)
- SSO integration (Okta, Azure AD) and JWT/OIDC principal verification

## Boundary Principle

The split follows one rule: **evaluation = core library, coordination = server.**

- Pipeline that takes a tool call and returns allow/block/warn -- core
- Persistence beyond local files, networking, coordination across processes -- server
- Stdout + File (.jsonl) sinks for dev/local audit -- core. Centralized audit dashboards and alerting -- server
- OTel instrumentation (emitting spans) -- core. Governance dashboards -- server
- Session (MemoryBackend) for single-process -- core. Multi-process session state via the hosted control plane -- server
- LocalApprovalBackend for development approval -- core. Production approval workflows (Telegram, Slack, Discord, review UI) -- server

## Dropped Features (do NOT implement)
- **Python CLI** — removed entirely. Go binary is the canonical CLI.
- **Gate install/uninstall** — removed from Python. Gate install is Go-only (`edictum gate install`).

- `reset_session()` — new run_id handles this naturally
- Redis StorageBackend — not our problem, application layer concern
- DB StorageBackend — OTel already covers queryable audit data

## What's Shipped

- v0.5.0: Core library — pipeline, 6 adapters, YAML rules, OTel, observe mode
- v0.5.1: Adapter bug fixes (CrewAI, Agno, SK)
- v0.5.2: Adapter bug fixes (LangChain, OpenAI)
- v0.5.3: Claude SDK on_postcondition_warn callback, edictum test (removed — use Go CLI)
- v0.5.4: Dry-run evaluation API (evaluate, evaluate_batch), edictum test --calls
- v0.6.0: Postcondition enforcement effects (redact/block), SideEffect classification
- v0.6.1: YAML tools: section for side-action classifications
- v0.6.2: Renamed to_sdk_hooks() → to_hook_callables()
- v0.7.0: env.* selector, Edictum.from_multiple() guard merging, Claude Code GitHub Actions
- v0.8.0: Bundle composition (compose_bundles, from_yaml multi-file), dual-mode evaluation
- v0.8.1: ContractResult → RuleResult rename, terminology enforcement
- v0.9.0: YAML extensibility (custom_operators, custom_selectors, metadata.* selector, template_dirs, from_yaml_string), adapter lifecycle (on_deny, on_allow, success_check, set_principal, principal_resolver), CompositeSink, --json/--environment flags (removed — use Go CLI), OTel TLS
- Docs overhaul: homepage, quickstart, concepts section, patterns, 7 guides
- edictum-demo repo: github.com/edictum-ai/edictum-demo

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [edictum-ai/edictum](https://github.com/edictum-ai/edictum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
