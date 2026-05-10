---
trigger: always_on
description: This repository hosts first-party environments that build on the [`verifiers`](https://github.com/primeintellect-ai/verifiers) framework, contributed by community members as well as the Prime Intellect team. Follow the guidance below when implementing a new environment, iterating locally, and preparing a pull request. These instructions apply across the entire repo.
---

# AGENTS.md

This repository hosts first-party environments that build on the [`verifiers`](https://github.com/primeintellect-ai/verifiers) framework, contributed by community members as well as the Prime Intellect team. Follow the guidance below when implementing a new environment, iterating locally, and preparing a pull request. These instructions apply across the entire repo.

## Local Setup

Clone repo and prepare workspace: 
```bash
# install git-lfs
sudo apt update && sudo apt install git-lfs

# clone repo
git clone git@github.com:PrimeIntellect-ai/community-environments.git
cd community-environments

# install uv
curl -LsSf https://astral.sh/uv/install.sh | sh
uv sync

# dev hooks
uv run pre-commit install

# prime-cli auth -- https://docs.primeintellect.ai/api-reference/api-keys
uv tool install prime 
prime config set-api-key <api-key> 
```

Creating a new environment:
```bash
# creates template at ./environments/<env_slug>/
prime env init <env-slug>
uv run vf-install <env-slug>
```

Running evaluations:
```bash
uv run vf-eval -s <env-slug> -m <model> # e.g. gpt-4.1, gpt-5-mini
```

Declare any extra dependencies directly your the environment's `pyproject.toml` (`./environments/<env_slug>/pyproject.toml`).
  
When pinning Git sources, use `tool.uv.sources`:
```toml
[tool.uv.sources]
tau2 = {git = "https://github.com/sierra-research/tau2-bench.git"}
```

## Coding Principles

### Style & Structure

- Format with `uv run ruff check --fix .` and keep imports tidy before sending a PR.
- Provide explicit type annotations for core functionality when relevant (though you may want to skip for reward functions).
- Validate typing with `uvx ty check ./environments/<slug>`.
- Avoid `# type: ignore` unless an interface truly requires it (if so, add a comment explaining why).
- Organize helpers into private modules once an environment grows beyond a
  single file; keep `__init__.py` minimal.

### Error Handling

- Fail fast with clear exceptions when credentials, datasets, or required
  tooling are missing.
- Keep branching narrow and well-commented. Prefer explicit validation over
  defensive defaults that hide misconfiguration.

### Documentation

- Each environment **must** include a concise `README.md` covering setup,
  dependencies, reward rubric, configuration knobs, and sample `vf-eval` usage.
- Update documentation whenever inputs, outputs, or required credentials change.

## Environment Implementation

### Canonical Layout

Each environment lives in `environments/<slug>/` and should contain:

- A `pyproject.toml` which includes `verifiers` and other required dependencies.
- A module that implements a `load_environment(...)` entrypoint which returns the configured environment.
- Optional assets (prompts, templates, datasets) colocated with the module.

Use subpackages (`environments/<slug>/utils/`) when code spans multiple files.

### Choosing Base Classes

Leverage the primitives provided by `verifiers.envs`:

| Pattern | Base Class | Use When | Notes |
| --- | --- | --- | --- |
| Single turn | `SingleTurnEnv` | Classic Q&A or scoring tasks | Supply dataset loading and reward helpers. |
| Multi turn | `MultiTurnEnv` | Conversational or iterative flows | Override `env_response`, `is_completed`, and manage turn state. |
| Stateless tools | `ToolEnv` | Python function tool invocation | Document tool side effects in the README. |
| Stateful tools | `StatefulToolEnv` | Tools needing persistent state or arg injection | Use `setup_state` / `update_tool_args` to track context. |
| MCP bridges | `MCPEnv` | Surfacing MCP servers | Follow the usage patterns in `environments/mcp_env` in the `verifiers` [repo](https://github.com/primeintellect-ai/verifiers). |

In general, you should never override `rollout()` directly. Instead, override into the hooks that the base classes
expose (`setup_state`, `env_response`, `is_completed`, reward functions, etc.).

### Rubrics & Rewards

- Define a `Rubric` that includes each reward function or metric directly; document rewards/metrics in the environment README.
- Reward functions may refer to `prompt`, `completion`, `answer`, `state`, `task`, and `info`, as well as any other objects specified in the `Rubric.class_objects` dict (e.g. `parser`).
- Keep reward functions self-contained; use `class_objects` and `state` for message-passing and helpers as needed.
- Reward functions should be passed to a rubric via `funcs=[first_reward,...]`  (optionally, with `weights=[wt1,...]`).
- Use 0-weighted reward functions for purely informational metrics.

### Configuration & IO

- Accept function arguments to `load_environment` only for essential customization (e.g. key hyperparameters such as `max_turns`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PrimeIntellect-ai/community-environments](https://github.com/PrimeIntellect-ai/community-environments) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
