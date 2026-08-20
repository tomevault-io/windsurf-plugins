---
trigger: always_on
description: An index, not a summary. Almost everything worth knowing is already in the
---

# AGENTS.md — working on this project

An index, not a summary. Almost everything worth knowing is already in the
repository — the point of this file is to tell you where, and to stop you
redoing a handful of things that have already been tried and measured.

## What it is

A six-agent CrewAI pipeline that assesses whether a piece of academic research
is commercially viable. Three evidence agents (academic / patent / market) run
in parallel, then a report writer, a reviewer, and a scorer. Served two ways
from one codebase: a FastAPI + vanilla-JS web client and a CLI.

Live at https://academic-commercialization-agent.up.railway.app

Current state: 1106 tests (529 subtests), CI green on Linux + Windows × Python
3.11/3.12, deployed on Railway.

## Commands

```bash
uv run pytest -q                       # the whole suite; ~7s, zero network
uv run --with ruff ruff check .        # CI uses latest ruff, the local pin is older
uv run uvicorn api.main:app --reload   # web client on :8000
uv run python -m academic_agent.main "<topic>"   # one run from the CLI
```

`uv run --with ruff` is deliberate. The pinned local ruff is 0.12.0 and CI
resolves `latest`; they disagree, and the flag is what makes a local check mean
what CI will say.

## Hard conventions

- **`filterwarnings = ["error::UserWarning"]`.** What it mainly guards: code
  paths that would reach a paid API warn and fall back rather than raising, so
  a test that leaks would otherwise pass silently. When a test trips it, read
  *which* warning fired before concluding anything. If it is the network one,
  the test is calling something real — fix the test's isolation, never add an
  ignore. But the project also warns deliberately on an unrecognised weight
  profile and on an audit screen that failed, and two test files assert those
  on purpose (`assertWarns`), so not every UserWarning is a leak.
- **No bare `except Exception`** without a `# noqa: BLE001` and a reason. ruff
  enforces it; the exceptions in this repo each say why in a comment.
- **CI also runs pylint, narrowly,** for `E0701` alone (except-clause ordering).
  ruff has no equivalent, and a real bug here was `except HTTPError` placed
  after `except URLError` — dead code, so every auth failure was retried as
  transient.
- **Commit messages carry the reasoning.** They are long on purpose and are a
  primary record — see below. **Do not add `Co-Authored-By` lines.**
- **Reply to the user in Chinese.** The code, comments, and commit messages
  stay in English.

## Where the reasoning lives

This project deliberately keeps its "why" out of chat logs and inside artifacts
that travel with the repo. When you want to know why something is the way it
is, in this order:

| Question | Where |
|---|---|
| Why is this code like this? | The comment above it. Non-obvious decisions are commented, including rejected alternatives |
| Why does this test exist? | Its docstring names the specific failure it caught |
| Why was this change made? | `git log` — bodies run to ~25 lines and explain the alternative that was rejected |
| Was this hypothesis tested? | `docs/prereg-*.md` — predictions and falsification criteria registered *before* paid runs |
| Full decision history, first person | `notes/简历项目说明.md` — **a separate private repo** (`shuxiachai/academic-agent-notes`), gitignored here. 3,526 lines, 45 write-ups. Ask the user for access if you need it |

Before writing or modifying CrewAI code specifically — the crew, the agents,
the task definitions — read `docs/crewai-reference.md`. That is the vendor's
auto-generated reference, and the library moves faster than most training
data. It used to occupy this filename, which meant an assistant opening the
repo got 1,000 lines about the framework and nothing about the project.

## Do not redo these

Each of these was implemented or measured and then rejected **on data**. The
numbers are the point; if you want to revisit one, beat the number.

- **Do not change the scoring formula or the `evidence_confidence` floor.**
  Every calibration figure this project reports comes from 30 runs scored under
  the current rules (26/30 TRL calibration, 30/30 formula correct, 30/30
  completed). Changing the computation retires that baseline, which is the only
  external anchor the project has. The floor is knowingly asymmetric — it only
  ever raises confidence — and `evidence.py` says so explicitly.
- **Do not add a "maturity language vs TRL" consistency check.** Built,
  measured on all 30 baseline reports, produced 2 flags and both were wrong: a
  paragraph attributes properties to several subjects and phrase matching
  cannot tell which one carries it. Requiring the topic's words nearby does not
  help — the incumbent in a carbon-capture report is also called "capture". The
  reasoning is preserved in `consistency.py` so nobody tries it a third time.
- **Do not make the uncited-claim screen blocking.** Measured on 30 delivered
  reports: 134 warnings across 30/30 reports, so blocking would have failed
  every run. After four exclusions it is 62 across 20/30 — still two thirds.
  The precision work is worth continuing; the blocking is not viable yet.
- **Do not add prompt caching.** Measured: 3.4% cache hit. A run is 6 requests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shuxiachai/academic-commercialization-agent](https://github.com/shuxiachai/academic-commercialization-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
