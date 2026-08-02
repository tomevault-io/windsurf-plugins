---
trigger: always_on
description: This file is auto-loaded by Claude Code in this repo. Keep it tight; longer prose belongs in `docs/`.
---

# CLAUDE.md — project memory for Repo2RLEnv

This file is auto-loaded by Claude Code in this repo. Keep it tight; longer prose belongs in `docs/`.

## What this is

**Repo2RLEnv** (`repo2rlenv` on PyPI) turns any GitHub repository into a verifiable RL training/eval dataset. End-to-end: **synthesis → standardize → train + eval**, focus on training. We emit datasets in the [Harbor](https://github.com/harbor-framework/harbor) task format so they drop straight into Harbor's runtime ecosystem (Local Docker / Modal / Daytona / E2B / Runloop + 22 agent harnesses).

GitHub: https://github.com/huggingface/Repo2RLEnv · PyPI: `repo2rlenv` · License: Apache-2.0.

## Architecture

Three layers, only the first is ours:

| Layer | We ship | We delegate |
|---|---|---|
| **Generation** (pipelines that produce tasks) | `src/repo2rlenv/pipelines/` — the moat | — |
| **Spec** (uniform output format) | `[metadata.repo2env]` extension to Harbor's `task.toml` | Harbor's task spec |
| **Consumption** (sandboxes / agents / runtime) | HF Hub publish bridge; planned TRL trainer bridge | Harbor's full stack |

## Where things live

```
src/repo2rlenv/
├── spec/                       # Pydantic input + output models (the contract)
├── pipelines/
│   ├── base.py                 # Pipeline Protocol + PipelineResult
│   ├── pr_diff.py              # SHIPPED — PR-diff mining; text-only gen, Docker-runnable env (6-component verifier)
│   ├── _pr_diff_verifier.py    # in-container 6-component diff-similarity reward (pure stdlib, base64-baked)
│   ├── _eval_script.py         # shared verifier-script + diff helpers (code_instruct, equivalence_tests)
│   ├── _env_guard.py           # anti-contamination: git-history scrub + egress-guard compose (all runtime pipelines)
│   └── _poc_agent.py           # agentic PoC-test synthesis for cve_patches (LLM + shell in the vuln sandbox)
├── bootstrap/                  # v0.2 — LLM-driven Docker env generation
│   ├── runner.py               # ensure_bootstrap() orchestrator
│   ├── agent.py                # ReAct loop
│   ├── docker.py               # DockerSandbox primitives
│   ├── language.py             # auto-detect Python/JS/Go/Rust/...
│   └── cache.py                # filesystem cache under ./envs/ (keyed on opts)
├── ui/                         # Unified Rich UI module (every CLI uses this)
│   ├── console.py              # singleton R2EConsole + install_logging()
│   ├── theme.py                # one place for colors + glyphs
│   ├── primitives.py           # success_panel, error_panel, kv_panel, ...
│   ├── live.py                 # live_view() context manager
│   └── views/
│       ├── bootstrap.py        # BootstrapView (split-panel live display)
│       └── generation.py       # GenerationView (progress bar + stats)
├── emitter/harbor.py           # Task → Harbor task.toml directory writer
├── hub.py                      # push to HF Hub + Harbor-compatible registry.json
├── reward.py                   # SWE-RL-style diff-similarity reward (stdlib only)
├── llm.py                      # LiteLLM wrapper + LiteLLM completion_cost tracking
├── github.py                   # `gh` CLI wrapper for PR list + diff fetch
├── auth.py                     # gh CLI → env var token resolution
├── config.py                   # YAML/TOML config loader
└── cli.py                      # argparse entry points

docs/                           # public docs (committed), three tiers:
├── README.md                   #   index
├── quickstart.md               #   install → first dataset → push, 10 min
├── reference/                  #   stable contracts + module-level API
│   └── SPEC.md · API.md · AUTH.md · BOOTSTRAP.md · AGENTS.md
├── pipelines/                  #   one page per synthesis pipeline
│   └── README.md · pr_diff.md · pr_runtime.md · ...
└── contributing/
    └── ADDING_A_PIPELINE.md    #   cookbook for shipping a new pipeline

plans/                          # internal working docs (gitignored)
references/                     # cloned inspiration repos (gitignored)
envs/, envs-*/, .r2e_cache/     # local artifacts (gitignored)
tests/                          # pytest; 620/620 pass as of v0.8.2.post3
.github/workflows/              # ci.yml (lint + matrix tests + build),
                                # release.yml (PyPI publish on tagged release)
CONTRIBUTING.md                 # dev setup, PR conventions, release flow
```

## Pipeline contract

Every synthesis pipeline implements `repo2rlenv.pipelines.base.Pipeline`:

```python
class Pipeline(Protocol):
    name: ClassVar[PipelineName]
    def __init__(self, input: GenerationInput, options: BaseModel) -> None: ...
    def run(self, out_dir: Path) -> PipelineResult: ...
```

The conformance test (`tests/test_pipeline_contract.py`) fails for any registered pipeline that doesn't conform. **Always run** `uv run pytest tests/test_pipeline_contract.py` after touching anything in `pipelines/`.

Adding a new pipeline: see [`docs/pipelines/ADDING_A_PIPELINE.md`](./docs/pipelines/ADDING_A_PIPELINE.md).

## UI conventions (use the shared module, not raw prints)

**Never use `print()`** in CLI code. Always:

```python
from repo2rlenv.ui import console

console.success("emitted task X")          # ✓ green

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [huggingface/Repo2RLEnv](https://github.com/huggingface/Repo2RLEnv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
