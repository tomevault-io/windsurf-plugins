---
trigger: always_on
description: **This file is a ROUTER, not a manual.** It carries only the rules whose violation
---

# Rules for AI Assistants

**This file is a ROUTER, not a manual.** It carries only the rules whose violation
causes damage before a pointer could be read. Everything else is a link you MUST
open before touching that subsystem: see
[Read before you touch](#read-before-you-touch). The frontend has its own router,
[`website/AGENTS.md`](website/AGENTS.md).

## What this is

Kiro Crew is an open-source personal AI agent: chat from the web dashboard, the
CLI, or a messaging channel like Slack and Discord; run multi-step tasks
unattended; schedule cron jobs; keep memory across sessions.

Kiro Crew's sole LLM provider speaks ACP. Its default backend runs `kiro-cli`
over ACP JSON-RPC; other verified ACP harnesses are selected with
`agent.acp_backend`. MCP tools supply the agent's host capabilities.

- **Backend:** Python package `kiro_crew` in `src/kiro_crew/`. **Frontend:** React
  + TS + Vite SPA in `website/`, built into `src/kiro_crew/static/dist/` and served
  by the backend.
- **Data home:** `~/.kiro/crew`, overridden with `KIROCREW_HOME`.
- **Distribution:** public GitHub, plain setuptools, public PyPI / public npm.

Full map: [overview](docs/architecture/overview.md). This repo is the de-Amazoned
public fork of an internal package; what must never come back is
[oss-fork-boundaries](docs/system-specs/oss-fork-boundaries.md), gated by
the `internal-content-scan` check (blocking on pull requests, forks included) and
the
`no-new-builtin-apps` rule in `AUTOSDE.yaml`.

## Read before you touch

Load the doc for the row you are working in **before** you change code. Update it
in the **same commit** when you change what it documents.

| If you are touching… | Read first |
|---|---|
| `platform/`, editions, CPP seam, governance | [platform-context](docs/system-specs/modules/platform-context.md) + [governance](docs/system-specs/modules/governance.md) |
| `security.py`, `hooks.py`, denied commands, sensitive paths | [security](docs/system-specs/modules/security.md) + [sel](docs/system-specs/modules/sel.md) |
| `config/` — the live watcher, `restart=True` marks, appliers, `config.json` writes | [config](docs/system-specs/modules/config.md) |
| the security model as a whole, threat boundaries | [security-deep-dive](docs/architecture/security-deep-dive.md) |
| `computer_use/` | [computer-use](docs/system-specs/modules/computer-use.md) |
| monitoring loops, `monitoring/`, `irq.py`, watches | [monitor-architecture](docs/system-specs/modules/monitor-architecture.md) (the paradigm) + [agent-interrupt-controller](docs/system-specs/modules/agent-interrupt-controller.md) + [babysit-pr-watch](docs/system-specs/modules/babysit-pr-watch.md) |
| `acp/`, kiro-cli transport, providers | [acp-client](docs/system-specs/modules/acp-client.md) + [providers](docs/system-specs/modules/providers.md) |
| picking or defaulting a model anywhere | [model-selection](docs/system-specs/common/model-selection.md) + [model-fallback](docs/system-specs/modules/model-fallback.md) |
| adding or adapting an agent harness (BYO, KAS, claude) | [harness-parity](docs/system-specs/modules/harness-parity.md) (invariants) + [harness-parity-gate](docs/ci/harness-parity-gate.md) (CI) |
| an agent spec: `agent_discovery.py`, `agent_spec_format.py`, `agent.py`'s spec writers, `acp/kas_agents.py`, or any field a spec carries | [agent-spec-fields](src/kiro_crew/docs/agent-spec-fields.md) (what each field does, per backend) + [agent-host-contract](docs/system-specs/modules/agent-host-contract.md) (the per-harness table) |
| the publicly selectable Claude backend | [claude-code-provider](docs/system-specs/modules/claude-code-provider.md) |
| sessions, slots, session keys, PIDs | [session](docs/system-specs/modules/session.md) + [history](docs/system-specs/modules/history.md) |
| session summaries, the chat summary panel, intent extraction | [session-summary](docs/system-specs/modules/session-summary.md) |
| memory, embeddings, vectors, lessons, skills, hooks | [memory-skills-hooks](docs/system-specs/modules/memory-skills-hooks.md) |
| `context.py`, `context_blocks.py`, what reaches the model's context | [context-management](docs/architecture/context-management.md) |
| MCP servers or tools (adding, changing, statelessness) | [mcp](docs/architecture/mcp.md) |
| apps, App Kit, manifests, app agents | [app-kit-platform](docs/system-specs/modules/app-kit-platform.md) + [app-kit/](docs/app-kit/README.md) |
| artifacts, companion chat | [artifacts](docs/system-specs/modules/artifacts.md) |
| `stt/`, `transcribe.py`, `voice_reply.py`, the mic, dictation, TTS | [stt-streaming](docs/system-specs/modules/stt-streaming.md) + [voice-streaming](docs/system-specs/modules/voice-streaming.md) |
| cron, learn, dashboard handlers | [learn-cron-dashboard](docs/system-specs/modules/learn-cron-dashboard.md) |
| Slack, Discord, any channel, messaging, approvals | [messaging](docs/system-specs/modules/messaging.md) + [slack-gateway](docs/system-specs/modules/slack-gateway.md) |
| subagents, spawn, orphan recovery | [subagent](docs/system-specs/modules/subagent.md) |
| crews, `select_crew`, crew bindings, Crew Mode slots | [crew-mode](docs/system-specs/modules/crew-mode.md) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kirodotdev/KiroCrew](https://github.com/kirodotdev/KiroCrew) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
