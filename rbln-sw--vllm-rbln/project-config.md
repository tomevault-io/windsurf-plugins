---
trigger: always_on
description: vllm-rbln is a vLLM platform plugin for Rebellions NPUs. Source lives in `vllm_rbln/`, tests in `tests/`.
---

# Agent instructions for vllm-rbln

vllm-rbln is a vLLM platform plugin for Rebellions NPUs. Source lives in `vllm_rbln/`, tests in `tests/`.

Two documents own their subjects; read them rather than duplicating them here:

- Environment setup, dependency and lockfile policy: `DEVELOPMENT.md`
- PR process and issue labels: `CONTRIBUTING.md`

## Accountability

A pure agent-authored change is not acceptable. Upstream vLLM states the same rule in its own `AGENTS.md`. The submitting human reads every changed line, runs the tests, and can defend the change without the agent. If you cannot say why a line is there, delete it rather than ship it.

## Commands

Run everything through `uv`. Never use the system `python3` or a bare `pip`.

```bash
uv run --no-sync pytest tests/vllm/v1/worker/test_rbln_worker.py::<one test> -x
uvx pre-commit run --files <every file you changed>
```

Use the narrowest test target that covers the change. Start by running the specific affected test, such as `test_file.py::test_name`, and widen to the full test file only when necessary. Do not start by running an entire test directory.

Run `pre-commit` through `uvx`, since it is not a project dependency. Pass `--files` explicitly to avoid depending on the current staging state.

A new `.py` file needs the Apache header that every other file carries; `check-license-header` rejects it otherwise. Copy the header from a neighbouring file.

`tests/vllm/` defines three session options (see its `conftest.py`):

- `--model-compile` — opt into whole-model compiles, minutes per test
- `--device-tensor {0,1}` — session-wide, cannot be parametrized per test
- `--num-hidden-layers N` — build only the first N decoder layers

They do not exist in `tests/optimum/`.

## Terms

The codebase already has a word for each of these. Use it, and do not reach for a synonym because the sentence reads better.

- **model impl path**, short form **model path** — which model implementation runs. The two are the **vllm model path** and the **optimum model path**, each named after the implementation it selects, and that name is the word in prose and in identifiers alike: `tests/vllm/` and `tests/optimum/`, `platform/vllm_impl.py` and `platform/optimum_impl.py`. Not "native", which used to be the prose form for the vllm model path and reads as a second concept next to it. Not "backend", not "mode". `torch.compile` describes how the vllm model path works and is not its name.
- **suite** — a top-level test tree: `tests/vllm/`, `tests/optimum/`.
- **lane** — a slice of a suite selected by a flag, a mark, or the device mode: the default lane, the `--model-compile` lane, the cpu and device lanes.

## Two model paths

Upstream's `--model-impl` selects the model path. `vllm` is the vllm model path, `transformers` and `optimum` are the optimum one, and `auto` (the default) leaves it to `resolve_model_impl()`, which takes the path this process was handed, and otherwise the one the model asks for: optimum where optimum-rbln implements the architecture, and vllm where it does not. Anything else is refused. `resolve_model_impl()` reads it before the config exists, and `create_engine_config` hands the field back its default so upstream's own resolution is untouched. A built `additional_config` names the path by being one of the two classes, and disagreeing with the flag is refused. `VLLM_RBLN_USE_VLLM_MODEL=1` still means `vllm`, warns, and goes away in 0.14.0.

| Path         | Owns                                                                        |
| ------------ | --------------------------------------------------------------------------- |
| optimum      | `model_executor/models/optimum/`, `utils/optimum/`, `v1/worker/optimum_*.py`, `platform/optimum_impl.py` |
| vllm         | `patches/`, `compilation/`, `v1/worker/rbln_*.py`, `platform/vllm_impl.py` |
| shared       | everything else, including `config.py`, which holds both paths' config classes |

**`config.py` defines the selector; only `__init__.py` and `platform/__init__.py` branch on it.** Do not branch on the model path anywhere else. Path-specific code belongs in the module that path owns.

- Say which path or paths you changed in the PR description.
- A change to one path must not alter the other. If it appears to need both, stop and ask before writing code.
- A new env var goes in three places in `envs.py`: the `TYPE_CHECKING` block, the `environment_variables` dict, and either `RBLN_COMPILE_ENV` or `RBLN_NON_COMPILE_ENV`. The two sets partition the mega-cache bundle key, and `test_mega_cache.py` asserts they cover every variable.
- Suites carry the path: `tests/vllm/` adopts `vllm` in its conftest and scrubs `VLLM_RBLN_*`; `tests/optimum/` has no suite-level conftest, so the tests there that build an engine name `optimum` themselves. An exported `VLLM_RBLN_USE_VLLM_MODEL` therefore changes what the rest of `tests/optimum/` exercises without failing.
- Do not select the model path inside a test to escape its suite.

## Patching upstream vLLM

`vllm_rbln/patches/` adapts upstream vLLM for RBLN. Both mechanisms live in `patches/registry.py`, both take a required `reason`, and only the vllm model path applies them. Registrations go first, then patches.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RBLN-SW/vllm-rbln](https://github.com/RBLN-SW/vllm-rbln) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
