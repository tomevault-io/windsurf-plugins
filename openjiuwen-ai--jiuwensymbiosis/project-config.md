---
trigger: always_on
description: Shared instructions for AI coding assistants working in `jiuwensymbiosis`.
---

# AGENTS.md

Shared instructions for AI coding assistants working in `jiuwensymbiosis`.
Keep this file cross-tool (Cursor / Copilot / Claude all read it). Prefer
nearby code and tests over assumptions.

`pyproject.toml` is the canonical source of truth for Python/tooling
settings. `CLAUDE.md` imports this file (`@AGENTS.md`) and adds
Claude-specific pointers only.

## Project Overview

JiuwenSymbiosis is an embodied agent framework built on `openjiuwen` for robotics. It provides hardware-agnostic tools, safety policies, and multi-agent collaboration. The core design principle is **Capability Mixin architecture**: a single codebase adapts to different robot form factors (SCARA, 6-DoF, suction cup, gripper) through mixin composition — new hardware only needs a YAML config + 6 adapter files.

## Build & Test Commands

```bash
# One-stop entry points (see Makefile); defaults to conda env "jiuwensymbiosis",
# override with `make check CONDA_ENV=` to use plain PATH.
make check        # ruff format --check + ruff check + mypy on staged files (mypy advisory)
make fix          # ruff format + ruff check --fix on staged files
make test         # pytest tests/unit_tests/ (no hardware/GPU)
make test-all     # pytest (incl. integration)
# Use COMMITS=N to check files changed in the last N commits instead of staged.

# Install in editable mode
pip install -e ".[dev]"                                    # core + test deps
pip install -e ".[full]" --extra-index-url https://download.pytorch.org/whl/cu128  # + vision/GPU deps
pip install -e ".[piper]"                                  # + piper hardware SDK
pip install -e ".[gui]"                                    # + 图形界面 (NiceGUI, 浏览器模式)

# Run tests
pytest                                                     # all unit tests (no hardware needed)
pytest tests/unit_tests/                                   # unit tests only (no hardware/GPU)
pytest -m integration                                      # integration tests (needs hardware/GPU)
pytest tests/unit_tests/agent/test_builder.py              # single test file
pytest -k "test_capabilities"                              # filter by test name

# Validate a hardware adapter
python scripts/validate_adapter.py --module jiuwensymbiosis.adapters.my_robot

# Run demo (mock mode: no hardware, no real LLM — uses MockArmEnv + MockModel).
# Task is not in the config — pass it via --query (or --voice at run time).
python examples/piper_pick_demo.py --config configs/piper/piper.yaml --mock --query "<任务>"
# CLI entry point (after pip install)
piper-pick-demo --config configs/piper/piper.yaml --mock --query "<任务>"

# Run the GUI (browser mode; selects a body + real config and runs on hardware)
python -m jiuwensymbiosis.gui        # or the console script: jiuwensymbiosis-gui
#   Opens the default browser at http://127.0.0.1:<port> (NiceGUI, never native=True,
#   so no pywebview/WebKitGTK). No extra system libs needed. The GUI does a startup
#   pre-check (jiuwensymbiosis/gui/preflight.py) and prints the exact package to
#   install (pip install -e ".[gui]") instead of crashing with a raw traceback.

# Lint / format / type-check (tools not installed by default; install: pip install ruff mypy)
ruff format .           # format (Black-compatible drop-in)
ruff check .            # lint; ruff check --fix . to auto-fix
mypy jiuwensymbiosis/
```

## Critical: Proxy Hygiene

`clear_proxy_env()` (defined in `jiuwensymbiosis/utils/proxy.py`, exported from `jiuwensymbiosis.utils` and `jiuwensymbiosis`) **must** be called before `import openjiuwen`. HTTP proxy env vars cause `httpx` to require `socksio` and route localhost through proxy, breaking local vLLM/detection calls. The root `conftest.py` does this automatically for tests.

## Centralised Logging

`jiuwensymbiosis.utils.logging` provides one choke point for all logging:

- `configure_logging(level="INFO", *, log_dir=None)` — idempotent root-logger setup: one `StreamHandler` with a uniform format (`%(asctime)s %(levelname)s %(name)s: %(message)s`) plus an optional `RotatingFileHandler` (`<log_dir>/jiuwensymbiosis.log`, 5 MB / 3 backups). `build_robot_agent` calls it with `RobotAgentConfig.log_level` / `log_dir`.
- `get_logger(name=None)` — thin alias over `logging.getLogger`; new code should use it. Legacy `logging.getLogger(__name__)` calls remain valid.
- The Piper driver's per-run `commands.log` (`_attach_cmd_log_handler`) now routes through `configure_logging` + a tagged `FileHandler` with the same format. Disable with `JIUWEN_PIPER_CMD_LOG=0`; override dir with `JIUWEN_PIPER_CMD_LOG_DIR`.
- `TraceLogHandler` — a `logging.Handler` that forwards `WARNING`+ records from `RobotAgentConfig.trace_capture_loggers` (default `["jiuwensymbiosis"]`) into the active execution trace, so rail warnings / detector failures land in the trace with no business-code changes.

## Architecture: Layered Capability-Gated Design

The framework has 7 layers, with data flowing top-down for commands and bottom-up for observations:

```
Agent Layer       RobotSession + build_robot_agent() + RobotAgentConfig
Safety Rails      SafetyRail / RecoveryRail / VisualFeedbackRail / SkillUseRail (before_tool_call hooks); TraceRail (parallel, optional)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openJiuwen-ai/jiuwensymbiosis](https://github.com/openJiuwen-ai/jiuwensymbiosis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
