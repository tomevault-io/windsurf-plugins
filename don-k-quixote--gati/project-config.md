---
trigger: always_on
description: This file documents how AI coding assistants should work on the GATI repo.
---

# AGENTS.md

This file documents how AI coding assistants should work on the GATI repo.
Read it fully at the start of every session.

---

## What GATI is

GATI (Green-time Allocation for Traffic Intersections) is a 4-week applied
research project at IIM-Lucknow on RL-based adaptive traffic signal control,
with two angles: heterogeneous Indian traffic conditions, and emergency
vehicle priority. The deliverable is either a course submission or a small
workshop paper (decision pending).

The project is currently in Week 3 of 4. Team of 8: Khalid (RL lead, primary
user), Ninad (RL collaborator), Vatsal/Vivek (env wrappers, demand modeling),
Aditya/Prateek/Rahul (SUMO env), Kushagra (PM).

The contract that defines scope, observation/reward semantics, and evaluation
protocol lives at `docs/Traffic-Signal-RL-Contract.docx`. Treat it as
authoritative for what the system is supposed to do. The contract has been
amended three times during Week 2 (pressure-based reward, lane segmentation,
tanh wait normalization); a team meeting to ratify these is pending. **Do
not propose contract amendments without explicit user approval.**

## Stack

- WSL2 Ubuntu, Python 3.11.x in conda env `gati`
- PyTorch 2.11.0+cu126 on RTX 4060 Laptop GPU (8GB VRAM)
- Stable-Baselines3 2.8.0, Gymnasium 1.2.3, sumo-rl 1.4.5
- SUMO 1.26.0 system-wide at `/usr/share/sumo`, `SUMO_HOME` set
- wandb 0.26.1 (online auth required for cloud sync; offline mode works without)

Per-session sanity check before substantive work:

```bash
cd ~/projects/GATI && conda activate gati && \
  echo $SUMO_HOME && \
  python -c "import sumo_rl; import torch; print('OK', torch.cuda.is_available())" && \
  pytest tests/ -q && \
  sumo -c nets/cross_indian_test/cross_indian_test.sumocfg --no-step-log --duration-log.disable
```

Expected: `/usr/share/sumo`, `OK True`, `15 passed`, silent SUMO completion.

## Repo layout

```
configs/default.yaml              # Single source of truth for experiment params
docs/                             # Contract (.docx), literature review, work logs (gitignored)
envs/observations.py              # IndianContextObservation (per-signal observation)
envs/rewards.py                   # WeightedCompositeReward (4-term composite)
envs/wrappers.py                  # GATIInfoWrapper (info dict synthesis)
envs/warmup_wrapper.py            # WarmupResetWrapper (silent steps on reset)
nets/cross_indian_test/           # SUMO test fixture (placeholder demand)
papers/                           # Open-access reference PDFs
results/                          # Run outputs (gitignored)
scripts/smoke_train.py            # Throwaway smoke trainer (kept for sanity checks)
scripts/train.py                  # Real PPO training pipeline (config-driven, wandb)
tests/test_env_contract.py        # 15 contract tests (all passing)
wandb/                            # Wandb local data (gitignored)
```

## Literature context

`docs/literature-review.md` is a 5-paper review covering EMVLight, PressLight,
MPLight, Wei survey, and Verma 2024, with explicit GATI positioning. Read
this first when making design decisions that touch:

- Reward formulations (especially anything pressure-related)
- Observation features (lane segmentation, vehicle-type features)
- Baselines and comparison protocols
- Paper-positioning language for related work

The review flags real concerns about the current reward design (Wei survey
calls multi-term composites like ours "ad-hoc"; literature-review.md
proposes adopting pressure as the wait/queue surrogate). Take those flags
seriously when scoping changes.

The raw PDFs in `papers/` (EMVLight, PressLight, MPLight, Wei survey) are
the source of truth when literature-review.md's summary is not enough —
typically when implementing a specific formula or replicating an exact
mechanism. Do not load the PDFs unconditionally; they consume context
window. Open them only when literature-review.md leaves a question
unanswered.

Verma 2024 is paywalled; only abstract-level claims are available.
literature-review.md flags those claims explicitly. Do not assume access
to the full Verma paper.

## Working rules

These rules came out of Week 1+2 incidents that cost real time. Treat them
as non-negotiable.

### File-state verification

Before editing any committed file, hash both disk and HEAD versions:

```bash
md5sum <path>
git show HEAD:<path> | md5sum
```

If they differ, run `git diff --stat <path>` to see direction-of-change. If
disk is *behind* HEAD (a stale local copy was saved over the canonical),
`git checkout HEAD -- <path>` before editing. Do not skip this for `.docx`
files — Word-saved-over-canonical is a known incident in Week 2.

### Verify before commit

When running `git status`, `git diff --stat`, or post-commit checks, **read
the output before sending the next instruction.** Do not chain ahead.

Three Week 2 incidents came from chaining: a Zone.Identifier file
accidentally committed, stale config paths missed in a commit, an untracked
sumocfg referenced by a committed config. Each required a follow-up cleanup
commit. The rule: every command output is evidence to read, not a checkbox.

### Recommendations need grounding

When asked to recommend a path, library function, file structure, or API

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Don-K-Quixote/GATI](https://github.com/Don-K-Quixote/GATI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
