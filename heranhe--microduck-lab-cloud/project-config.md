---
trigger: always_on
description: This repo is a community harness for training and watching RL policies for the
---

# Microduck local-training workspace

This repo is a community harness for training and watching RL policies for the
[Microduck](https://pollen-robotics.com/microduck) — Pollen Robotics' ~25 cm,
~800 g bipedal robot (14 Dynamixel XL330 servos, IMU, 50 Hz control) — **on an
ordinary Mac, no CUDA GPU required**. It is not affiliated with Pollen Robotics.

The workspace is four side-by-side checkouts. This repo tracks two of them; the
two upstream Pollen repos are cloned next to them (they are in `.gitignore`):

| Repo | What it is | Stack |
|---|---|---|
| `microduck_local/` | **This repo.** Local CPU-MuJoCo + Stable Baselines 3 PPO prototyping harness — same 61-obs contract and MJCF as microduck_rl. Includes `duck-lab`, the streaming backend for duck-viewer, and the 🎓 teach-a-trick training loop. | Python 3.12 / uv |
| `duck-viewer/` | **This repo.** Next.js + react-three-fiber browser viewer — many policies/checkpoints walking side by side, live over WebSocket from `duck-lab`. | Next.js / TS |
| `microduck/` | Upstream: the robot's onboard software, shipped ONNX policies, docs. Clone from `pollen-robotics/microduck`. | Rust workspace |
| `microduck_rl/` | Upstream: the official GPU training stack (MuJoCo Warp + mjlab + PPO), BAM actuator sim2real recipe, ONNX export. Clone from `pollen-robotics/microduck_rl`. **This is the sim2real recipe; this repo is the prototyping loop.** | Python 3.12 / uv |

## Setup

One command does all of the below (upstream clones at the pinned shas, the
shipped policies from the Hub, `uv sync`, `npm install`, a smoke test) on a
Mac or Linux:

```bash
git clone <this repo> microduck-workspace && cd microduck-workspace && ./scripts/setup.sh
```

By hand:

```bash
git clone <this repo> microduck-workspace && cd microduck-workspace
git clone https://github.com/pollen-robotics/microduck
git clone https://github.com/pollen-robotics/microduck_rl
# The contract, golden-bit and symmetry tests are measured against specific
# upstream models and policies — the same shas CI pins (.github/workflows/tests.yml).
# (The golden bits are per platform, tests/goldens/; a platform without a
# recording skips them — record yours with MICRODUCK_RECORD_GOLDENS=1.)
git -C microduck_rl checkout badc4e7ffe5507fd7acb1a21487bd2925c1afe5a
git -C microduck checkout 2c61dcc1f03440541cdc0729f7a375b2a9ea3005
cd microduck_local && uv sync            # needs https://docs.astral.sh/uv/
# The shipped policies left the microduck repo for the Hub on 2026-09-03
# (ef4becf); the pinned sha above still vendors them, any later one does not.
# This Hub revision is byte-identical to the vendored set — see setup.sh.
uv run hf download pollen-robotics/microduck-policies \
  alpha_walking.onnx alpha_stand.onnx alpha_sitstand.onnx alpha_ground_pick.onnx \
  ball_kick_left.onnx ball_kick_right.onnx roller.onnx roller_crouch.onnx roulade.onnx \
  --revision 088524a64e2557dc453256b6071dbb9d23888802 --local-dir ../microduck/policies --quiet
cd ../duck-viewer && npm install
```

`microduck_local` finds the MJCF models in `../microduck_rl` (override with
`MICRODUCK_RL_DIR`) and the shipped reference policies in
`../microduck/policies/` (downloaded from
[pollen-robotics/microduck-policies](https://huggingface.co/pollen-robotics/microduck-policies)
by `setup.sh`; upstream no longer vendors them).

## Read the repo-local docs first — they are authoritative

- `microduck_local/README.md` — what the harness is/is not for, every command,
  the measured performance story.
- `microduck_local/AGENTS.md` — **the training playbook for agents**:
  invariants, reward-design rules, verification discipline. Read it before
  touching rewards, observations, or training code.
- `duck-viewer/README.md` — viewer architecture and the GPU pitfalls already hit.
- `microduck_rl/AGENTS.md` (upstream) — the full sim2real playbook the local
  harness mirrors.
- `.claude/skills/render-rollout/SKILL.md` — how to *look* at what a policy
  actually does (works as plain documentation for any agent, not just Claude).
- `.claude/skills/record-world/SKILL.md` — how to *record* a world scenario
  (living room, playroom, soccer pitch) to video + a contact sheet + an events
  log, headless under a seed, and read what the ducks did. Debug the `/sim`
  page with this, not by describing what a browser tab looked like.
- `docs/mars-roadmap.md` — the plan for a THIRD body, Innate's MARS (a
  wheeled base with a 6-DoF arm): what is measured, the `Body`/`RobotSpec`
  split that makes the next robot a registry entry, and the phases with
  the number that settles each. Read it before adding any robot.
- `docs/roadmap.md` — the working list of experiments: what to run next, the
  command for each, and the number that would settle it. Read it before
  starting anything open-ended, and **write the answer back into the item**
  when you finish one — a negative result is worth as much as a positive one,
  and this is where the next person finds out it was already tried.

## What runs where

- **A second body:** the Unitree G1 (29 joints, 99-obs) trains, exports,
  renders and appears in the lab beside the ducks — `robots/spec.py` is the
  seam, `uv run fetch-g1` the setup. A lab contract, not a sim2real one.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [heranhe/microduck-lab-cloud](https://github.com/heranhe/microduck-lab-cloud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
