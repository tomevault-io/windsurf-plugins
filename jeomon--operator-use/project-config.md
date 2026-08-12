---
trigger: always_on
description: Top-level guidance for Claude Code when working on this repository.
---

# Operator — CLAUDE.md

Top-level guidance for Claude Code when working on this repository.

## Project Overview

**Operator** is a stateful Python AI agent harness. It wraps a multi-provider LLM inference layer with session persistence, context compaction, an extension/package system, sandbox enforcement, browser/computer automation, and a gateway that connects the agent to multiple messaging channels simultaneously (Telegram, Discord, Slack, WebSocket, Email, Twitch, stdio).

Four layers build on each other:

1. **Engine** (`operator_use/engine/`) — raw LLM streaming loop, tool execution, abort signal. No knowledge of sessions or extensions.
2. **Agent** (`operator_use/agent/`) — turn orchestration, retry, compaction scheduling, extension event fan-out. Implements `ExtensionContext`.
3. **Runtime** (`operator_use/runtime/`) — session lifecycle, slash-command dispatch, gateway/cron/subagent wiring.
4. **Gateway** (`operator_use/gateway/`) — channel adapters, async message bus, per-session agent routing.

## Repository Layout

```
operator_use/
  agent/         engine/        session/       runtime/
  extension/     hooks/         resource/      gateway/
  inference/     settings/      package/       builtins/
  compaction/    subagent/      cron/          process/
  mcp/           acp/           auth/          tool/
  skill/         commands/      message/       prompt/
  browser/       computer/      peer/          team/
  workflow/      knowledge/     sandbox/       diagnostics/
  memory/        bus/           console/       templates/
tests/
docs/            ← read these before exploring code
```

## Quick Navigation

Find detailed documentation before reading source:

| Need to understand… | Read |
|---|---|
| Full system — channels → gateway → agent → engine → session | `docs/architecture.md` |
| Turn flow, retry, compaction scheduling | `docs/agent.md` |
| Guardrail interface, loading, loop detection | `docs/guardrails.md` |
| LLM loop, tool execution modes, queues | `docs/engine.md` |
| Session JSONL, branching, reconstruction | `docs/session.md` |
| Extension loading, `api.config`, dispatch | `docs/extensions.md` |
| Package install, `operator.json`, wiring | `docs/packages.md` |
| Event types, hook registration | `docs/hooks.md` |
| Channels, message bus, stream phases | `docs/gateway.md` |
| Models, providers, auth | `docs/inference.md` |
| Tool interface, execution modes | `docs/tool.md` |
| SKILL.md format, discovery | `docs/skill.md` |
| Slash commands, builtin commands | `docs/commands.md` |
| Credentials, OAuth, token refresh | `docs/auth.md` |
| Named profiles, AGENT.md format | `docs/profiles.md` |
| Browser automation via CDP | `docs/browser.md` |
| Desktop computer control | `docs/computer.md` |
| Sandbox policy, OS-level isolation | `docs/sandbox.md` |
| Knowledge base, document injection | `docs/knowledge.md` |
| Multi-agent teams, mailboxes | `docs/team.md` |
| Python workflows, DSL globals | `docs/workflow.md` |
| ACP transports, server/client | `docs/acp.md` |

## Key Invariants

Violating these breaks the architecture:

- **Layering is strict.** Engine ← Agent ← Runtime. Engine imports nothing from agent or runtime. Never skip a layer.
- **Single event funnel.** Every Engine event reaches extensions through `Agent._on_engine_event()` only. Nothing bypasses it.
- **Session writes are deferred.** Entries only hit disk after the first `AssistantMessage`. Failed retries are fully rewound before the next attempt.
- **Compaction runs at save points.** After `save_point` (turn complete, session durable) — never mid-turn.
- **Builtin names win.** Builtin tool/command/skill names always take priority over extensions and packages. No silent shadowing.
- **Extension errors are non-fatal.** Catch, log, continue. A handler exception must never abort an active turn.

## Common Tasks

**Add a builtin tool** → create `operator_use/builtins/tools/my_tool.py` exporting `tool = MyTool()`.

**Add a user tool** → drop `my_tool.py` in a profile's `~/.operator/profiles/<name>/tools/` or, for a project, in `<project>/.operator/tools/` (loaded automatically when Operator runs in that repo).

**Write an extension:**
```python
# ~/.operator/profiles/<name>/extensions/my_ext.py  (or <project>/.operator/extensions/my_ext.py)
from pydantic import BaseModel
from operator_use.extension.types import ToolDefinition
from operator_use.tool.types import ToolResult

class Params(BaseModel):
    text: str

async def _run(params, invocation, ctx):
    return ToolResult.ok(invocation.id, params.text.upper())

def extension(api):
    mode = api.config.get("mode", "default")   # set via extension_list in settings.json
    api.on("session_start", lambda e, ctx: None)
    api.register_tool(ToolDefinition(
        name="shout", description="Shout text.", parameters=Params, execute=_run,
    ))
```

**Configure an extension** in `~/.operator/settings.json`:
```json
{
  "extensions": true,
  "extension_list": [
    { "name": "my_ext", "enabled": true, "settings": { "mode": "strict" } }
  ]
}
```

**Install a package:**
```python
from operator_use.package.installer import install_package
from operator_use.settings.paths import get_packages_dir

install_package("git:github.com/user/my-tools", get_packages_dir())

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jeomon/Operator-Use](https://github.com/Jeomon/Operator-Use) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
