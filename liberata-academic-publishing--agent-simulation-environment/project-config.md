---
trigger: always_on
description: Agent brief for this academic research project. This is the source of truth for
---

# Liberata Peer-Review Simulation

Agent brief for this academic research project. This is the source of truth for
project context; keep it concise and current. `README.md` is the human-facing doc.

## What this is

An agent-based simulation of the Liberata academic publishing platform. Each
day, every agent chooses how to spend its turn between two activities:

1. **Writing papers** (advancing its own research), and
2. **Peer review** (participating in a review marketplace for academic capital).

The research question is **good-faith vs bad-faith peer review**: the sim
presents both choices and observes the emergent agentic behavior. Reviewer share
does not depend on the good/bad-faith classification; the classification only
affects the reviewed paper's future accrual rate.

## Core mechanics

`config.py` is the single source of truth for all tunable parameters. Edit the
`SimConfig`/`TrainConfig` dataclass defaults to change behavior everywhere; pass
CLI flags for one-off overrides.

- **Writing effort**: each `write_paper` adds a `writing_effort_delta`; once
  cumulative progress reaches the current paper's effort target, a paper
  publishes and progress resets. `paper_effort_mode` can preserve fixed targets
  or sample the 50-150 timestep band discussed in sync.
- **Review paradigms**: each environment run is either `continuous` or
  `discrete`, configured by `review_paradigm`; paradigms are not mixed inside one
  run.
- **Continuous review effort**: agents choose time spent by continuing or
  finishing reviews. The environment classifies completed reviews as bad faith
  below `good_faith_review_threshold` and good faith at or above it.
- **Discrete review effort**: agents choose `bad_faith` or `good_faith` when
  claiming a review. By default `T_B = 1`, `T_G = 5 * T_B`, and discrete
  manuscript work uses `T_M = 200 * T_B`.
- **Review reward curve**: by default `review_effort_curve = "sigmoid"` models
  `E = F(T)` with low impact for short reviews, a rise through the good-faith
  region, and saturation for long reviews. Set it to `"log"` to use the older
  logarithmic curve, or `"jump"` to test the high-effort threshold experiment.
- **Review-share economics**: a review grants ownership share on the reviewed
  paper. Fair-market offers use
  `ε/(1+ε) × (F−A₀)/F × reviewer_surplus_share` (incremental surplus split;
  default 50/50), estimated from reviewer epsilon history and forecast horizon,
  then adjusted by quality/scarcity/adaptive multipliers and clamped to
  `[min_offer_share, author share]` (defaults 0--100%). Author adaptive pricing is controlled only by
  `pricing_policy` (`static_fair_market` | `adaptive_multiplier`); scarcity
  pricing is disabled automatically when adaptive pricing is active.
- **Results display**: `History.to_dict()` exports `agent_group_summary`, and
  both `run_simulation.py` and the static `docs/` gallery compare heuristic,
  random, probabilistic, RL, and low-talent RL agent outcomes.
- **RL settings** live in `SimConfig` too (`rl_backend`, `rl_epsilon`,
  `rl_gamma`, reward weights). Continuous tabular/linear RL uses `train_rl.py`;
  discrete 3-action RL (write / bad claim / good claim) uses
  `train_discrete_rl.py` and `DiscreteQLearningAgent`. Low-talent RL agents
  (`LowTalentQLearningAgent`) load a separate policy trained with
  `train_rl.py --low-talent` (saved to `policies/policy_<backend>_low_talent`).
- **DQN settings** are separate (`num_dqn_agents`, `dqn_*` hyperparameters);
  train with `train_dqn.py`, policies saved to `policies/policy_dqn.pkl`.

## Architecture map

- `config.py` - `SimConfig` (`SIM`) and `TrainConfig` (`TRAIN`) dataclasses;
  config-first with CLI overrides. `default_policy_path()` resolves policy files.
- `Agent.py` - abstract `Agent` base + the action protocol: `write_paper`,
  `peer_review`, `finish_review_write_paper`, `finish_review_peer_review`.
  `ActionRecord` describes one turn. `Agent.all_papers` is a shared class list.
- `HeuristicAgent.py`, `QLearningAgent.py`, `DiscreteQLearningAgent.py`, `DQNAgent.py`, `RandomAgent.py` - agent variants,
  including random controls and discrete-only probability agents.
- `Paper.py` - paper economics, reviews, and accrual; defines
  `MIN_REVIEW_EFFORT_THRESHOLD`, `REVIEW_EFFORT_PER_DAY`.
- `Environment.py` - the world / turn loop (`agentact`, `nextstep`).
- `History.py` - run logging and metrics (e.g. `gini`).
- `run_simulation.py` - main entry point (run a sim, print summary, optionally
  archive to the `docs/` gallery).
- `train_rl.py` - continuous self-play RL training + greedy evaluation.
- `train_discrete_rl.py` - discrete 3-action RL; saves `policies/policy_<backend>_discrete.pkl`.
- `train_dqn.py` - DQN self-play training; auto-saves to `policies/policy_dqn.pkl`.
- `visualize.py` - charts and summary figures for saved runs.
- `docs/` - static GitHub Pages gallery of saved runs (`docs/data/<run_id>/`).

## Commands

```bash
# Run a simulation (prompts to archive afterward)
python run_simulation.py
python run_simulation.py --no-archive            # don't save
python run_simulation.py --name "my run"         # save non-interactively
python run_simulation.py --review-paradigm discrete --random-agents 5 --no-archive

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Liberata-Academic-Publishing/Agent-Simulation-Environment](https://github.com/Liberata-Academic-Publishing/Agent-Simulation-Environment) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
