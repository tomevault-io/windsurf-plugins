---
trigger: always_on
description: JAX continual-learning research track for elizaOS
---

# Alberta Framework — agent guide

JAX continual-learning research track for elizaOS
([The Alberta Plan](https://arxiv.org/abs/2208.11173)). This tree is a
**development fork** of `lalalune/alberta` (fork point `2ac3533`), not a
lightly-patched vendor copy — see `VENDORING.md` for the divergence summary
and the canonical upstream URL. The robot track imports the continual-RL
subset in-process; keep `requires-python >= 3.12` and the `numpy >= 1.26`
floor intact.

**Current headline lane:** the IPMNIST screening/confirmation campaign,
which is development-grade and permanently nonpromoting. Results move; read
the `summary_*.json` files and `publication_runs/RESULTS.md` under
`outputs/ipmnist_screening/` instead of copying numbers into overview docs,
and re-measure the selected baseline before any A/B. The theory snapshot is
`docs/research/ipmnist-theory.md`; raw records and audits live beside the
outputs. Check `docs/evidence/negative-results.md` before retrying an idea.

## Layout

```
alberta_framework/
  core/        learners and adaptive optimizers, Horde, prediction/control,
               learned state, memory, world models, dreaming,
               options/STOMP/OaK, feature lifecycles, and PrototypeAgent
  streams/     synthetic prediction streams + gauntlet, closed_loop,
               pavlovian, recurring_multiagent
  evaluation/  strict evidence artifacts, validators, the evidence registry,
               evidence CLIs, and bounded development diagnostics
  benchmarks/  IPMNIST lanes (upgd_ipmnist, ipmnist_screening,
               upgd_label_emnist), Forager integrations
  utils/       multi-seed experiments, statistics, metrics, export
  steps/       public Step 1–12 kernels: smoke CLIs for Steps 1–2,
               pipeline.py consumes Steps 3–4, Steps 5–12 are
               library-surface only (cited by docs/status.md)
outputs/       evidence + campaign artifacts — see immutability rules below
tests/         unit, integration, scientific, and replay checks
```

Key documents:

- Status & evidence: `docs/status.md` (levels L0–L3, completion gates) ·
  `docs/evidence/methodology.md` (property-by-property map)
- Active campaign: `docs/research/ipmnist-theory.md` ·
  `outputs/ipmnist_screening/{RUNBOOK,FINAL_REPORT,AUDIT,CEILING_ANALYSIS,SOTA_LANDSCAPE_2026}.md`
- Durable records: `docs/archive/forager-comparator-audit.md` ·
  `docs/design/rtu-taylor-correction.md` ·
  `docs/evidence/negative-results.md`
- Runbook: `docs/runbooks/foragax-open-screen.md`
- Benchmarks/infra: `FORAGER_BENCHMARK.md` ·
  `docs/archive/historical-forager-reconstruction.md` · `VENDORING.md` · `CHANGELOG.md`

`README.md` is the index; if you add a root doc, link it there.

## Running things

Always use the project venv:

```bash
.venv/bin/python -m pytest tests/<file> -q                  # one file
.venv/bin/python -m pytest tests -q                         # full suite
.venv/bin/python -m pytest --collect-only -q | tail -1     # count of record
.venv/bin/python -m ruff check .                           # lint (line length 100)
.venv/bin/python -m mypy                                   # strict, py312
.venv/bin/alberta-evidence-status                          # evidence registry
```

See `[project.scripts]` in `pyproject.toml` for the current console-script
inventory. The ones you'll reach for are `alberta-evidence-status`,
`alberta-forager-benchmark`, `alberta-foragax-open-screen`, and the
`alberta-forager-matched-*` family. Benchmark executions happen through
scripts/CLIs, never inside pytest — tests must stay CI-cheap unless
explicitly registered as a scientific lane.

## Marker lanes

- `unit` — fast isolated behavior/contract tests; never scientific evidence.
- `integration` — spans components, persistence, or process/CLI boundaries.
- `scientific` — frozen promoted-evidence protocols; may be expensive and
  require preregistered seeds.
- `slow` — wall-clock heavy modules (>~30s serial); excluded from the fast
  per-PR CI lane.
- `package` — built-distribution and installed-entry-point contracts; isolated
  in the package CI lane.

The fast runtime selector is `-m "not slow and not package"`.

## Evidence-promotion rules (fail-closed)

- **Never auto-promote.** Passing tests, replays, or reruns do not upgrade a
  claim. Promotion requires a frozen preregistered protocol, untouched
  held-out seeds, a versioned artifact schema, and its strict validator
  accepting the artifact.
- **Frozen seeds stay frozen.** Calibration/development seeds and consumed
  evidence seeds can never be reused for promotion. Consumed-seed replays are
  explicitly nonpromoting.
- **Pinned `outputs/` artifacts are immutable.** Never overwrite, edit, or
  delete `outputs/ftl_decision/` (sha-pinned), `outputs/continual_ia/`
  (historical chain + source snapshot), `outputs/recurring_feature/`,
  `outputs/scale_robust_feature/evidence.v2.json`,
  `outputs/continual_multiagent/`, `outputs/step2_canonical/`,
  `outputs/evidence_manifest.json`, the sealed/`QUARANTINED.md` forager
  roots, or the chmod-frozen negative-result dirs. New runs write to NEW
  paths and new schema versions. `outputs/ipmnist_screening/` and
  `outputs/upgd_ipmnist/` hold the active campaign's development artifacts —
  append, don't rewrite.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elizaOS/asi](https://github.com/elizaOS/asi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
