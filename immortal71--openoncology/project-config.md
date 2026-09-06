---
trigger: always_on
description: handles the same mutation having different approved drugs by tumour type (BRAF V600E
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

OpenOncology ranks cancer drugs against a patient's mutation profile. A VCF goes in;
a ranked list of approved drugs, repurposing candidates, or a custom-discovery brief
comes out. Three parts: a FastAPI backend (`api/`), a Next.js 14 frontend (`web/`),
and a Nextflow genomics pipeline (`pipeline/`). Celery workers chain
`genomic_worker` to `ai_worker` to `notify_worker`; Postgres, Redis, and MinIO back them.

The output is clinical evidence, so the ranking path is treated as scientific code
rather than application code. That distinction drives most of the rules below.

## Commands

`make help` lists the task runner. Targets are POSIX-sh friendly and this project is
developed on Git Bash under Windows.

```bash
make test              # backend + frontend unit + E2E
make test-backend      # both pytest suites with the coverage gate
make test-frontend     # Vitest
make test-e2e          # Playwright (boots the app in demo mode)
make lint              # ruff + eslint + tsc
```

Running services locally:

```bash
cd api && uvicorn main:app --reload     # API on :8000, docs at /docs
cd web && npm run dev                    # frontend on :3000
docker-compose up --build                # full stack
```

### The two pytest suites are not one suite

Both the top-level `ai/` package and the app's `api/ai/` package claim the import
name `ai`. One interpreter can bind that name to exactly one of them, so the suites
run as **separate pytest invocations** and coverage is stitched together with
`--cov-append`:

```bash
PYTHONPATH=. pytest api/tests/    # FastAPI app + services (49 modules)
PYTHONPATH=. pytest ai/tests/     # top-level ai/ package
```

Always run from the repo root with `PYTHONPATH=.` and an explicit path.
`pyproject.toml` sets `testpaths = ["api/tests"]` so a bare `pytest` can never drag
both `ai` packages into one process, and `--import-mode=importlib` sits in `addopts`
because collecting `ai/tests/*` fails without it.

Single test: `PYTHONPATH=. pytest api/tests/test_ranking.py::test_name -v`

### Coverage gate

**52**, declared once in `pyproject.toml` under `[tool.coverage.report] fail_under`.
`Makefile` and `.github/workflows/ci.yml` inherit it by omitting `--cov-fail-under` on
the ai-suite invocation. The api-suite invocation passes an explicit
`--cov-fail-under=0`, because coverage is only complete after the second run appends
to it.

Do not put the number back on a command line. It lived in two files before, drifted to
62 against 63, and a local `make test-backend` passed what CI rejected.
`api/tests/test_coverage_threshold_consistency.py` now fails the build if the README
badge, the `CONTRIBUTING.md` prose, or a `--cov-fail-under` literal disagrees with the
config.

The threshold dropped from 69 to 52 when `*/tests/*` was omitted from coverage: a test
module is ~100% covered by the act of running it, and 34 of them were inflating the
headline. Coverage did not regress. The comment in `pyproject.toml` still cites 53.93%
over 8,353 statements; the suite has grown since and now measures about 61% over
8,798, so the gate has more slack than that comment implies.

One trap this creates. Any `pytest --cov=...` run inherits the gate unless it opts
out, so running a single suite alone measures only that suite: `pytest ai/tests/
--cov=ai --cov=api` reports about 14% and exits non-zero with all 42 tests passing.
Add `--cov-fail-under=0`, or drop `--cov`, when you only want to know if tests pass.

### Import paths

Backend modules import as if `api/` were the root: `from database import get_db`, not
`from api.database import ...`. That is how it runs in production (`api/` on
sys.path), and `api/tests/conftest.py` reproduces it with a `sys.path.insert`. Scripts
under `scripts/` put both the repo root and `api/` on the path, so they use
`from api.services.benchmark import ...` instead.

## Protected paths, read before editing

A `PreToolUse` hook (`.claude/hooks/guard_protected.py`, patterns in
`.claude/protected-paths.json`) hard-blocks writes to the ranking and validation
pipeline: `api/ai/**`, `api/services/benchmark.py`, `scripts/**`, `data/**`,
`validation_results/**`, `test-fixtures/**`, `artifacts/**`, the benchmark JSON files,
the ranking test modules, and `PAPER.md`.

The hook is code, not advice. It exists because this repo's four real defects, a
deduplication bug in the holdout sampler, leakage between tuning and evaluation sets,
a live CIViC call during evaluation, and set-iteration order changing output, all
passed the test suite. An agent optimizing for green tests reproduces that class of
bug, and the reported Hit@3 and MRR move without anyone noticing.

If the hook fires: record the proposed change in `BACKLOG.md` under
`## Needs human decision` and move on to other work. Do not route around it with Bash,
sed, a script, or a copy. Routing around it is worse than not making the change.
Humans edit these files in an interactive session with `OO_GUARD_OFF=1`.

Know its one hole before relying on it. The hook derives the repo root from where the
script itself lives, and `relative()` returns `None` for any path outside that root,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [immortal71/openoncology](https://github.com/immortal71/openoncology) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
