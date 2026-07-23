---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**android-ai-agent** is a Python framework for autonomous Android device automation via ADB, using LLMs routed through [OpenRouter](https://openrouter.ai). It runs on Termux (arm64) and requires zero Rust/C++ build dependencies.

## Setup

```bash
# Termux
pkg install python adb
git clone https://github.com/Mohd-Mursaleen/android-ai-agent && cd android-ai-agent
pip install -e .
export OPENROUTER_API_KEY="your-key-here"
python check.py

# macOS / Linux
chmod +x setup.sh && ./setup.sh
```

## Commands

```bash
# Health check
python check.py

# Run a task
python run.py "open Settings and show the Android version" --verbose
python run.py "Open Instamart, add milk to cart" --steps 40 --quiet

# python run.py options:
#   --steps N      hard cap on action count (default 25)
#   --device ID    ADB serial (auto-detected if omitted)
#   --quiet        suppress step-by-step output
#   --check        run health check only, then exit
#   --quality N    screenshot quality (default 100, no compression)
#   --screenshot   take screenshot, send to Telegram, exit (no automation)
```

**Testing:**

```bash
pytest
pytest tests/test_android.py
```

## Architecture

Three-component pipeline per action cycle:

1. **Executor** (`executor/android.py`) — takes a screenshot via `adb exec-out screencap -p`, compresses it by `image_quality` percent, returns base64. All ADB taps/swipes/types go through here.
2. **Planner** (`graph/nodes/planner.py`) — text-only LLM; breaks the goal into 2–5 ordered subgoals as JSON.
3. **Cortex** (`graph/nodes/cortex.py`) — vision LLM; receives screenshot + UI element list from Contextor, returns one JSON action.

All LLM calls go through `android_agent/openrouter.py` using the OpenAI-compatible SDK pointed at `https://openrouter.ai/api/v1`. Set `OPENROUTER_API_KEY` in the environment.

### Execution loop (`graph/runner.py`)

```
state = planner_node(state)          # build subgoal plan
while True:
    state = orchestrator_node(state) # activate next subgoal
    route = convergence_node(state)  # "end" / "replan" / "continue"
    if route == "end": break
    if route == "replan": state = planner_node(state); continue
    state = contextor_node(state, executor)   # screenshot + uiautomator dump
    state = cortex_node(state)               # decide next action
    state = executor_node.execute(state)     # run ADB command
    state = summarizer_node(state, executor) # verify action worked
```

### Lock / mutex system

Only one automation task can run at a time (single Android screen).

- **Lock file**: `~/storage/shared/android_agent/agent.lock`
- **Contents**: JSON with `pid`, `goal`, `started_at`
- **Acquired** by `run_task()` at start, **released** at end (including on crash via try/finally)
- **Stale detection**: If the lock file exists but the PID is dead, the lock is automatically cleaned
- **Busy check script**: `scripts/check_busy.sh` — returns exit 0 (`FREE`) or exit 1 (`BUSY: details`)
- **SKILL.md Step 0**: Iota must run `check_busy.sh` before every automation run

### Monitoring & Telegram notifications

Notifications are integrated directly into the Python runner — no external monitor scripts needed.

1. **Start notification** — sent by `run.py` before `run_task()` is called.
   Uses `urllib.request` (stdlib only). Includes goal string and max steps.

2. **Progress updates (every `_PROGRESS_EVERY` steps)** — sent by `runner.py` inside the
   main loop, after `summarizer_node` runs. Default: every 2 steps.
   Each update sends `state.latest_screenshot_b64` (the summarizer's fresh post-action
   screenshot) with a caption: step count, elapsed time, current subgoal, last action.

3. **Final result** — sent by `runner.py` in the `finally` block.
   Fires even if the run raised an exception or was cancelled.
   Takes one last fresh screenshot; falls back to `state.latest_screenshot_b64` on
   ADB failure. Includes success/failure emoji, step count, elapsed time,
   completed/failed subgoals, and failure reason if applicable.

`_notify_telegram()` in `runner.py` uses `urllib.request` (stdlib — no `requests` package).
`_PROGRESS_EVERY = 2` is a module-level constant in `runner.py`; change it to tune frequency.

### Screen wake behavior

`scripts/wake_and_unlock.sh` is a smart idempotent script:
- Checks `mWakefulness` via `dumpsys power` — only sends WAKEUP if screen is off
- Checks `deviceLocked` via `dumpsys trust` — only swipes if locked
- **Never presses Home or Back** — current app state is preserved
- Safe to run before every automation — does nothing if screen is already awake and unlocked
- The Planner handles navigation to the correct app via subgoals

### Coordinate scaling

Default quality is 100 (no compression, no scaling). If quality is reduced,
`AndroidExecutor.image_scale_factor = 100 / quality`. When screenshots are compressed to N% of original size, coordinates from visual analysis are in that smaller space. `click_at_a_point` multiplies by `image_scale_factor` to hit the correct physical pixels.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PsProsen-Dev/OpenClaw-On-Android](https://github.com/PsProsen-Dev/OpenClaw-On-Android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
