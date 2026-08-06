---
trigger: always_on
description: Guidance for AI coding agents (GitHub Copilot, Claude, etc.) working in this
---

# AGENTS.md

Guidance for AI coding agents (GitHub Copilot, Claude, etc.) working in this
repository. Read this top section before any change. Route focused work to the
appropriate domain guide rather than loading the entire codebase. 

1. Read this file
2. Identify the domain of the task.
3. Load the matching `.subagents/AGENTS-{DOMAIN}.md` rules file into current 
   context as primary instruction, or spawn a subagent(s) scoped to that domain 
   if the task is complex.
4. If applicable, aggregate subagent outputs in the orchestrating session.

### Subagent routing table

| You're working on… | Load this guide | Owns |
|---|---|---|
| Agent loader internals, session/token management, model types | [.subagents/AGENTS-CORE.md](.subagents/AGENTS-CORE.md) | `internal/agent`, `internal/session`, `internal/model` |
| **Author-facing agent file format** (`*.agent.md`, `*.lifecycle.yaml`, front-matter, multi-turn) | [.subagents/AGENTS-AGENTDEF.md](.subagents/AGENTS-AGENTDEF.md) | The user-visible agent definition spec |
| **Tool / skill / toolset resolution** (precedence, naming, per-turn scope) | [.subagents/AGENTS-TOOLS-SKILLS-TOOLSETS.md](.subagents/AGENTS-TOOLS-SKILLS-TOOLSETS.md) | Resolution semantics across `tools`, `toolsets`, `skills` |
| Agent execution, tool calling, MCP runtime, response caching, bot messaging | [.subagents/AGENTS-RUNTIME.md](.subagents/AGENTS-RUNTIME.md) | `internal/runner`, `internal/tool`, `internal/mcp`, `internal/cache`, `internal/notify` |
| Scheduling, queues, background HTTP poll workers | [.subagents/AGENTS-WORKER.md](.subagents/AGENTS-WORKER.md) | `internal/scheduler`, `internal/queue`, `internal/worker` |
| **Tannery** (event-driven curing service, hides, artifacts) | [.subagents/AGENTS-TANNERY.md](.subagents/AGENTS-TANNERY.md) | `internal/curing`, `internal/artifact`, `internal/hide`, `internal/safepath` |
| Config loading, CLI subcommands, schema validation, flag/env wiring, HTTP API | [.subagents/AGENTS-SERVE.md](.subagents/AGENTS-SERVE.md) | `internal/config`, `internal/cli`, `internal/schema`, `internal/secret`, `internal/devtools`, `cmd/leather` |
| **`shell-mcp` companion binary** (manifest format, templating, `--no-shell`, JSON-RPC conformance) | [.subagents/AGENTS-SHELL-MCP.md](.subagents/AGENTS-SHELL-MCP.md) | `cmd/shell-mcp` |
| **Browser UI** (`ui/`) | [.subagents/AGENTS-UI.md](.subagents/AGENTS-UI.md) | `ui/` SPA, design tokens, API contract layer |
| **Replay subsystem** (capture, storage format, action endpoints, redaction) | [.subagents/AGENTS-REPLAY.md](.subagents/AGENTS-REPLAY.md) | Replay capture + `/replay/...` API + replay UI views |
| Tests, benchmarks, Makefile, CI, linting | [.subagents/AGENTS-QUALITY.md](.subagents/AGENTS-QUALITY.md) | `*_test.go`, `Makefile`, `.github/workflows` |
| **Shared stdlib leaf utilities** (atomic file writes, JSON persistence, ID generation, flat-YAML parsing, HTTP response helpers) | [.subagents/AGENTS-QUALITY.md](.subagents/AGENTS-QUALITY.md) | `internal/fileutil`, `internal/jsonstore`, `internal/ids`, `internal/yamlx`, `internal/httpx` |
| **Performance** (hot paths, benchmarks, allocation budgets, pprof, baseline) | [.subagents/AGENTS-PERFORMANCE.md](.subagents/AGENTS-PERFORMANCE.md) | Performance posture; cross-cutting |
| **Security** (threat model, secret handling, API authn/authz, trust boundaries, prompt-injection) | [.subagents/AGENTS-SECURITY.md](.subagents/AGENTS-SECURITY.md) | Trust-boundary policy; cross-cutting |
| **Operations** (deploy layout, systemd/launchd, single-process lock, backup/restore, upgrade) | [.subagents/AGENTS-OPERATIONS.md](.subagents/AGENTS-OPERATIONS.md) | Deployment + lifecycle; cross-cutting |
| **Integrations authoring** (how to add a notifier, MCP server, webhook worker, Skeptic-style scanner) | [.subagents/AGENTS-INTEGRATIONS.md](.subagents/AGENTS-INTEGRATIONS.md) | Authoring patterns across `internal/notify`, `internal/mcp`, `internal/worker` |
| **Examples & tutorials** (`tanning/`, demo agents/skills/toolsets, `docs/tutorials/`) | [.subagents/AGENTS-EXAMPLES.md](.subagents/AGENTS-EXAMPLES.md) | `tanning/`, tutorial sequence, example-as-test policy |
| **Observability** (log levels per component, run history JSONL, status/health/metrics endpoints) | [.subagents/AGENTS-OBSERVABILITY.md](.subagents/AGENTS-OBSERVABILITY.md) | `internal/logging`, history records, `/status`, `/metrics`, `/healthz` |

If a task spans multiple domains, spawn one subagent per domain in parallel and
merge their findings.

**Maintaining this file:** a dedicated agent skill handles keeping AGENTS.md
and all subagent guides synchronized with the codebase. See
[.agents/skills/agents-doc-lifecycle/SKILL.md](.agents/skills/agents-doc-lifecycle/SKILL.md).

**Release workflow:** two agent skills automate the release process:
- [`release-prep`](.agents/skills/release-prep/SKILL.md) — version detection, CHANGELOG, docs update, commit + push.
- [`release-tag`](.agents/skills/release-tag/SKILL.md) — pre-flight gates, annotated tag push, pipeline trigger.

---

## What leather is

`leather` is a slim local agent runtime, orchestration harness, and
workflow interface for developer workstations and home-network servers. No external dependencies, stdlib-only Go.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TGPSKI/leather](https://github.com/TGPSKI/leather) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
