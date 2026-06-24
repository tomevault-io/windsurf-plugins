---
trigger: always_on
description: This file documents project-specific guidance for Claude Code and defines available skills/tools.
---

# Claude Code Configuration

This file documents project-specific guidance for Claude Code and defines available skills/tools.

## Project Overview

**City Simulation** — A realistic 3D city simulation with LLM-powered autonomous agents.

- **Architecture**: Godot 4 (C# / GDScript) + Python sidecar (asyncio, websockets, Anthropic SDK)
- **Status**: M1–M5 complete (city scene, living world, visual polish, scripted agents, LLM brains)
- **Repository**: [pufacz/citysimulation](https://github.com/pufacz/citysimulation)
- **Tech Stack**: Godot 4.3 (Forward+), Python 3.11+, asyncio, websockets, Anthropic Claude

## Skills

### GitHub Sync Skill

**Purpose**: Sync local git repository to GitHub with customizable organization and repository name.

**Usage**:
```bash
python3 sync_repo_skill.py --org <organization> --repo <repo_name> [options]
```

**Parameters**:
- `--org, --organization`: GitHub organization or username (required)
- `--repo, --repository`: Repository name on GitHub (required)
- `--private`: Make repository private (default: public)
- `--force`: Force push to overwrite remote history
- `--branch, -b`: Branch to push (default: current branch)
- `--no-push`: Create repository without pushing

**Examples**:
```bash
# Sync to organization repo (public)
python3 sync_repo_skill.py --org pufacz --repo citysimulation

# Private repository
python3 sync_repo_skill.py --org myorg --repo myproject --private

# Force push a specific branch
python3 sync_repo_skill.py --org pufacz --repo test --force --branch main
```

**Location**: `./sync_repo_skill.py` (Python) or `./sync-to-github.sh` (Bash)
**Documentation**: See `SYNC_SKILL_README.md`

## Development Workflow

### M1–M5 Complete — What's Implemented

| Milestone | Status | Focus | Files |
|-----------|--------|-------|-------|
| **M1** | ✅ | City scene (buildings, roads, lighting, camera) | `godot/scripts/city_*.gd` |
| **M2** | ✅ | Living world (traffic, pedestrians, HUD) | `godot/scripts/vehicle.gd`, `pedestrian*.gd` |
| **M3** | ✅ | Visual polish (weather, LOD, props, follow camera) | `godot/scripts/weather.gd`, `free_fly_camera.gd` |
| **M4** | ✅ | Scripted agents + sidecar skeleton | `godot/scripts/agent*.gd`, `brain/citybrain/*.py` |
| **M5** | ✅ | LLM-powered agent decisions (Anthropic) | `brain/citybrain/llm.py`, `budget.py` |

### M6 — Next Phase

**Goal**: Long-term memory, agent conversations, player chat, multi-provider support.

**Key tasks**:
- Agent memory persistence (reflection, summarization)
- Inter-agent conversations (when agents meet)
- Player ↔ agent chat interface
- Multi-provider abstraction (Anthropic, OpenAI, etc.)
- Save/load world state

### Running the Project

**Godot Editor** (interactive):
```bash
godot --path godot -e
```

**Godot Headless** (for testing, automation):
```bash
godot --headless --path godot --quit-after 300
```

**Python Sidecar**:
```bash
cd brain

# With Anthropic API key (real LLM decisions)
uv run python -m citybrain

# Or, mock LLM mode (no API calls, canned decisions)
CITYBRAIN_MOCK_LLM=1 uv run python -m citybrain
```

**Full Stack** (Godot + sidecar in parallel):
```bash
# Terminal 1: Start sidecar
cd brain && uv run python -m citybrain

# Terminal 2: Run Godot
godot --path godot
```

### Configuration

**Agent LLM Profiles**: [`godot/agent_profiles.json`](godot/agent_profiles.json)
- Named presets: `default`, `premium`, `flagship`
- Per-agent assignments with overrides
- Resolution order: built-in defaults → `default` profile → assigned profile → agent-specific overrides → in-game UI edits

**API Keys**: [`brain/.env`](brain/.env) (git-ignored)
- `ANTHROPIC_API_KEY=sk-ant-...` for real LLM decisions
- `CITYBRAIN_MOCK_LLM=1` for offline development

**In-Game Settings**: Persisted to `user://agent_llm.json` (highest precedence)
- Select agent → Inspector → LLM brain section
- Edit model, enabled state, tokens/day

## Coding Conventions

### GDScript (Godot)

- **File naming**: `snake_case.gd` for scripts, `PascalCase` for class names
- **Indentation**: Tabs (Godot default)
- **Signals**: Use `decision_needed.emit(agent, trigger)` pattern for event-driven logic
- **Determinism**: World logic in `_process()` with `SimClock.sim_delta()`; never block on sidecar

### Python (Sidecar)

- **File naming**: `snake_case.py` modules
- **Classes**: `PascalCase` for class names
- **Type hints**: Use `from __future__ import annotations` + full type hints
- **Async**: All I/O uses `asyncio`; avoid `time.sleep()` except in tests
- **Pydantic**: Models for all protocol messages (type safety + validation)

## Testing

### Godot

```bash
# Headless test run (no rendering)
godot --headless --path godot --quit-after 300

# With movie-writer frames (visual validation)
godot --path godot --write-movie /tmp/frames.png --quit-after 60
```

### Python

```bash
cd brain
uv run pytest -v
uv run pytest tests/test_budget.py  # Budget metering tests
uv run pytest tests/test_llm.py      # LLM integration tests
uv run pytest tests/test_server.py   # Decision routing tests
```

### End-to-End

1. Start sidecar: `cd brain && CITYBRAIN_MOCK_LLM=1 uv run python -m citybrain`
2. Run Godot: `godot --headless --path godot --quit-after 600 ++ --speed=30`
3. Check logs:
   ```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pufacz/citysimulation](https://github.com/pufacz/citysimulation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
