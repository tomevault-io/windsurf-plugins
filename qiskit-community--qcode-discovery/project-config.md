---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install dependencies
# IMPORTANT: uv sync removes packages from groups not listed. Always include ALL needed groups.
uv sync                                          # core only
uv sync --group dev                              # + pytest, python-igraph
uv sync --group dev --group evolve               # + openevolve, litellm
uv sync --group dev --group evolve --group tracking  # + wandb (all groups)

# Run tests
uv run python -m pytest tests/ -v          # all 161 tests (~2s)
uv run python -m pytest tests/ -v -m "not slow"  # skip distance estimation tests
uv run python -m pytest tests/test_known_codes.py::TestEvaluator::test_gross_code_quick -v  # single test

# Run evaluation pipeline
uv run python main.py --quick                    # k-only across all 18 lattices
uv run python main.py --lattices 12,6 6,6        # specific lattices with distance
uv run python main.py --lattices 12,6 -v         # verbose logging

# Run evolutionary search (requires running LiteLLM proxy)
uv run python evolve/run_evolution.py --model anthropic/claude-sonnet-4-5-20250514 --iterations 100

# CSS Campaign 4 (mixed-monomial) — paper run used the server config (pop=750)
uv run python evolve/run_evolution.py --config evolve/config_ansatz_server.yaml --seed evolve/seed_solution_ansatz.py --iterations 300

# Non-CSS Campaign 5 (PBB 4-tuples)
uv run python evolve/run_evolution.py --noncss --iterations 200
```

## Architecture

The system discovers bivariate bicycle (BB) and perturbed bivariate bicycle (PBB) quantum LDPC codes by evolving Python functions that generate candidate polynomial pairs/tuples, then evaluating them through a multi-stage cascade. Five campaigns discovered 465 distinct codes (97 CSS, 368 non-CSS).

### Evaluation core (`evaluation/`)

- **bb_code.py** — Construct CSS `qldpc.BBCode` from exponent tuples
- **pbb_code.py** — Construct non-CSS PBB codes from 4-tuples (A, B, C, D); C=D=None gives CSS
- **evaluator.py** — 5-stage cascade for CSS: validate → k → quick distance → refined distance → exact distance. `evaluate_candidate()` is the central function. Also has `evaluate_candidate_milp()` for MILP-in-loop
- **distance.py** — BP-OSD distance estimation (CSS codes, OSD_0 and OSD-CS)
- **distance_milp.py** — MILP exact distance: CSS (Hamming weight) and symplectic (non-CSS, binary-OR linearization of per-qubit support) formulations via HiGHS/scipy
- **distance_bposd_noncss.py** — BP-OSD for non-CSS codes with achievable-syndrome sampling (critical: random syndromes fail on non-CSS codes)
- **clifford_equivalence.py** — Check if a non-CSS code is genuinely non-CSS or Clifford-equivalent to CSS
- **mirror_code.py** — Mirror code construction (Khesin & Lu, arXiv:2603.05496)
- **tanner_equivalence.py** — Permutation-equivalence decision via BLISS canonical labeling of the colored Tanner graph (sound + complete for permutation equivalence; not LC / Clifford / general code equivalence). CSS variant (`canonical_hash`) uses 3 colors {qubits, X-checks, Z-checks}; non-CSS variant (`canonical_hash_noncss`) uses 3 colors {qubits, per-stabilizer X-support, per-stabilizer Z-support} with a tying edge between each stabilizer's X-support and Z-support vertices (without the tying edge, BLISS would permit independent permutations of X- and Z-support classes — strictly broader than stabilizer permutation equivalence).
- **results.py** — JSON persistence & Pareto front
- **tracking.py** — JSONL run logging and metrics

FOM = k*d²/n. Trust filter: `DISTANCE_TRUST_RATIO=1.3` and `DISTANCE_UNTRUST_RATIO=2.0` in evaluator.py.

### Evolutionary framework (`evolve/`)

Three campaign families with separate seeds, configs, and evaluators:

| Campaign | Seed | Evaluator | Config | Polynomial type |
|----------|------|-----------|--------|----------------|
| 1-3 (CSS trinomial) | `seed_solution.py` | `openevolve_evaluator.py` | `config.yaml` | 2-tuple (A, B) |
| 4 (CSS mixed-monomial) | `seed_solution_ansatz.py` | `openevolve_evaluator.py` | `config_ansatz.yaml` | 2-tuple (A, B), 4-6 terms |
| 5 (non-CSS PBB) | `seed_solution_noncss.py` | `openevolve_evaluator_noncss.py` | `config_noncss.yaml` | 4-tuple (A, B, C, D) |

- **run_evolution.py**: Entry point. `--noncss` flag auto-selects Campaign 5 config. Supports `--models` for ensemble, `--resume` for checkpoints, `--config`/`--seed` overrides
- **prompt_context.md / prompt_context_ansatz.md / prompt_context_noncss.md**: Domain knowledge fed to LLM for mutations

### Key types

CSS candidates: `list[tuple[int, int]]` — `(x_exp, y_exp)` pairs. A candidate pair is `(A_terms, B_terms)`.
Non-CSS candidates: 4-tuples `(A_terms, B_terms, C_terms, D_terms)` where C, D are perturbation polynomials.
`generate_candidates(ell, m)` returns lists of candidate tuples.

## Domain context


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qiskit-community/qcode-discovery](https://github.com/qiskit-community/qcode-discovery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
