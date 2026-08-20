---
trigger: always_on
description: ASI is SlopDotCash's JAX continual-learning research and hillclimbing project. Its
---

# ASI — agent guide

ASI is SlopDotCash's JAX continual-learning research and hillclimbing project. Its
north star is one end-to-end agent that keeps learning through an operational
life, retains and reuses useful knowledge, adapts without whole-agent or
task-by-task reinitialization, operates within explicit compute, memory, and
latency budgets, and scales from research benchmarks to real work — especially
robotics.
State-of-the-art continual learning is the destination, not a current claim.

[The Alberta Plan](https://arxiv.org/abs/2208.11173) is a foundational
inspiration and a source of inherited mechanisms, vocabulary, and file names.
It is not ASI's specification or outer boundary. Follow the evidence wherever
it leads: improve Alberta-derived ideas, combine them with other continual-
learning and reinforcement-learning methods, or replace them when stronger
concepts win.

This tree is a **development fork** of `lalalune/alberta` (fork point
`2ac3533`), not a lightly-patched vendor copy — see `VENDORING.md` for the
divergence summary and canonical upstream URL. ASI is the project identity;
`alberta_framework`, the `alberta-framework` distribution, `alberta-*` CLIs,
and historical schema IDs remain compatibility and provenance interfaces. Do
not casually rename them. The robot track imports the continual-RL subset
in-process; keep `requires-python >= 3.12`, the `numpy >= 1.26` floor, and the
existing import surface intact.

**Current program hillclimb:** run a permanently nonpromoting matched
development scorecard across `SwitchingTwoStateMDP` and `RiverSwimMDP` with
frozen/no-learning, random, finite-horizon privileged dynamic-programming, and
strong SARSA-family controls plus explicit resource accounting. This is
development selection only; it does not populate `reference-dev` or create
performance or scientific evidence.

The scorecard implementation is in
`alberta_framework/{reference_life_controls.py,benchmarks/reference_life_scorecard.py}`
and runs through `asi-reference-life-scorecard`. Its literal-frozen plan has
12 consumed development seeds, 2 environments, and 6 arms (144 fresh-process
shards); every shard binds its current source identity, and aggregation requires
one matching current identity. The privileged normalization arm is an
environment-bound finite-horizon dynamic-programming control and is excluded
from candidate Pareto comparisons.
All agent RNG roots use explicit Threefry keys. Records enforce the fixed reward
lattice, complete counters, canonical initial/final numeric payload accounting
(including static oracle policy bytes), and permanently nonpromoting policy.
Timing remains telemetry-only until a separately qualified timing protocol
exists. The implementation and validators do not constitute a completed run or
performance result; consistency hashes are not authenticated execution proof.

The unfrozen `preview1` transaction protocol, pure reducer/CAS ledger,
primitive-only exact-dispatch Prototype bridge, and aggregate runner are in
`alberta_framework/{reference_agent,prototype_reference_adapter,reference_life}.py`.
One immutable aggregate owns agent, environment, transaction, dispatch, RNG
cursor, metrics, counters, recovery state, transcript, and generations. Its
process-local outer lock covers command issuance through execution, receipt,
outcome, learning/metrics staging, and one aggregate commit; retained tests
cover concurrency, horizon, strict execution validation, faults, and
no-redispatch pending-outcome recovery. The primitive RiverSwim slice uses a
distinct environment manifest/state discriminator and stationary metrics,
rejects configurations outside `2 <= n_states <= 12` before its exponential
oracle is constructed, passes the exact same runner-derived JAX key to
execution and validation, and replays the keyed stochastic transition during
strict validation.

The development-only checkpoint codec in
`alberta_framework/reference_life_checkpoint.py`, covered by
`tests/test_reference_life_checkpoint.py` and
`tests/test_reference_life_riverswim_checkpoint.py`, now implements the
current-schema quiescent whole-life checkpoint and exact-resume gate for the
primitive Prototype + SwitchingTwoState and Prototype + RiverSwim lives. A
successful save uses Linux atomic no-replace publication for one immutable
generation, advances commit/checkpoint generations, nests the complete
Prototype v3 checkpoint, binds the aggregate state plus current
source/runtime/dependency identities and consistency hashes, reconstructs fresh
components from the environment discriminator, validates strict
cross-component adoption, and produces exact original-versus-restored
continuation from the same persisted barrier.

This remains a `preview1` L0 mechanism, not a frozen or portable checkpoint
contract, `reference-dev`, safety conformance, robotics readiness, RiverSwim
learning or performance benefit, performance evidence, or scientific evidence.
Checkpoints support only quiescent pre-completion state for the two implemented
simulators and fail closed across current source/runtime/dependency drift; they
do not restore in-flight, halted,
pending-outcome, completed, or physical state. The hashes bind consistency but

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SlopDotCash/asi](https://github.com/SlopDotCash/asi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
