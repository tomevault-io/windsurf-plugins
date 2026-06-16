---
trigger: always_on
description: aegis                         # full-screen TUI (opens ConfigPanel
---

# Agents

## Running

    aegis                         # full-screen TUI (opens ConfigPanel
                                  # when there's no .aegis.yaml)
    aegis serve                   # headless: MCP plane + optional Telegram
    aegis config ...              # scriptable .aegis.yaml authoring
                                  # (agent / queue / telegram / default-agent
                                  #  / plugin-dir / show)

`aegis` and `aegis serve` both resolve the project root via
`find_project_root()` (closest ancestor containing `.aegis.yaml`); the
harness subprocess is rooted there unless `--cwd` overrides.
`.aegis.yaml` is the single config substrate — it carries `agents:`,
`queues:`, `telegram:`, `schedules:`, `remotes:`, `groups:`, and
`plugin_dirs:` sections. Drop-in overlays live under
`.aegis/{agents,queues,schedules,groups}/*.yaml` and merge fail-loud
with inline entries. `@workflow`-decorated functions are registered by
auto-importing every `*.py` under each `plugin_dirs` entry (default
`.aegis/plugins/`).

## Package management

Use `uv` (not pip): `uv pip install -e .`, `uv run pytest`.

## Layout

- `src/aegis/cli.py` - typer entrypoint (`aegis`, `aegis serve`,
  `aegis workflow`, `aegis budget`, `aegis schedule`)
- `src/aegis/cli_config.py` - the `aegis config ...` subapp; all writing
  verbs route through `aegis.config.edit` helpers.
- `src/aegis/tui/config_panel.py` - the TUI ConfigPanel tab + AddAgentModal;
  mounted at boot when there's no `.aegis.yaml`, also reachable mid-session
  via `F2`.
- `src/aegis/config/__init__.py` - Agent / Permission / Effort /
  Provider dataclasses + `find_project_root`, `load_config`,
  `load_queues`, `load_telegram_config` — all YAML-backed thin
  wrappers around `aegis.config.yaml_loader.load_config`.
- `src/aegis/config/yaml_loader.py` - the real YAML parser:
  `.aegis.yaml` + overlays → `AegisConfig` (agents, queues, schedules,
  remotes, groups, telegram, plugin_dirs). Fail-loud on default_agent /
  queue-agent / max_parallel violations.
- `src/aegis/drivers/` - HarnessDriver seam + concrete drivers:
  `claude.py` (Claude Code, full-featured — multi-turn via stream-json
  INPUT, per-invocation MCP injection via `--mcp-config`),
  `gemini.py` (Gemini CLI, v1 one-shot — `gemini -p <prompt>
  --output-format stream-json --approval-mode <mode>`),
  `opencode.py` (OpenCode, v1 one-shot — `opencode run <message>
  --format json -m <provider/model>`). Per-driver stream parsers in
  `gemini_parse.py` and `opencode_parse.py` map each CLI's events into
  the canonical `aegis.events` types. Gemini and OpenCode workers do
  NOT inject aegis MCP in v1 (their MCP config is global, not
  per-invocation) — workers can do their task but cannot call back to
  `aegis_enqueue`; sufficient for queue-worker semantics where the
  substrate captures the worker's final assistant text as the result.
  Per-provider config classes (`ClaudeCode`, `GeminiCLI`, `OpenCode`)
  in `config.py` carry only the fields each provider actually uses;
  legacy flat `Agent(harness="…", model=…, effort=…, permission=…)`
  shape still works via a back-compat shim.
- `src/aegis/events.py` - stream-json parser (typed events)
- `src/aegis/render.py` - pure render_event(ev) -> Rich renderable | None
- `src/aegis/core/` - harness-agnostic session core: `AgentSession`
  (turn loop, metrics, state, observer callbacks — `session.py`) and
  `SessionManager` (AppBridge impl: spawn/close/interrupt/handoff over
  many AgentSessions — `manager.py`). The TUI's ConversationPane and the
  Telegram frontend both delegate to these.
- `src/aegis/telegram/` - Telegram bot front-end: `BotClient` (long-poll
  Bot API with exponential backoff + `retry_after` handling — `bot.py`),
  pure formatting helpers (`format.py`: `escape_md`, `status_line`,
  `chunk`), and `TelegramFrontend` (`/new /close /interrupt /agents
  /sessions /<handle> /help`, bare-text routing with auto_prompt suffix,
  mid-turn status refresher — `frontend.py`). Activated by `aegis serve`
  when the `telegram:` block (`token` + `chat_id`) is configured in
  `.aegis.yaml` (token also accepted via `AEGIS_TELEGRAM_TOKEN`, which
  wins over the YAML value).
- `src/aegis/tui/` - Textual app shell (app.py) + per-tab ConversationPane
  (pane.py), TabBar/StatusBar (widgets.py), AgentState (state.py),
  SessionMetrics (metrics.py), generated handles (names.py), AgentPicker
  modal (picker.py), Theme registry + AegisColors role map (themes.py;
  `aegis-ink` default)
- `src/aegis/mcp/` - FastMCP server (`server.py`: BRIEFING/PRIMING,
  `aegis_meta` + slice-2 inter-agent tools `aegis_list_sessions`,
  `aegis_list_agents`, `aegis_handoff` + queue-v1 tools `aegis_enqueue`,
  `aegis_task_status`; `mcp_config_json`) + `AppBridge`/`SessionInfo`
  (`bridge.py`: pure Protocol the server consumes; `AegisApp` and
  `SessionManager` both implement it) + `AegisMCP` runtime
  (`runtime.py`: co-resident HTTP server, port pick, start/stop,
  `bind(bridge)`). The app owns one shared instance, binds itself,
  starts it before the first spawn, and injects strict
  (`--mcp-config` + `--strict-mcp-config`) into every spawned claude
  alongside a primer system-prompt that bakes the pane's handle

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apiad/aegis](https://github.com/apiad/aegis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
