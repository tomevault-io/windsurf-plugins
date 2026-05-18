---
trigger: always_on
description: <!-- Go-to brief for AI coding agents working on AReaL. -->
---

<!-- Go-to brief for AI coding agents working on AReaL. -->

# AGENTS.md -- AReaL Agent Operations Guide

## Quick reference

**Tech stack**: Python 3.12+ | PyTorch | FSDP2 / Megatron / Archon | SGLang / vLLM

```bash
# Environment
uv sync --extra cuda            # CUDA + SGLang inference (default); for vLLM: cp pyproject.vllm.toml pyproject.toml && cp uv.vllm.lock uv.lock && uv sync --extra cuda
uv sync --extra sandbox         # Daytona cloud sandbox backend (optional)
source .venv/bin/activate        # activate venv BEFORE pre-commit or git commit if venv exists
pre-commit install --install-hooks  # hooks: Ruff, clang-format, mdformat, nbstripout, conventional-commits
pre-commit run --all-files       # lint + format everything

# Tests
uv run pytest tests/test_<topic>.py

# CLI docs
uv run python docs/generate_cli_docs.py

# Docs build (canonical, release-aligned)
./docs/build_all.sh
# Do NOT use `jupyter-book build docs/en|docs/zh` directly for final preview/release,
# because it skips AReaL-specific static setup and output packaging.
```

**Hard rules** -- never violate:

- No wildcard imports (`from x import *`).
- No hardcoded secrets, paths, or endpoints.
- No skipping pre-commit hooks.
- No guessing cluster configs or rebuilding CUDA/driver stacks.
- Integration tests require multi-node hardware -- explain skips explicitly.

**Always do**:

- Read relevant files before modifying code.
- Run `pre-commit run --all-files` before committing.
- Follow existing code patterns in the same module.
- Add tests for new functionality.
- Ask for decisions and clarifications with short, structured options instead of broad
  open-ended questions. Use the platform's native question/clarification tool if
  available.

**Ask first** before:

- Modifying config structures in `areal/api/cli_args.py`.
- Adding new dependencies.
- Changing launcher or scheduler logic.
- Deleting or renaming public APIs.

When unsure, leave a `TODO(agent)` comment and note the constraint in your response.

______________________________________________________________________

## Repository map

```
areal/                     Core Python package
|-- api/                   Config dataclasses, contracts, IO structs
|-- dataset/               Stateful dataset loaders (GSM8K, Geometry3K, CLEVR, ...)
|-- engine/                Training backends (FSDP2, Megatron) + inference adapters
|-- experimental/          Prototype engines/workflows (Archon MoE engine)
|-- infra/                 Launchers (Local/Ray/Slurm), schedulers, utilities
|-- models/                Model adapters (Megatron-Core, Transformers, custom heads)
|-- reward/                Built-in reward functions + math parsers
|-- tests/                 Unit/integration test suites
|-- trainer/               High-level orchestrators (PPOTrainer, SFTTrainer)
|-- utils/                 Cross-cutting helpers (logging, data, checkpoints, RL ops)
+-- workflow/              RolloutWorkflow implementations (RLVR, multi-turn, vision)

docs/                      Jupyter Book docs (https://areal-project.github.io/AReaL/)
examples/                  Training scripts and launcher recipes
```

______________________________________________________________________

## Code style & patterns

- **Composition over inheritance** -- keep hierarchies \<= 2 levels; prefer delegation.

| Type             | Pattern         | Example                                   |
| ---------------- | --------------- | ----------------------------------------- |
| Config dataclass | `XxxConfig`     | `GRPOConfig`, `FSDPEngineConfig`          |
| Engine class     | `XxxEngine`     | `FSDPEngine`, `ArchonEngine`              |
| Workflow class   | `XxxWorkflow`   | `RLVRWorkflow`, `MultiTurnWorkflow`       |
| Reward function  | `xxx_reward_fn` | `gsm8k_reward_fn`, `geometry3k_reward_fn` |

**Logging**: `areal.utils.logging.getLogger(name)` with **PascalCase** names -- never
`print` or `logging.__name__`. Per-rank format: `[{Component} Rank {N}]`. Register new
loggers with color in `areal/utils/logging.py`.

**Performance**:

- No GPU-CPU sync in hot paths (`.item()`, `.tolist()`, `print(tensor)`).
- Batch ops over Python loops on tensor elements.
- Explicit `dtype`/`device`; `torch.Size` assertions for shape validation.

**Typing & imports**: explicit type hints; reuse `areal/api/cli_args.py` dataclasses; no
wildcard imports; heavy optional deps inside functions.

**Async**: rollout workflows must stay non-blocking (`await` + `aiofiles`); no sync I/O
in `arun_episode`.

______________________________________________________________________

## Domain experts & skills

Fire the appropriate **expert subagent** or **load a skill** based on what you're
working on. Experts are read-only consultants with deep domain knowledge; skills are
step-by-step implementation guides.

| Working on...                | Fire subagent      | Load skill          |
| ---------------------------- | ------------------ | ------------------- |
| FSDP engine code             | `fsdp-expert`      | --                  |
| Archon engine / new model    | `archon-expert`    | `add-archon-model`  |
| Megatron engine code         | `megatron-expert`  | --                  |
| RL algorithms / PPO / GRPO   | `algorithm-expert` | --                  |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [areal-project/AReaL](https://github.com/areal-project/AReaL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
