---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Essential Commands

### Initial Setup

```bash
mix deps.get
```

The termbox2 NIF source (in `packages/raxol_terminal/lib/termbox2_nif/c_src/`) is vendored directly in the repo -- no git submodules needed.

### Building & Compilation

```bash
MIX_ENV=test mix compile
MIX_ENV=test mix compile --warnings-as-errors
```

### Testing

```bash
MIX_ENV=test mix test --exclude slow --exclude integration --exclude docker
MIX_ENV=test mix test test/path/to/test_file.exs      # specific file
MIX_ENV=test mix test test/path/to/test_file.exs:42   # specific line
MIX_ENV=test mix test --max-failures 5                 # limit failures
MIX_ENV=test mix test --failed                         # rerun failed
```

Note: `TMPDIR=/tmp` and `SKIP_TERMBOX2_TESTS=true` are set automatically via `.claude/settings.json`. `MIX_ENV=test` must be specified explicitly for compile and test commands.

### Code Quality

```bash
mix raxol.check               # All checks: format, compile, credo, dialyzer, security, test
mix raxol.check --quick       # Skip dialyzer
mix raxol.check --only format,credo  # Run specific checks only
mix raxol.check --skip test   # Skip specific checks
mix format                    # Format code
mix format --check-formatted  # Check formatting (CI)
mix credo                     # Style checks
mix dialyzer                  # Type checking
```

### Running Examples

```bash
mix run examples/getting_started/counter.exs  # Known working example (TEA model)
```

Working examples: `counter.exs`, `getting_started/todo_app.exs`, `apps/todo_app.ex`, `apps/showcase_app.exs`, `demo.exs` (all TEA pattern).
`demo.exs` is the flagship demo showing dashboard layout, live stats, and OTP differentiators.

Agent examples: `agents/code_review_agent.exs` (single agent with shell commands), `agents/agent_team.exs` (coordinator + worker team pattern), `agents/ai_cockpit.exs` (multi-agent AI cockpit with real LLM streaming -- mock by default, `FREE_AI=true` for LLM7.io, supports Anthropic/OpenAI/Ollama/Groq).

Sensor examples: `sensor_hud_demo.exs` (3 mock sensors with gauge, sparkline, threat HUD Components).

Adaptive examples: `adaptive_ui_demo.exs` (behavior tracking, layout recommendations, feedback loop).

Playground: `mix raxol.playground` -- interactive Component catalog with 30 demos across 8 categories (input, display, feedback, navigation, overlay, layout, visualization, effects). Demos are self-contained TEA apps in `lib/raxol/playground/demos/`. Chart demos use View DSL functions directly. SSH mode: `mix raxol.playground --ssh` serves the playground over SSH (port 2222 by default). Production SSH enabled via `RAXOL_SSH_PLAYGROUND=true` env var in fly.toml.

### Development

```bash
mix phx.server                # Start Phoenix server (includes Tidewave in dev)
mix raxol.gen.specs lib/path  # Generate type specs for private functions
mix docs                      # Generate documentation
```

### Headless MCP Tools

`mix mcp.server` starts the MCP server on stdio (for Claude Code integration). Six Raxol-specific tools are registered at startup: `raxol_start`, `raxol_screenshot`, `raxol_send_key`, `raxol_get_model`, `raxol_stop`, `raxol_list`. Tools are auto-derived from the Component tree via `Raxol.MCP.ToolProvider` -- each interactive Component exposes semantic actions (e.g., Button exposes `click`, TextInput exposes `type_into`/`clear`/`get_value`). Set `mcp_exclude: true` in Component attrs to suppress tool derivation for internal Components.

### Development Scripts

```bash
./scripts/dev.sh test [pattern]  # Run tests with grep filter
./scripts/dev.sh test-all        # Comprehensive test suite
./scripts/dev.sh check           # Pre-commit quality checks
./scripts/dev.sh dialyzer        # Static analysis with PLT caching
./scripts/dev.sh setup           # Environment setup
```

## Architecture

Raxol is a multi-surface application runtime for Elixir built on OTP. One TEA module renders to terminal, browser (LiveView), SSH, and MCP (agent surface). It covers the component model, agent runtime, sensor fusion, distributed swarm, and time-travel debugging. Four UI paradigms: React, LiveView, HEEx, Raw.

### Application Model

**TEA (The Elm Architecture) is the canonical app model.** Applications implement `init/1`, `update/2`, and `view/1` callbacks, mapped to a GenServer via `Raxol.start_link/2` which delegates to `Raxol.Core.Runtime.Lifecycle.start_link/2`. Do not introduce competing application models (e.g., LiveView-style `mount/render`).

```elixir
use Raxol.UI, framework: :react      # React patterns (TEA)
use Raxol.UI, framework: :liveview   # Phoenix LiveView patterns
use Raxol.UI, framework: :heex       # Phoenix templates
use Raxol.UI, framework: :raw        # Direct terminal control
```

### Extracted Packages

The codebase splits into focused packages under `packages/`:

```
packages/
├── raxol_core/      # Behaviours, utils, events, config, accessibility, plugins
├── raxol_terminal/  # Terminal emulation (VT100/ANSI), termbox2 NIF, screen buffer
├── raxol_sensor/    # Sensor fusion (zero Raxol deps)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DROOdotFOO/raxol](https://github.com/DROOdotFOO/raxol) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
