---
trigger: always_on
description: AI-powered research tools for quantitative social science: structural econometrics, causal inference, game theory, applied micro, and reproducible pipelines. Built on the compound workflow principle: each unit of research work makes subsequent work easier.
---

# compound-science Plugin

AI-powered research tools for quantitative social science: structural econometrics, causal inference, game theory, applied micro, and reproducible pipelines. Built on the compound workflow principle: each unit of research work makes subsequent work easier.

## Core Workflow

**Brainstorm → Plan → Work → Review → Compound → Repeat**

1. `/workflows:brainstorm` — Explore research approaches, compare methods, pick one. Writes to docs/brainstorms/.
2. `/workflows:plan` — Create an implementation plan from the brainstorm. Writes to docs/plans/.
3. `/workflows:work` — Execute the plan with quality gates and convergence monitoring.
4. `/workflows:review` — Multi-agent parallel review (econometric-reviewer, numerical-auditor, identification-critic, journal-referee).
5. `/workflows:compound` — Extract reusable solutions into docs/solutions/ by category.

Each step has an interactive handoff offering the next step. Use `/lfg` to chain all five automatically, or `/slfg` for parallel swarm execution. Ideate (`/workflows:ideate`) is an optional divergent pre-step before brainstorming.

## Skills (20)

### Workflow (6) — core research loop
- `/workflows:ideate`, `/workflows:brainstorm`, `/workflows:plan`, `/workflows:work`, `/workflows:review`, `/workflows:compound`

### Chain (2) — automated multi-step execution
- `/lfg` — Sequential: brainstorm → plan → work → review → compound (with hard gates)
- `/slfg` — Parallel swarm variant of `/lfg`

### Wrappers (2) — thin routing skills
- `/estimate` — Routes to `/workflows:work` with estimation pipeline context from `empirical-playbook`
- `/replicate` — Routes to `reproducibility-auditor` agent

### Domain knowledge (10) — reusable reference skills
- `structural-modeling` — NFXP, MPEC, BLP, dynamic discrete choice, auction models
- `causal-inference` — IV/2SLS/GMM, DiD, RDD, synthetic control, matching
- `causal-ml` — Double ML, causal forests (GRF), DR-Learner, post-LASSO, high-dimensional controls
- `game-theory` — Nash/SPE/BNE equilibria, entry models, conduct testing, bargaining, multiple equilibria
- `identification-proofs` — Formal identification arguments: target parameter → model → rank conditions → regularity conditions
- `bayesian-estimation` — MCMC, Stan/PyMC/Numpyro, prior elicitation, MCMC diagnostics, Bayesian structural models
- `reproducible-pipelines` — Makefile/Snakemake/DVC, Stata pipelines, environment management, replication standards
- `empirical-playbook` — Method selection, diagnostics by method, power analysis, estimation pipeline, sensitivity analysis, data acquisition (FRED/World Bank)
- `publication-output` — Publication-quality tables and figures: stargazer-style tables, event study plots, RD plots, specification curves
- `submission-guide` — Pre-submission checklists, journal-specific formatting for 20+ journals, referee response strategy and templates

## Agents

### Review (5) — domain-specific code review and methodology verification
- `econometric-reviewer` — Reviews identification, inference, standard errors, calibration strategy, specification flow (model → estimator → code)
- `mathematical-prover` — Verifies proof steps, completeness, regularity conditions, fixed-point arguments
- `numerical-auditor` — Checks floating-point stability, convergence, RNG seeding, matrix conditioning, simulation design (DGPs, metrics, seeds)
- `identification-critic` — Evaluates identification argument completeness, exclusion restrictions, support conditions, equilibrium existence/uniqueness
- `journal-referee` — Adversarial journal referee simulation (contribution, literature, robustness, external validity)

### Research (3) — literature and data investigation
- `literature-scout` — Systematic search for related methods, seminal papers, prior applications
- `methods-explorer` — Estimator properties, computational tradeoffs, software implementations, benchmark parameters and calibration targets
- `data-detective` — Data quality investigation: distributions, missingness, duplicates, panel structure

### Workflow (2) — process, reproducibility, and coordination
- `reproducibility-auditor` — Structural and functional checks for reproducible pipelines and replication packages
- `workflow-coordinator` — Multi-agent workflow coordination, dispatch, triage, and progress tracking

## Domain Signal → Agent Routing

When compaction drops context, use this table to route research questions:

| Signal | Primary Agent | Skill |
|--------|--------------|-------|
| Identification, instruments, exclusion, endogeneity | `identification-critic` | `causal-inference` |
| Estimation, SEs, convergence, calibration | `econometric-reviewer` | `empirical-playbook` |
| Proof, theorem, regularity conditions | `mathematical-prover` | `identification-proofs` |
| Floating-point, Hessian, conditioning, MCMC diagnostics | `numerical-auditor` | `bayesian-estimation` |
| Simulation, DGP, Monte Carlo, power | `numerical-auditor` | `empirical-playbook` |
| Equilibrium, Nash, entry, auction | `identification-critic` | `game-theory` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [James-Traina/compound-science](https://github.com/James-Traina/compound-science) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
