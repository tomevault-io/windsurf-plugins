---
trigger: always_on
description: make install           # installs both server (uv sync) and frontend (npm ci)
---

# glossogen

## Setup

```bash
make install           # installs both server (uv sync) and frontend (npm ci)
make install-server    # server only
make install-frontend  # frontend only
```

## Linting

```bash
make lint              # runs both server and frontend linters
make lint-server       # server only (black, isort, ruff, pyright, vulture, custom linters)
make lint-frontend     # frontend only (prettier --write, eslint, stylelint, tsc)
make check-frontend    # frontend CI mode (prettier --check, no auto-fix)
```

## Project Structure

- For a step-by-step guide on adding a new scenario, see [docs/creating-a-scenario.md](docs/creating-a-scenario.md).
- For adding a metric, in this repo or in your own package, see [docs/creating-a-metric.md](docs/creating-a-metric.md).
- `src/` — application source code
- `src/glossogen/scenarios/<scenario_name>/` — one folder per scenario, containing:
  - `README.md` — scenario documentation
  - `scenario.py` — scenario class (channels, timing, tools, injections, turn logic, knobs schema)
  - `prompts/` — Jinja2 templates for agent system prompts and injection messages
  - `evaluation/` — scenario-specific metrics (optional)
  - `events.py` — scenario-specific `EventBase` subclasses (auto-discovered)
  - `run_detail_extension.py` — optional hook for surfacing scenario-specific data on the run-detail API
  - `scripts/` — one-off scripts that import directly from this scenario (smoke runners, probe-bank generators, etc.). Cross-scenario tools live in the repo-root `scripts/` instead.
- `src/glossogen/runtime/` — autonomous mode runtime (MCP server + coordination):
  - `simulation_state.py` — shared state: channels, sessions, locks, callbacks, world context, token counters, current round, injection delivery (`deliver_round_injections`, `deliver_postmortem_injections`, `has_postmortem_for_round`)
  - `mcp_tools.py` — MCP tool definitions (read_notifications, read_channel, send_message, etc.)
  - `mcp_server.py` — starts FastMCP over Streamable HTTP
  - `game_clock.py` — round progression and termination detection (delegates injection delivery to `SimulationRuntime`)
  - `agent_session.py` — per-agent notification queue, reaction delay, idle tracking
  - `scenario_mcp_tool.py` — ScenarioMcpTool for scenario-specific tool registration
  - `scenario_world.py` — ScenarioWorld ABC, WorldContext, MessageEvent, RoundAdvancedEvent
- `src/glossogen/runners/` — autonomous mode agent runners:
  - `agent_runner_base.py` — abstract base class for agent runners
  - `pydantic_ai_runner.py` — Pydantic AI agent runner via pydantic-ai framework
  - `pydantic_ai_model_factory.py` — per-provider mapping from `(model, provider)` to a pydantic-ai `model=` argument and default `ModelSettings`; shared by the runner and the platform's post-simulation `protocol_probe` helper
  - `communication_protocol.py` — shared prompts and constants for the agent communication protocol
- `src/glossogen/config_overrides.py` — Hydra-style dot-notation config override parser
- `src/glossogen/mcp_tool_rejection.py` — `surface_value_errors`, applied where scenario tools and the run browser's tools are registered. The MCP server shows the caller a `ToolError`'s message and reduces any other exception to `Error executing tool <name>`; this turns a `ValueError`, which is how code not written against MCP refuses a call, into a `ToolError` so the agent reads the reason. The base communication tools raise `ToolError` directly
- `src/glossogen/knob_filter.py` — the `<knob><operator><value>` filter grammar, parsed and applied against a run's recorded `scenario_config`. The knob name ends at the **first** operator and the longest one there wins, so a value may itself contain one (`judge_model=gpt>=5`). Comparison is typed from the recorded value rather than the knobs schema, so it imports no scenario class and a run predating a schema change still answers. A knob recorded as null is a value (`swap_round=null` selects the runs that never swapped); a knob the run never recorded matches nothing, even under `!=`. At the root because the listing, the export request model and the CLI's selection resolver all read it
- `src/glossogen/scenario_registry.py` — maps scenario name strings to the `SimulationScenario` classes shipped here; lives outside `glossogen.scenarios` package init so importing event-related modules doesn't trigger eager loading of every scenario
- `src/glossogen/scenario_loader.py` — the only way anything resolves a scenario name. Checks `SCENARIO_REGISTRY`, then scenarios other installed distributions declared. `get_scenario_class` raises, `find_scenario_class` returns `None`, `available_scenario_names` lists without importing, `iter_scenario_classes` imports every one
- `src/glossogen/scenario_entry_points.py` — reads the versioned `glossogen.scenarios.v<N>` entry-point group from installed metadata. Reading imports nothing, which is what lets event discovery cover external scenarios without re-entering the `models.event` import cycle

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agencyenterprise/GlossoGen](https://github.com/agencyenterprise/GlossoGen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
