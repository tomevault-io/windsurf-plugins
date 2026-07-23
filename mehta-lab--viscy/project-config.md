---
trigger: always_on
description: VisCy is a **uv workspace monorepo** for virtual staining and computational microscopy. Sub-packages live under `packages/`.
---

# VisCy — Claude Code Reference

## Project

VisCy is a **uv workspace monorepo** for virtual staining and computational microscopy. Sub-packages live under `packages/`.

## Repo Layout

```
pyproject.toml              # Root config (ruff, pytest, uv workspace)
packages/
  viscy-data/               # Data loading and Lightning DataModules
  viscy-models/             # Neural network architectures
  viscy-transforms/         # Image transforms
src/viscy/                  # Umbrella package (re-exports)
applications/               # Self-contained research applications
```

### Packages vs Applications

- **Shared code belongs in `packages/`**, not in applications.
- **Applications must not import from each other.** If two applications need the same logic, move it to an existing package or create a new one.
- Applications are consumers of packages — the dependency graph always flows `applications/ → packages/`, never sideways.

---

## Development

### Environment Setup

Use `uv` package manager. Run commands with `uv run <command>`. Edit `pyproject.toml` to modify dependencies and sync to update `uv.lock`.

```sh
uv venv -p 3.13
uv sync --all-packages --all-extras
```

If `uv` is not installed:
```sh
curl -LsSf https://astral.sh/uv/install.sh | sh
```

On HPC, symlink the uv cache out of your home directory first:
```sh
mkdir -p /hpc/mydata/firstname.lastname/.cache/uv && ln -s /hpc/mydata/firstname.lastname/.cache/uv ~/.cache/uv
```

For full setup instructions (installing uv, creating a venv, syncing dependencies), see [CONTRIBUTING.md](./CONTRIBUTING.md).

### SLURM scripts for Lightning DDP jobs

When hand-writing `.slurm` scripts that launch Lightning via `srun`, always use `--ntasks-per-node=N` (not `--ntasks=N`). Lightning's `SLURMEnvironment` validates `SLURM_NTASKS_PER_NODE` at trainer init and raises `RuntimeError: You set --ntasks=N in your SLURM bash script, but this variable is not supported. HINT: Use --ntasks-per-node=N instead.` — the job then dies seconds into the allocation.

Invariant: `#SBATCH --ntasks-per-node=N` must equal `trainer.devices` in the YAML config and `#SBATCH --gpus=N` (single-node) or `#SBATCH --gpus-per-node=N` (multi-node).

The dynacell launcher (`applications/dynacell/tools/submit_benchmark_job.py`) already emits `--ntasks-per-node` correctly; this note is for hand-written scripts (e.g., `applications/cytoland/examples/configs/*/run_*.slurm`).

### Job monitoring and inspection

**Process state ≠ training completeness.** Wandb's `state: finished` only means `wandb.finish()` was called — Lightning calls it on clean SIGTERM teardown via `SLURMEnvironment`, so a `scancel`'d run shows `finished` identically to one that hit `max_epochs`. Always cross-check.

**Liveness check (is the job alive *right now*?):** `wandb.Api().run(...).heartbeatAt` is authoritative. Do not infer liveness from `last.ckpt` mtime, internal step counter, or a single `nvidia-smi` snapshot.

**Completeness check (did the job finish its goal?):** combine three sources, all required:
1. `sacct -j <job_id> --format=JobID,State,ExitCode,Elapsed,TimeLimit` — `CANCELLED+` with `ExitCode 0:0` is the signature of a user `scancel`; `TIMEOUT` is wall-time hit; `COMPLETED` with ExitCode 0:0 is the only unambiguous success.
2. The resolved fit YAML at `/hpc/projects/comp.micro/virtual_staining/models/dynacell/.../resolved/fit_*_<timestamp>.yml` — read `trainer.max_epochs` / `trainer.max_steps`. The wandb `r.config` dict only stores model init args, **not trainer args**.
3. Wandb run summary — compare final `epoch` to `trainer.max_epochs`. Final epoch of e.g. `135/200` is killed mid-training, not done; `200/200` is the only credible success indicator.

The `output.log` / `wandb-output.log` for the run will contain `Received SIGTERM: 15` if Lightning's signal handler caught a scancel — a useful confirmation when sacct is ambiguous.

**Before cancelling jobs:** the job name in `squeue` is not a complete description. `FCMAE_VSCyto3D_Pretrained_A549_Membrane` could be a fit run OR a predict run — they share the trained-model directory naming. Verify the actual purpose via:
- The `Comment` field (`squeue -j <id> -o "%k"`) if the launcher set one
- The resolved YAML path (fit vs predict subdirectory)
- The wandb run config for that job ID

When the user says "cancel all jobs," scope it to **batch jobs only**, never the interactive nomachine session. Read job names carefully — a job that has been alive for >24 h on a multi-GPU allocation is almost certainly training, not a predict run that should be ~hours.

**Subagent prompts for job status:** ask for completeness vs config, not just liveness. A prompt like "check the liveness of wandb run X" returns `state: finished` for a SIGTERM'd run and reads as success. Phrase it as "is run X complete relative to its configured `max_epochs`, and what was the exit reason (clean finish, scancel, OOM, timeout, exception)?"

### Joint vs single-set training batch semantics

`HCSDataModule` and `BatchedConcatDataModule` produce the same number of GPU samples per training step — but the YAML `batch_size` value that gets there is **different by a factor of `num_samples`**. Easy to misread either by skimming.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mehta-lab/VisCy](https://github.com/mehta-lab/VisCy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
