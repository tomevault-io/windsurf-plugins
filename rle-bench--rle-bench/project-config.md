---
trigger: always_on
description: Instructions for Claude Code working in this repo. Read this before editing.
---

# AGENTS.md — RLE-Bench

Instructions for Claude Code working in this repo. Read this before editing.

## What this is

RLE-Bench is a full-stack robotics engineering benchmark for coding agents,
packaged as [Harbor](https://github.com/laude-institute/harbor) tasks. Nine
families (task01–09) each hand the agent a real engineering problem —
learn policies under a metered simulator, engineer a VLA training recipe,
train a humanoid tracking controller, solve tabletop puzzles, design a robot
from stock parts, co-design teleop hardware, build a perception stack, ship a
bin-clearing policy — and score the submission with a verifier the agent never sees.

You are building the **harness** (golden models, metrics, scenario runners,
scorers), NOT solving the tasks. Keep each harness and its agent-facing task
cleanly separated.

## Hard invariants — do not violate

1. **Score against the harness's own instrumentation, never the agent's artifacts.**
   Checkpoints read the harness's own sim state. The agent-under-test's self-reported
   CSVs/plots are graded *separately* (checkpoints S6.*) for consistency only.
   Nothing in `tasks/task08/harness/base_design/` may trust files the agent produced as truth.
2. **The agent's environment must never contain ground truth.** Nothing under
   `tasks/task08/environment/` or in `instruction.md` may include or import
   `harness.base_design`, `tasks/task08/harness/assets/golden/`, or the reference design. Ground truth lives
   only in the verifier image (`tasks/task08/tests/`), which Harbor runs as a *separate*
   environment (`[verifier] environment_mode = "separate"` in `task.toml`). Keep
   this import/content boundary absolute.
3. **Determinism.** Pin MuJoCo version (`requirements.txt` and both Dockerfiles),
   timestep, solver iterations, and all seeds. No wall-clock, no unseeded RNG.
   Same input => same reward.
4. **Golden-first.** Never implement a stability check without a matching analytic
   unit test. The golden model is ground truth: it must score ~100 and never tip.

## Harbor task layout — follow exactly

Each task is a Harbor task directory. Its generated content (environment/assets,
tests/harness, tests/models, solution/payload) is GITIGNORED and rebuilt by
`make task-assets` (or `make taskNN`, which also builds the family's images) --
a fresh clone must run one of them before `harbor run`. Task 08:

```
tasks/task08/
├── task.toml              # metadata, timeouts, resources; [verifier] environment_mode = "separate"
├── instruction.md         # agent-facing prompt: spec, budgets, envelope (qualitative), I/O contract
├── environment/           # agent container — NO ground truth (invariant #2)
│   ├── Dockerfile         # pinned Python + MuJoCo, headless; same pins as requirements.txt
│   └── assets/            # component library + shelf scene + agent-facing harness API
├── solution/             # optional
│   └── solve.sh           # Harbor Oracle: reference solution or documented protocol smoke check
└── tests/                 # the verifier — scoring runs here, not in the agent container
    ├── Dockerfile         # installs the harness, golden controller, reference design (must contain /tests/test.sh)
    └── test.sh            # re-runs scenarios on the agent's submitted model, invokes the scorer,
                           # writes /logs/verifier/reward.json
```

Harbor rules to respect when touching anything under `tasks/`:

- **Reward file:** `tests/test.sh` must write `/logs/verifier/reward.json` —
  `{"reward": <total in [0,1]>, ...}` plus per-checkpoint breakdown fields
  (Harbor reads `reward.json` first, falls back to `reward.txt`). The total
  encodes the weighted stage sum with the Stage-1 gate already applied.
- **Artifact handoff:** the agent's deliverables (submitted model, controller,
  analysis CSVs) reach the separate verifier via `/logs/artifacts/`
  and the `artifacts = [...]` list in `task.toml`. The verifier treats them as
  untrusted input (invariant #1).
- **Absolute paths** in `test.sh` and any `solve.sh` (Harbor copies them to `/tests/`
  and `/solution/` at runtime).
- **Network:** default `network_mode = "no-network"` for both agent and verifier;
  everything needed is baked into the images.
- **Oracle solutions are optional.** A task may omit `solution/`, including
  per-step solutions. If provided, document whether the Oracle solves the task
  or only exercises the protocol. Validate full solutions against their expected
  scores; a protocol smoke check need not solve the task and does not establish
  solvability. Run `-a oracle` only where a runnable script is available.
  Existing golden-model, metric, and verifier checks remain required.

## Repo layout

One rule places every file: what consumes it, and whether it is code, data,
or a build input.

```
rlebench/            ONE Python package. core/: the task-agnostic spine
                     (scoring aggregate + gate, model validity) that families
                     import as rlebench.core and stage into their images.
                     The rest is host-only: the operations CLI
                     (list/prepare/run/clean/oracle/sweep/check/inspect/doctor/view)
                     and the Python it runs (taskgen, the inspectors).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RLE-Bench/RLE-Bench](https://github.com/RLE-Bench/RLE-Bench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-15 -->
