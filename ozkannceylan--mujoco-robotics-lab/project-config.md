---
trigger: always_on
description: You are working inside the `mujoco-robotics-lab` repository.
---

# CLAUDE.md — Robotics Lab Series

You are working inside the `mujoco-robotics-lab` repository.

## Goal

Build a portfolio-ready robotics lab series using MuJoCo, progressing from simple planar arms to VLA-controlled humanoid manipulation. See `MASTER_PLAN.md` for the full roadmap.

## Context

- Engineer has a mechatronics background with a master's in RL for mobile robotics
- Lab 1 (2-link planar arm) is complete — FK, Jacobian, IK, PD control, trajectory generation
- Lab 2 (UR5e 6-DOF) is complete — scales Lab 1 foundations to an industrial arm with Pinocchio
- Lab 3 (Dynamics & Force Control) is complete — RNEA/CRBA, gravity compensation, Cartesian impedance, hybrid force control
- Lab 4 (Motion Planning) is complete — Pinocchio+HPP-FCL collision checking, RRT*, TOPP-RA trajectory parameterization
- Lab 5 (Grasping & Manipulation) is complete — custom parallel-jaw gripper, DLS IK, pick-and-place state machine, Lab 3+4 integration
- Labs 6–9 are planned — dual-arm, locomotion, whole-body, VLA
- End goals: strengthen fundamentals for humanoid VLA work, prepare for robotics interviews, build a portfolio demo

## Tech Stack

- **Python 3.10+**
- **MuJoCo** — physics simulation, rendering, contact dynamics
- **Pinocchio (pin)** — analytical FK, Jacobian, dynamics (RNEA, ABA, CRBA), collision checking (HPP-FCL)
- **NumPy** — all numerical computation
- **Matplotlib** — plotting, 3D visualization
- **meshcat-python** — optional interactive 3D viewer
- **ROS2 Humble** — bridge node integration (later labs)

## Architecture Principle

```
Pinocchio = analytical brain (FK, Jacobian, M, C, g, IK)
MuJoCo   = physics simulator (step, render, contact, sensor)
```

- Use Pinocchio for ALL analytical computations
- Use MuJoCo for simulation execution and rendering
- Never duplicate computation — if Pinocchio computes it, don't recompute in MuJoCo
- Cross-validate between the two as a correctness check

---

## Per-Lab Workflow

**This is the mandatory workflow for every lab. Follow it in order.**

### Step 1 — Read the lab brief

Each lab has a detailed plan file in the `plan/` directory: `plan/LAB_XX.md`. Read it fully before doing anything else. It contains objectives, theory scope, architecture, implementation phases, key design decisions, and success criteria.

### Step 2 — Create the lab folder and `tasks/` subfolder

```
lab-N-<name>/
├── tasks/
│   ├── PLAN.md           ← Step 3: write this first
│   ├── ARCHITECTURE.md   ← Step 4: write this before any code
│   ├── TODO.md           ← Step 5: create from PLAN, update after every step
│   └── LESSONS.md        ← Step 6: log bugs, debug strategies, insights
├── src/
├── models/
├── docs/
├── docs-turkish/
├── media/
├── tests/
├── ros2_bridge/
└── README.md
```

### Step 3 — Write `tasks/PLAN.md`

Break the lab brief into concrete implementation steps. This is your contract — you execute this plan, nothing more, nothing less.

Format:
```markdown
# Lab N: [Title] — Implementation Plan

## Phase 1: [Name]
### Step 1.1: [Specific task]
- What to build
- Expected output / how to verify
### Step 1.2: ...

## Phase 2: [Name]
### Step 2.1: ...
...
```

### Step 4 — Write `tasks/ARCHITECTURE.md`

Document the full technical architecture BEFORE writing any code. This file is the source of truth for how modules connect.

Must include:
- **Module map:** which Python files exist and what each one does
- **Data flow:** what data flows between modules (diagram or description)
- **Key interfaces:** function signatures for the main APIs
- **Model files:** which MJCF/URDF files are needed and where they come from
- **Dependencies on previous labs:** what is imported from `shared/` or earlier labs

### Step 5 — Create and maintain `tasks/TODO.md`

Generate from PLAN.md. Update after every completed step.

Format:
```markdown
# Lab N: TODO

## Phase 1: [Name]
- [x] Step 1.1: [description] — DONE (2026-03-16)
- [ ] Step 1.2: [description]
- [ ] Step 1.3: [description]

## Phase 2: [Name]
- [ ] Step 2.1: ...

## Current Focus
> Step 1.2: [what you're working on right now]

## Blockers
> None / [describe any blockers]
```

Rules:
- Check off items immediately after completion
- Update "Current Focus" before starting each step
- Log blockers as they appear
- Never skip ahead without updating TODO

### Step 6 — Maintain `tasks/LESSONS.md`

Log every bug, failed approach, and debug insight AS IT HAPPENS. This is not written after the fact — it's a live journal.

Format:
```markdown
# Lab N: Lessons Learned

## Bugs & Fixes

### [Date] — [Short description]
**Symptom:** What went wrong
**Root cause:** Why it happened
**Fix:** What resolved it
**Takeaway:** What to remember for future labs

## Debug Strategies

### [Technique name]
When to use it, how it helped

## Key Insights

### [Insight]
Brief explanation of something non-obvious learned during implementation
```

---

## Execution Rules

1. **Read LAB_XX.md → Write PLAN → Write ARCHITECTURE → Create TODO → Then code.** Never skip steps.
2. **Update TODO.md after every completed step.** If you forget, the next session starts with stale state.
3. **Log bugs in LESSONS.md immediately.** Don't wait until the end. Future labs will hit the same issues.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ozkannceylan/mujoco-robotics-lab](https://github.com/ozkannceylan/mujoco-robotics-lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
