---
trigger: always_on
description: macLLM agents are built on top of smolagents `ToolCallingAgent`.
---

# Agent Architecture

## Overview

macLLM agents are built on top of smolagents `ToolCallingAgent`.

The agent layer adds:

- a registry of named agent classes
- managed-agent delegation for domain-specific work
- a shared base class for model, tool, and prompt wiring
- a thin factory used by conversation state
- token callbacks integrated with per-conversation metadata

## Managed Agents

Managed agents are regular `MacLLMAgent` subclasses used as delegated specialists.

This is the main architectural choice in the agent system. Top-level agents keep a small tool surface
and delegate specialized work, such as file or calendar tasks, to subagents with narrower instructions
and tool sets.

The parent lists subagents in `macllm_managed_agents`. The base class resolves those names from the registry
and registers **lazy** stand-ins (`LazyManagedMacLLMAgent` in `macllm/agents/lazy_managed.py`) with smolagents as
`managed_agents`. A real subagent instance is created only the **first time** the parent delegates to that
specialist (first `__call__`), not when the parent agent is constructed. Until then, `preload_skill` and full
subagent `__init__` work for that specialist do not run.

At runtime, a parent agent delegates by sending a natural-language task to the managed agent.
The subagent runs its own tool loop and returns a report. Delegation is recorded in
`agent.memory.steps` as `TaskStep` entries, which the UI can render.

### Registry and Factory

The registry lives in `macllm/agents/__init__.py`.

- `AGENT_REGISTRY` is the map from stable agent name to agent class
- `get_agent_class(name)` resolves a named agent
- `get_default_agent_class()` returns the default top-level agent

The factory lives in `macllm/core/agent_service.py`.

- `create_agent(...)` is the single runtime construction path used by conversation state

## Per-Agent Configuration

Each agent can be configured via `[agents.<name>]` sections in `config.toml`:

```toml
[agents.notes]
instructions = "You are a notes management assistant..."
skills = ["organize-notes", "format-markdown"]
```

- `instructions` -- custom instructions injected into the agent's system prompt. Overrides any
  code-level fallback. The project `config/config.toml` ships default instructions for all built-in
  agents. Users can override per-agent in `~/.config/macllm/config.toml`.
- `skills` -- list of skill names the agent can access via `read_skill`. When present, the agent
  receives a filtered skill catalog in its system prompt and `read_skill` is auto-added to its
  tool set. An empty or absent list means no skill access (unless the agent already has `read_skill`
  in its `macllm_tools`, in which case it sees all model-invocable skills).
- `preload_skill` -- name of a skill whose body is appended to the agent's instructions when
  that agent is constructed. For top-level agents this is at conversation agent creation; for **managed**
  subagents this happens when the subagent is **first materialized** (lazy), i.e. on first delegation.
  The skill content is baked into the system prompt, so it is always available as context
  without the agent needing to call `read_skill`. This is useful for user-specific preferences or
  conventions that should always apply (e.g. note formatting rules, calendar defaults). The skill
  is resolved via `SkillsRegistry.get()`, so even skills with `disable-model-invocation: true`
  can be preloaded.

These sections are optional. When absent, agents fall back to code-level custom instructions
(if any) and have no config-driven skill access or preloaded skills.

## Structure

The shared base class is `MacLLMAgent` in `macllm/agents/base.py`.

Its architectural contract is:

- agent identity is declared at the class level
- tool membership is declared symbolically and resolved centrally
- instructions and skill access are configured via `[agents.<name>]` in config.toml
- planning and token accounting are reported through shared callbacks
- managed-agent composition is declared by name, not by direct object wiring

Prompt behavior is split between prompt templates and custom instructions. Token accounting is reported
through `create_step_callback()`, which updates `conversation.llm_metadata`. Tool execution progress
is recorded by smolagents in `agent.memory.steps` and rendered by the UI from there.

Each agent’s system prompt includes **The user's current time & location** (`user_situation` in
`macllm/agents/prompts/default.yaml`): local date/time with IANA time zone and approximate GPS plus
reverse-geocoded place text when available. This is assembled in `get_device_context()` in
`macllm/core/device_context.py` and injected by `MacLLMAgent.initialize_system_prompt()` (not via a tool).

## Threading Model

Each conversation runs its agent on its own background thread. Multiple conversations can have
agents running simultaneously. The agent thread is set up with a thread-local conversation
context so that tools and step callbacks automatically route to the correct conversation.

Agents themselves are not aware of multi-threading. They run the same smolagents tool-calling

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [appenz/macLLM](https://github.com/appenz/macLLM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
