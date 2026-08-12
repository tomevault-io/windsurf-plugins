---
trigger: always_on
description: Iteratively patches task verifiers/evals against reward-hacking exploits. Three
---

# Adversarial Verifier Hardening

Iteratively patches task verifiers/evals against reward-hacking exploits. Three
agents (Terminus-2 via Harbor) play adversarial roles:

1. **Hacker** — runs with an aggressive exploit prompt, explicitly told to bypass tests.
2. **Fixer** — given the hack trajectory, modifies any task file to block the exploit.
3. **Solver / Oracle** — pre-check and post-fix validation (validates task is solvable & fix doesn't break it).
4. **Targeted replay** (optional, `--replay-enabled`) — after solver accepts the fix, re-runs a constrained hacker on the patched task with the prior exploit as a replay target. If it re-lands, the fix is rejected.

**Pooled / jumper mode** (optional, `--pool-enabled`): tasks share a defense repo served by a host-side `git daemon`. Fixers clone the pool, pull latest, commit, and push; other tasks consume those pushes on their next iteration via a "skip-hacker" path that lets the fixer integrate pool changes instead of attacking.

## Mode flags (two orthogonal axes)

Two independent flags control the loop:

* `--oracle` — selects the **pre-check/validation mechanism**: deterministic
  reference copy (`reference.py → solution.py`) if set, agent solver
  (Terminus-2) if not.
* `--kernelbench-mode` — selects the **prompt/template bundle**: KernelBench
  framing (speedup metric, `eval_kernel.py`, runtime-minimization hacker goal,
  KB-specific fixer template) if set, generic framing (pass/fail reward,
  `test_outputs.py`, verifier-bypass hacker goal, generic fixer template) if
  not.

The two commonly travel together (KernelBench runs pass both, generic
non-KernelBench tasks pass neither), but they're decoupled because nothing forces the
coupling — a generic task with a deterministic reference could use `--oracle`
alone, and a KB-style framing could conceivably run with an agent solver.

| axis | flag | True | False (default) |
|------|------|------|-----------------|
| Pre-check | `--oracle` | `run_oracle_solver` (deterministic) | `run_solver_agent` (Terminus-2) |
| Prompts / templates | `--kernelbench-mode` | KB framing, `eval_kernel.py`, speedup | Generic framing, `test_outputs.py`, reward |
| Hack / solver thresholds | `--hack-threshold` / `--solver-threshold` | KB uses `10` / `0.5` | Default `1.0` / `1.0` |
| `.legitimate` marker | `--no-legitimate-marker` disables | enabled by default | enabled by default |
| `solver_privileged` | N/A when `--oracle` (deterministic) | — | Optional — inject `/solution/` as solver hint |
| Hardened state | — | `output/hardened/<task_id>/` | `output/hardened/<task_id>/` |

## Loop

Pre-check: solver/oracle must pass the original task.

For each iteration (up to `max_iterations`, default 10):
1. **Hacker attacks** (up to `hacker_retries`, default 3). If all fail → task is **robust**.
2. **Fixer patches** the task (one attempt per iteration).
   - Runs in Docker with tests/, solution/ baked in.
   - Edits files in `/logs/artifacts/` (git repo); only committed changes extracted.
   - On retry: previous attempt + solver output mounted read-only.
   - Can mark hack as legitimate via `.legitimate` sentinel (enabled by default; `--no-legitimate-marker` to disable).
3. **Validate**: solver/oracle must still pass. If not, revert; feedback sent to fixer next iteration.
4. **Targeted replay** (if `--replay-enabled`): up to `replay_retries` attempts to reproduce the prior exploit on the patched task. If reward ≥ `hack_threshold`, the fix is rejected (outcome `replay_broke_fix`) and feedback goes to the next fixer turn. Hardened state is only committed when both solver **and** replay agree.

## Batch concurrency

`harden_batch` dispatches on `pool_server` for two distinct drivers:

* **Pool mode (iteration barrier).** When `--pool-enabled`, all active tasks advance one hardening iteration at a time via `asyncio.gather` on per-task generator yields. The barrier keeps the shared pool's history in lockstep — every task observes the same pool state at iter N start, then optionally pushes, then advances. `pool_max_consecutive_syncs` relies on this lockstep to be meaningful. Cost: fast tasks idle at the barrier waiting for slow peers.
* **Pool-disabled mode (independent).** Tasks have no shared mutable state, so each task generator is driven to completion concurrently with no iter-level synchronization. Container concurrency is still capped by `--max-concurrent`. Throughput is strictly higher than barrier mode for uneven task durations.

Both drivers share the same `asyncio.Semaphore(max_concurrent_containers)` acquired inside `_harden_task_phases` for each container-heavy step.

## Module layout

```
harden-v0/
  harden/
    __main__.py       # CLI (python -m harden)
    config.py         # HardenConfig / BatchHardenConfig
    loop.py           # harden_task + _run_solver dispatch
    agent.py          # run_hacker / run_oracle_solver / run_solver_agent / run_fixer
    llm.py            # retry wrapper around litellm.acompletion
    instructions.py   # build_hacker_instruction + build_targeted_replay_instruction + FIXER templates + SOLVER_HINT
    journal.py        # per-task hacker+fixer journal (shared between the two roles)
    durable.py        # JSON-backed durable-computation decorator (granular resume)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [few-sh/harden-v0](https://github.com/few-sh/harden-v0) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
