---
trigger: always_on
description: Always-on project rules for the PhD research repository — simulation, digital twins, uncertainty, formal control, demanufacturing.
---

# PhD Research Repository — Project Rules

## What This Is

A multi-project PhD repository focused on **holonic demanufacturing coordination under structural uncertainty**, combining discrete-event simulation, digital twin architectures, conservative learning, formal DES/supervisory-control guarantees, and LLM-mediated coordination.

## Research Domain

End-of-life electronics demanufacturing — automated disassembly lines where product variety, component degradation, and runtime surprises create **structural uncertainty** (uncertainty over which actions are even feasible, not just parametric noise). The work spans 5 papers/work packages from DES modeling through system-level evaluation.

## Source of Truth

- **Simulation code is the ground truth.** Claims in papers/thesis must be grounded in verifiable simulation outputs and event logs.
- **No lookahead — ever.** Every decision, belief update, or action selection at time T must use only events and observations with timestamps ≤ T.
- **Causal correctness is non-negotiable.** Simulation events must respect process ordering; no retroactive state modifications.
- **Reproducibility is mandatory.** Every experiment must be reproducible from a seed + config + code version.

## Stack

| Layer | Tech |
|-------|------|
| Simulation engines | SimPy ≥ 4.0, salabim, custom DES engines |
| Core language | Python 3.10+ |
| Numerics | NumPy, Pandas |
| Configuration | YAML, JSON |
| Visualization | Pygame (2D token animation), Tkinter (GUI), matplotlib (eval plots) |
| Communication | paho-mqtt (digital twin), Flask (mediator REST API) |
| Testing | pytest |
| Writing | Markdown, LaTeX (papers/thesis) |

## Project Layout

```
Daisy/                      # iPhone disassembly line DES (SimPy)
  main.py                   # CLI: headless, --viz, --batch, --replay
  config/                   # YAML config, loader, schema
  sim/                      # SimPy model (entities, stations, system, monitor)
  experiments/              # Scenario definitions, batch runner
  viz/                      # Pygame visualization, replay binding
  runs/                     # Experiment outputs (events.csv, samples.csv, metadata.json)

DES Playground/             # Multi-sim umbrella (harbour, demanufacturing, manufacturing loop)
  run.py                    # Entry point
  config/                   # YAML configs
  src/
    harbour_sim/            # Port logistics DES
    demanufacturing_sim/    # Holonic demanufacturing with agents, fault injection, LLM orchestrator
    manufacturing_loop_sim/ # Circular manufacturing DES

SUDE/                       # Laptop pilot demanufacturing DES (custom engine)
  main.py                   # CLI: headless or --viz
  config/                   # JSON config
  sim/                      # Custom DES core, model, process, distributions
  viz/                      # Pygame animation
  outputs/                  # Metrics, results

Demanufacturing/            # Holonic digital twin (MQTT-based)
  digital_twin/
    core/                   # Twin engine (SimPy), state store
    holons/                 # Product/resource holons, uncertainty module
    io/ , io_layer/         # MQTT observer, mediator API (Flask)
    mock_hardware/          # Mock factory, robot, operator (MQTT publishers)
    viz/                    # Pygame schematic view

FWO/                        # PhD planning, papers, and main research codebase
  full_phd/                 # Main PhD package (demanuf)
    demanuf/
      des/                  # DES engine, model, supervisor, scenarios
      twin/                 # Event-sourced twin (store, schema, replay, queries)
      holons/               # Baseline holon protocol
      learning/             # Belief updates, feasibility, learning-to-ask
      mediation/            # LLM intent proposal, mediation gate
      eval/                 # Ablation, sweep, metrics, reports
    docs/                   # Work-package notes (wp1–wp5)
    tests/                  # Per-WP test suites
    data/                   # Evaluation runs and outputs

Study Material/             # DES and supervisory control lecture materials
State of the art/           # BPMN models, literature artifacts
RAISE/                      # Visualization prototypes
```

## Conventions

- Prefer **vectorized NumPy/Pandas** over row-by-row loops where applicable.
- Prefer **YAML or JSON configs** for all scenario parameters — never hardcode.
- Simulation parameters (distributions, capacities, probabilities) must be **calibratable** and clearly marked when assumed.
- Every simulation run must produce a **reproducible event log** (events.csv or equivalent).
- Use **seeds** for all stochastic elements; document seed policy in configs.
- Do not introduce ML/LLM components unless explicitly requested or they are part of the mediation gate architecture.

## Simulation Integrity

- Events must be **causally ordered** — no retroactive state changes.
- Resource constraints must be **enforced**, not approximated.
- Exception handling (jams, failures, unknown products) must be modeled explicitly.
- Monitor/logging must not alter simulation behavior (observer pattern only).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TizioMaurizio/Leuven](https://github.com/TizioMaurizio/Leuven) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
