---
trigger: always_on
description: This file is a contributor guide for LLM coding agents working in this repository.
---

# AGENTS.md

This file is a contributor guide for LLM coding agents working in this repository.

## 1) What This Repository Is

Silisocs is a native social simulation framework with an optional Concordia
compatibility bridge for legacy scenarios. It has:

- YAML-first scenario and runtime configuration (Hydra + OmegaConf)
- A social-media game-master/environment layer
- Multiple platform backends (Twitter-like, Reddit-like, Mastodon)
- Declarative persona pipeline plus custom builder extension path
- Probe-based evaluation and rich runtime telemetry
- Streamlit dashboard for scenario creation and launch

The runtime entrypoint is:

- `src/silisocs/runtime/runner.py`

## 2) High-Level Architecture

Core runtime layers:

### 1. Agent Construction Layer
- `src/silisocs/runtime/construction/agent_builders/`
- Builds agent construction specs from `agents.persona_pipeline` and class data sources
- Supports fixed-action set loading and template rendering
- Entry point: `AgentBuilder.build_agent_configs()`

### 2. Agent Runtime Layer
- `src/silisocs/agents/base_agent.py` — Abstract Agent interface
- `src/silisocs/agents/native.py` — default LLM-backed native agent
- `src/silisocs/agents/fixed.py` — deterministic fixed-action agent
- Custom agents subclass `Agent`, accept a `LanguageModel`, implement
  `name`, `observe(str)`, and `act(ActionSpec) -> ActionOutput`
- To add a custom agent, point `persona_pipeline.classes.*.class_path` at the
  runtime class and provide strict constructor `params`

### 3. Game Master Layer (Component-Slotted Architecture)
- `src/silisocs/environments/gm/base_game_master.py` — Base coordinator
- `src/silisocs/environments/gm/game_master.py` — ComponentGameMaster and MultiFlowGameMaster
- `src/silisocs/environments/gm/components/` — Pluggable components:
  - `next_acting.py` — Determine which agent acts next
  - `observe.py` — Generate timeline/episode observations
  - `resolve.py` — Parse agent output into backend actions
  - `app_update.py` — Schedule backend/recommendation updates
- To add custom component: implement `Component` interface, set in `env.gm.components.{role}.class_path`

### 4. Engine Layer (Execution Policies)
- `src/silisocs/simulation_engines/base_engines.py` — BaseRuntimeEngine, FlowRuntimeEngine (multi-flow scheduling)
- `src/silisocs/simulation_engines/multi_gm.py` — MultiGMRuntimeEngine (multi-GM orchestration)
- `src/silisocs/simulation_engines/policies/` — loop, step, and turn policies:
  - Turn policy: `single_action`, `fixed_count`, `open_ended`
  - Step policy: `base`, `sequential`, `flow`, `multi_gm`
  - Loop policy: default episode loop
- To add custom policy: implement the relevant policy ABC and reference it via `class_path`

### 5. Backend Action Layer
- `src/silisocs/environments/backends/base.py` — ActionCatalog, base app interface
- `src/silisocs/environments/backends/twitter_like/` — TwitterLikeApp with SQL backend
- `src/silisocs/environments/backends/reddit_like/` — RedditLikeApp
- `src/silisocs/environments/backends/mastodon/` — Real Mastodon server integration
- Actions discovered via `@app_action(name=..., description=...)` decorator
- To add custom backend: subclass `SocialBackendApp`, implement action methods, register in app factory

### 6. Runtime Orchestration
- `src/silisocs/runtime/runner.py` — CLI entrypoint, Hydra config composition
- `src/silisocs/runtime/config.py` — Config validation and initialization
- Handles: model creation, direct runtime construction, initialization,
  simulation execution, checkpoint save/resume

## 3) Configuration Model

Top-level config composition (`src/silisocs/conf/experiment.yaml`):

- Defaults: `world: default`, `agents: default`, `sim: base`, `env: twitter_like`, `eval: base`

Config groups and their base files:

| Group | Base file | Controls |
|-------|-----------|----------|
| `world` | `world/default.yaml` (`@package _global_`) | Run params, setting, event, data |
| `agents` | `agents/default.yaml` (`@package agents`) | Persona pipeline, shared memories |
| `sim` | `sim/base.yaml` (`@package sim`) | LLM, engine, tool-calling, memory, checkpoint |
| `env` | `env/twitter_like.yaml` (`@package env`) | Backend, GM components, initialization |
| `eval` | `eval/base.yaml` (`@package eval`) | Probe configuration |

Key sim knobs (`src/silisocs/conf/sim/base.yaml`):

| Parameter | Value | Notes |
|-----------|-------|-------|
| `sim.llm.name` | gpt-4o-mini | Default LLM model |
| `sim.llm.temperature` | 0.5 | Sampling temperature |
| `sim.llm.disabled` | false | No-op model for testing |
| `sim.action_mode` | custom | Prompt style (`custom` or `generic`) |
| `sim.tool_calling.mode` | single | `none` \| `single` \| `multi` |
| `sim.engine.step.built_in` | base | `base`, `sequential`, `flow`, or `multi_gm` |
| `sim.engine.turn_policy.built_in` | single_action | `single_action` \| `fixed_count` \| `open_ended` |
| `sim.checkpoint.every_n_steps` | null | Checkpoint frequency (run_study.py sets 1 by default) |

Key run params live in `world/default.yaml` (at config root via `@package _global_`):

| Parameter | Default | Notes |
|-----------|---------|-------|
| `num_agents` | 100 | Number of agents |
| `num_steps` | 50 | Simulation episodes |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sandbox-social/silisocs](https://github.com/sandbox-social/silisocs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
