---
trigger: always_on
description: MQTT-based personal AI assistant. Coordinator + A2A-over-MQTT agents + MQTT broker as infrastructure backbone.
---

# Skitter

MQTT-based personal AI assistant. Coordinator + A2A-over-MQTT agents + MQTT broker as infrastructure backbone.

## Quick Orientation

| What | Where |
|---|---|
| Coordinator (A2A orchestrator, session management, DAG dispatch) | `skitter/coordinator/` |
| Agent runner (CLI-to-A2A convenience wrapper) | `skitter/agent_runner.py` |
| Discovery (build + parse A2A agent/workflow cards) | `skitter/discovery.py` |
| LLM client (Anthropic + OpenAI SDKs) | `skitter/llm.py` |
| Graph generation + validation | `skitter/graph_gen.py` |
| Runtime API + app creation | `skitter/runtime_api.py` |
| A2A protocol (message types, topics, validation, requester helper) | `skitter/a2a.py` |
| MQTT v5 transport (connection, properties, broker probe) | `skitter/mqtt.py` |
| Config loading (~/.skitter/), dataclasses | `skitter/config.py` |
| DB interface (SQLite/PostgreSQL) | `skitter/db.py` |
| Agent/skill scaffolding (runtime-backed) | `skitter/create_agent.py` |
| App/session management (coordinator wrappers) | `skitter/manage.py` |
| One-shot A2A request | `skitter/request.py` |
| Interactive A2A session client | `skitter/cli.py` |
| Container management (up/down/status/logs) | `skitter/services.py` |
| Interactive setup wizard | `skitter/setup.py` |
| Diagnostic health checks | `skitter/doctor.py` |
| CLI dispatch | `skitter/__main__.py` |
| Dashboard (single-file, MQTT-connected) | `dashboard.html` |

## Docs

| Doc | Content |
|---|---|
| `docs/architecture.md` | Design principles, topic scheme, execution flows, recovery model |
| `docs/spec/a2a.proto` | A2A v1.0.0 proto (canonical schema) |
| `docs/spec/a2a-specification.md` | A2A v1.0.0 spec (JSON serialization rules, examples) |
| `docs/spec/a2a-over-mqtt-transport.md` | A2A-over-MQTT v0.1 transport binding |
| `docs/spec/a2a-over-mqtt-architecture.md` | A2A-over-MQTT design rationale |
| `CONTRIBUTING.md` | Project structure, config reference, env vars, testing, lint |
| `README.md` | User-facing quickstart, how-it-works |

## Architecture in One Paragraph

Clients publish JSON-RPC requests to `$a2a/v1/request/{org}/{unit}/{agent_id}`. Any A2A-over-MQTT compliant agent can handle requests; skitter ships an agent-runner as a convenience for wrapping CLI tools, but it's not required. For composed apps, the coordinator subscribes to the app's request topic, creates a DB-backed session, and dispatches A2A requests to individual agents. The coordinator only sends A2A requests and collects replies and doesn't care how agents are implemented. Locally: agents + Docker EMQX.

## Key Concepts

- **SKITTER_HOME.** All config and agent definitions live under `~/.skitter/` by default. Override with `SKITTER_HOME` env var or `--skitter-home` CLI flag.
- **Immutable sessions.** Persisted once by coordinator in DB, never mutated. Per-task status tracked separately.
- **Namespace separation.** `$a2a/v1/...` for client-facing A2A protocol (request, reply, discovery, events).
- **Native sub-agents.** Agent identity owned by the external runtime (Claude Code or Codex). Skitter reads metadata from definition files (`~/.skitter/agents/*.md`, `~/.skitter/agents/*.toml`); the agent-runner passes system instructions directly to the CLI tool and streams results back over MQTT.
- **Skills.** Reusable instruction modules stored in `~/.skitter/skills/<name>/SKILL.md` (YAML frontmatter with name/description, followed by instructions). Agent definitions reference skills by name via `skills: [name1, name2]` in frontmatter. At startup, the agent-runner symlinks referenced skills into the runtime-native path (`.claude/skills/` for Claude Code, `.agents/skills/` for Codex) inside the agent's resource directory, and sets `cwd` so the runtime discovers them natively. Skills are shared across agents.
- **Independent agents.** Agents are any A2A-over-MQTT compliant process. The coordinator doesn't spawn or manage them.
- **Session continuity.** Both Claude Code and Codex agents support multi-turn conversations via A2A `context_id`. The agent-runner captures the CLI-native session ID (`session_id` from Claude, `thread_id` from Codex) on the first request and maps it to the A2A `context_id`. Subsequent requests with the same `context_id` use `--resume <session_id>` for Claude or `codex exec resume <thread_id>` for Codex. On resume, Claude skips `--permission-mode` and `--settings` (inherited from the original session). Apps also support conversation continuity: the coordinator loads prior completed sessions for the same `(app_id, context_id)` and injects conversation history into task prompts.
- **A2A protocol compliance.** All protocol-facing code must conform to A2A v1.0.0 and the A2A-over-MQTT v0.1 binding. Local copies of the specs live in `docs/spec/`. Use `/a2a-compliance` to validate after protocol changes.

## Command Model

| Category | Runs where | Examples |
|---|---|---|
| **Local CLI** | Host process, no containers needed | `setup`, `create-agent`, `doctor`, `list-agents`, `list-apps` |
| **Service management** | Host process, manages Docker containers | `up`, `down`, `status`, `logs` |
| **A2A client** | Host process, talks to running services over MQTT | `ask` (primary), `chat`, `request`, `create-app`, `cancel-session` |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [id/skitter](https://github.com/id/skitter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
