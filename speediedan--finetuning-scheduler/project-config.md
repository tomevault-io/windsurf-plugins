---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Fine-Tuning Scheduler (FTS) is a PyTorch Lightning callback for multi-phase, scheduled fine-tuning.

## Environment

Development uses **traditional venvs built by a repo script**, not `uv venv` or `uv run` directly.
Activate the venv and invoke `python`/`pytest` normally.

```bash
export FTS_VENV_BASE=/mnt/cache/${USER}/.venvs
export FTS_TARGET_VENV=fts_latest
export FTS_REPO_DIR=${HOME}/repos/finetuning-scheduler

./scripts/build_fts_env.sh --repo-home=${PWD} --target-env-name=fts_latest --venv-dir=${FTS_VENV_BASE}
source ${FTS_VENV_BASE}/${FTS_TARGET_VENV}/bin/activate
```

Key `build_fts_env.sh` flags: `--oldest`, `--no-commit-pin`, `--venv-dir`, `--torch-backend`,
`--from-source="lightning:${HOME}/repos/lightning:pytorch"`, `--uv-install-flags`, `--dry-run`.

Venv placement matters: uv hardlinks only work within the same filesystem as the uv cache, hence
`--venv-dir=/mnt/cache/${USER}/.venvs` rather than a home-directory venv.

Manual install into an existing env:

```bash
export UV_OVERRIDE=${PWD}/requirements/ci/overrides.txt
uv pip install -e ".[all]"
```

`requirements/` holds CI and docs pins only — there is no top-level `requirements.txt`.
`requirements/ci/requirements.txt` and `requirements-oldest.txt` are **generated**; regenerate with
`./requirements/utils/lock_ci_requirements.sh`, never hand-edit.

## Testing

```bash
python -m pytest src/finetuning_scheduler tests -v          # src/ IS a test target (--doctest-modules)
python -m pytest tests/test_fsdp.py::test_name -v --capture=no
python -m coverage run --source src/finetuning_scheduler -m pytest src/finetuning_scheduler tests -v
```

`src/finetuning_scheduler` must be passed alongside `tests` — `--doctest-modules` is in `addopts`, so
docstring examples run as tests.

**Plain pytest does not run the standalone or experimental-patch tests.** Those need
`tests/special_tests.sh`, which selects on `@RunIf(...)`-generated `skipif` marker kwargs (there are no
custom pytest markers despite `--strict-markers`):

```bash
./tests/special_tests.sh                                                   # defaults to --mark_type=standalone
./tests/special_tests.sh --mark_type=standalone --filter_pattern='test_f'
./tests/special_tests.sh --mark_type=standalone --collect_dir='src/fts_examples' --filter_pattern='model_parallel_examples'
./tests/special_tests.sh --mark_type=exp_patch --filter_pattern='test_f' --experiment_patch_mask="1 0 0 1"
```

Other flags: `--log_file`, `--log-dir`, `--experiments_list`, `--allow-failures`. The
`--experiment_patch_mask` bit order follows `tests/.experiments`. To run one standalone test directly:
`PL_RUN_STANDALONE_TESTS=1 python -m pytest tests/test_x.py::test_y -v`.

`RunIf` conditions live in `tests/helpers/runif.py` (`min_cuda_gpus`, `standalone`, `bf16_cuda`,
`exp_patch`; aliases `alone`, `bf16_alone`). CUDA-marked tests gate on `PL_RUN_CUDA_TESTS=1`.

When torch or Lightning leaks a new env var, add it to the allowlist in the `restore_env_variables`
autouse fixture in `tests/conftest.py` rather than working around the failure.

Full local coverage (~30 min) is orchestrated by `scripts/gen_fts_coverage.sh`; wrap long multi-GPU runs
in `scripts/manage_standalone_processes.sh --use-nohup` (VS Code kills plain nohup jobs).

## Serializing GPU work on a shared host

A multi-GPU host is often shared by more consumers than it looks: several interactive/agent sessions in
this repo, sessions in a sibling project (`interpretune` shares this project's host and self-hosted CI
agent), and the self-hosted Azure Pipelines agent, which can dispatch a GPU job as soon as one is approved.
Two GPU suites at once contend **silently** — OOM, flaky timing, or mutual slowdown rather than an obvious
error — so it is easily misdiagnosed as a real test failure.

`scripts/gpu_lease_wrap.sh` provides an **opt-in** `flock`-based lease. It is a **complete no-op unless
`GPU_LEASE_CMD` points at a lease implementation**, so contributors and hosted CI are unaffected and nothing
here is required to work on this project.

```bash
export GPU_LEASE_CMD=/path/to/gpu_lease.sh        # opt in for this shell
./tests/special_tests.sh --mark_type=standalone   # now serialized
```

`tests/special_tests.sh` and `scripts/gen_fts_coverage.sh` self re-exec under the lease, so one acquisition covers a
whole suite. The Azure GPU pipeline participates too, by bind-mounting the lease directory into the job
container (`flock` works on the inode, so container and host processes interlock).

Two things worth knowing up front:

- **Waiting is normal, not a failure.** A run may sit at `'gpu' lease is held; waiting...` for as long as
  whatever holds it (a CI job is ~37 min). Let it queue — do not disable the lease or kill the holder.
- **Notebooks and other interactive GPU work are not lease-aware**, by design: a Jupyter kernel lives for
  hours, so holding the lease for its lifetime would starve CI. Check `--status` before starting one, and
  for a long session reserve deliberately with `--hold` / `--release`. The lease warns whoever acquires it
  next that unleased processes are on the GPUs.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [speediedan/finetuning-scheduler](https://github.com/speediedan/finetuning-scheduler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
