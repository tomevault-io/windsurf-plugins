---
trigger: always_on
description: Operations Research problem repository featuring mathematical formulations, algorithm implementations, and benchmarks. Educational/research-focused with academic rigor (references, complexity analysis, scheduling notation). **57 top-level problems** across **10 families**: Scheduling, Routing, Packing & Cutting, Location & Network, Stochastic & Robust, Combinatorial, Supply Chain, Continuous Optimization, Multi-Objective, plus 48 problem variants. **2339 total tests**.
---

# CLAUDE.md

## Project Overview

Operations Research problem repository featuring mathematical formulations, algorithm implementations, and benchmarks. Educational/research-focused with academic rigor (references, complexity analysis, scheduling notation). **57 top-level problems** across **10 families**: Scheduling, Routing, Packing & Cutting, Location & Network, Stochastic & Robust, Combinatorial, Supply Chain, Continuous Optimization, Multi-Objective, plus 48 problem variants. **2339 total tests**.

**Author**: Mohammad Ghafourian Nasiri
**License**: MIT
**Python**: 3.10+ required (uses `from __future__ import annotations`, `|` union syntax)

## Repository Structure

```
MathematicalModeling/
├── CLAUDE.md
├── README.md              # Project overview + taxonomy
├── CONTRIBUTING.md         # Guidelines for adding problems/algorithms
├── requirements.txt        # numpy, scipy, matplotlib, pytest, pandas
├── docs/
│   └── taxonomy.md         # Full problem classification (9 families)
├── shared/                 # Reusable infrastructure
│   ├── parsers/
│   │   └── taillard_parser.py   # Taillard benchmark downloader/parser
│   ├── exact/              # (placeholder for shared exact method components)
│   ├── metaheuristics/     # (placeholder for generic metaheuristic frameworks)
│   └── visualization/      # (placeholder for visualization tools)
└── problems/
    └── 1_scheduling/
        ├── flow_shop/      # FULLY IMPLEMENTED
        │   ├── instance.py             # FlowShopInstance, FlowShopSolution dataclasses
        │   ├── benchmark_runner.py     # CLI for evaluating algorithms on Taillard instances
        │   ├── exact/
        │   │   ├── johnsons_rule.py    # Optimal for F2||Cmax, O(n log n)
        │   │   ├── mip_formulation.py  # SciPy HiGHS + OR-Tools CP-SAT solvers
        │   │   └── branch_and_bound.py # Taillard lower bound, NEH warm-start
        │   ├── heuristics/
        │   │   ├── palmers_slope.py    # Slope index, O(n*m + n log n)
        │   │   ├── guptas_algorithm.py # Priority-based, O(n*m + n log n)
        │   │   ├── dannenbring.py      # Rapid Access weighted Johnson's, O(n*m + n log n)
        │   │   ├── cds.py             # Campbell-Dudek-Smith, O(m * n log n)
        │   │   ├── neh.py             # Nawaz-Enscore-Ham, O(n^2 * m), best constructive
        │   │   └── lr_heuristic.py    # Multi-candidate greedy, O(n^3 * m)
        │   ├── metaheuristics/
        │   │   ├── iterated_greedy.py      # Ruiz & Stuetzle (2007), state-of-the-art
        │   │   ├── simulated_annealing.py  # Osman & Potts (1989), classic SA
        │   │   ├── genetic_algorithm.py    # Reeves (1995), OX crossover + insertion mutation
        │   │   ├── tabu_search.py          # Nowicki & Smutnicki (1996), fast TS
        │   │   ├── ant_colony.py           # Stützle (1998), ACO with MMAS pheromone bounds
        │   │   └── local_search.py         # Swap, insertion, or-opt, VND neighborhoods
        │   ├── variants/
        │   │   ├── no_wait/           # Fm | prmu, no-wait | Cmax
        │   │   │   ├── instance.py    # NoWaitFlowShopInstance, delay matrix computation
        │   │   │   ├── heuristics.py  # NN, NEH-NW, Gangadharan-Rajendran
        │   │   │   ├── metaheuristics.py  # Iterated Greedy for no-wait
        │   │   │   └── README.md
        │   │   ├── blocking/          # Fm | prmu, blocking | Cmax
        │   │   │   ├── instance.py    # BlockingFlowShopInstance, departure times
        │   │   │   ├── heuristics.py  # NEH-B, Profile Fitting
        │   │   │   ├── metaheuristics.py  # Iterated Greedy for blocking
        │   │   │   └── README.md
        │   │   └── setup_times/       # Fm | prmu, Ssd | Cmax
        │   │       ├── instance.py    # SDSTFlowShopInstance, setup time matrices
        │   │       ├── heuristics.py  # NEH-SDST, GRASP-SDST
        │   │       ├── metaheuristics.py  # Iterated Greedy for SDST
        │   │       └── README.md
        │   └── tests/
        │       ├── test_flow_shop.py       # 57 tests, original PFSP algorithms
        │       ├── test_new_algorithms.py  # 38 tests, new algorithms + variants
        │       └── test_ts_aco_sdst.py     # 35 tests, TS, ACO, SDST variant
        ├── parallel_machine/ # FULLY IMPLEMENTED (7 Python files, 43-test suite)
        │   ├── instance.py              # ParallelMachineInstance (Pm, Qm, Rm)
        │   ├── exact/
        │   │   └── mip_makespan.py      # MIP formulation via SciPy HiGHS
        │   ├── heuristics/
        │   │   ├── lpt.py               # LPT (4/3 approx) + SPT for ΣCj
        │   │   ├── multifit.py          # MULTIFIT (1.22 approx), FFD + binary search
        │   │   └── list_scheduling.py   # Greedy list scheduling (2-1/m approx)
        │   ├── metaheuristics/
        │   │   └── genetic_algorithm.py # GA with integer-vector encoding

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mghnasiri) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
