---
trigger: always_on
description: Project-specific context for Claude Code when working in this repo.
---

# CLAUDE.md

Project-specific context for Claude Code when working in this repo.

## Reference Docs

Fetch these on demand when working on related code — don't assume their content, check the live page:

- **JAX** — https://docs.jax.dev/en/latest/ — core dependency (`jax[cuda12]`).
- **flashbax** — https://instadeepai.github.io/flashbax/ — replay buffer backend; only `roxie/utils/memory.py`'s `ReplayManager` should call it directly (see JAX Conventions above).
- **EnvPool** — https://envpool.readthedocs.io/en/latest/ — CPU physics backend, wrapping dm_control's own C++ implementation (not MJX/Warp) behind a gym/dm_env `reset`/`step` API with a native thread pool; only `roxie/environment/loader.py`'s `build_envpool_env` and `roxie/environment/vector.py`'s `EnvPoolVectorEnv` should call it directly. Its pool never exposes `mjModel`/`mjData`, and its `step()` is closed-loop (one call per timestep) — bridged into the fused training `jax.lax.scan` via an `ordered=True` `jax.experimental.io_callback` in `roxie/utils/rollout.py`'s `pool_advance`, which `EnvPoolRollout` binds as its `advance`.
  - Its [XLA interface](https://envpool.readthedocs.io/en/latest/content/xla_interface.html) (`env.xla()`) was built out, measured and **reverted** (2026-09-17) — don't re-attempt without new evidence. The callback cost is per STEP, so it amortizes over `parallel_envs` and is already negligible at benchmark width; the gain only becomes visible at low env counts. It also needs process-global `jax_enable_x64` (every dm_control task declares its action spec float64 in C++ and `send` does no numpy cast), which `mujoco_playground`'s warp FFI cannot tolerate, and it SIGSEGVs on the eval path because `reseed` rebuilds the pool while EnvPool names its FFI target after `id(pool)` and `evaluate` is jitted on a static `self`.
  - Batching the per-step callback — a host-side Python step loop, or one bulk `io_callback` running all T steps — was prototyped and measured, and **does not pay** (2026-09-17). `scripts/profile_envpool_chunk.py` is the measurement: swap only `EnvPoolRollout.advance` and subtract the runs. The callback splits into a FIXED per-crossing cost (~90us) and a host<->device TRANSFER cost (~1.5us per env per step), and only the fixed part is what batching removes — a bulk callback moves the same bytes. T is `steps_between_updates // parallel_envs`, so at the benchmark's 1024 envs T is **2** and the removable cost is 0.18ms of a 71ms iteration (0.3%); at 64 envs 3.6%, at 16 envs 11.2%. Both prototypes then gave that back in the extra per-step dispatches the fused chunk does not pay, netting out at or below the current path. `ordered=False` is a wash too (identical wall time to three figures over 20 chunks), and `ordered=True` must stay regardless: `roll_random`'s actions do not depend on the previous observation, so nothing but the token sequences its callbacks.
  - A `jax.profiler` trace of the same cell confirms XLA's compute lanes are **100% idle for the whole of every callback** — 11 of 12 `tf_XLAEigen` lanes do literally nothing while the 12th runs it. That is NOT reclaimable capacity at benchmark width: `/proc/self/stat` over the same window puts EnvPool's own C++ threads at **10.7 of 12 cores** during exactly that stall (`runtime.cpu_cores` and `env.num_threads` are pinned equal for this reason), so the idle lanes are idle because the cores are taken. At 16 envs the pool only fills 5.6 of 12 and ~19% of the machine really is dead — but the way to spend it is overlapping the replay write behind the next physics step, not batching the crossings, and it is worth nothing at the 1024 envs the grid runs. Two trace caveats: profiling inflates the `python` lane ~6x (a chunk reads 126ms against 21.9ms unprofiled), so only the trace's STRUCTURE is usable, and the ordered-token machinery it makes look expensive (`_add_tokens_to_inputs` -> `device_put` -> `block_until_ready`) is exactly that artifact. EnvPool's threads are not XLA-instrumented and never appear in the trace at all, which is why the occupancy question needs `/proc`.

- **rlax** — https://rlax.readthedocs.io/en/latest/index.html — RL primitives (`td_learning`, `categorical_td_learning`, GAE); used throughout `roxie/losses/` and `roxie/agents/ppo.py`, typically vmapped over the batch/env axis (see JAX Conventions above).
- **MuJoCo** — https://mujoco.readthedocs.io/en/stable/overview.html — physics backend (`mujoco-mjx`); this repo targets MJX, mujoco_warp, and native-CPU MuJoCo across environments (see `pyproject.toml`, `scripts/check_warp.py`, `roxie/utils/native_player.py`).
- **Brax** — https://github.com/google/brax — not a dependency here; useful as a reference JAX physics/RL implementation when comparing approaches.
- **Acme** — https://github.com/google-deepmind/acme — not a dependency here; useful as a reference RL agent architecture (DeepMind) when comparing agent/loop design.
- **Google Python Style Guide** — https://google.github.io/styleguide/pyguide.html — section 3.8 is the docstring and comment contract this repo follows (see Docstrings and Comments below). Check it rather than guessing at section spelling or indentation.

## No Legacy

**roxie is unreleased. Nothing in it is legacy, and nothing needs backward

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vittorione94/roxie](https://github.com/vittorione94/roxie) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
