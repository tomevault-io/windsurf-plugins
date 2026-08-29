---
trigger: always_on
description: Operating guide for AI coding agents (and humans) working in
---

# AGENTS.md

Operating guide for AI coding agents (and humans) working in
`azure-functions-agents-runtime`. Read this first. It defines **how** we make
changes so features stay aligned with the architecture and each other.

- **What this project is:** a markdown-first programming model that turns an
  agent project (`*.agent.md` + config) into an `azure.functions.FunctionApp`,
  powered by the Microsoft Agent Framework (MAF).
- **Design source of truth:** [`docs/architecture.md`](docs/architecture.md).
  This file governs *process*; that file governs *design*. Keep both in sync.

---

## 0. Golden rule

**Every change starts with triage + a dedicated worktree.** Never commit feature
work directly on `main`. Classify the change, size it, then follow the matching
lane below. When in doubt about scope or design, stop and ask.

---

## 1. Feature development lifecycle

Classify the change first, then pick a lane:

| Scope | Examples | Lane |
| --- | --- | --- |
| **nit** | typo, comment, formatting, trivial rename | Worktree → implement → gate → PR |
| **bug** | incorrect behavior, regression | Worktree → (repro test first) → fix → gate → PR |
| **small feature** | self-contained, single module, no new public surface | Worktree → short design note in PR → implement → test → docs → gate → PR |
| **medium+ feature** | new public surface, cross-module, new authoring format or discovery behavior | **Full FRD pipeline** (below) |

### The medium+ pipeline (phases)

Run these phases in order. Each has an exit gate; do not advance until it is met.

| Phase | What happens | Exit gate |
| --- | --- | --- |
| **0 · Triage + Worktree** | Agree scope; create the worktree (§2). | Scope + lane agreed |
| **1 · FRD** | Write a Feature Requirements Document (§4): problem, goals/non-goals, proposed design, **Decisions log**, test plan, docs impact. | FRD drafted |
| **2 · Architecture review** *(planning mode)* | Review the FRD for **completeness** and alignment with the module map in `docs/architecture.md`. Iterate. | Human sign-off recorded in the Decisions log → status `Finalized` |
| **3 · Implementation** | Implement **product changes only**, per the finalized FRD. Keep diffs surgical. | `ruff` + `mypy` clean |
| **4 · Testing** | Design/extend coverage for the new behavior; add tests under `tests/`. | Full gate green (§3) |
| **5 · Docs** | Update `docs/architecture.md` (module map / pipeline), `docs/front-matter-spec.md`, `docs/triggers.md`, and `README.md` as relevant. **For schema changes:** run `eng/scripts/generate_config_reference.py` then use the `update-schema-docs` skill to sync examples. | Docs reflect reality; DoD met (§6) |

> Phases 2 and 4 are explicit *review* checkpoints. Treat them as separate
> passes (ideally a dedicated review sub-agent): an **architecture review** that
> judges the FRD, and a **testing review** that judges coverage. Do not let the
> author's implementation context bias the review.

> **Tooling:** the `add-feature` workflow skill
> ([`.github/skills/add-feature/SKILL.md`](.github/skills/add-feature/SKILL.md))
> automates phases 1–5, and FRDs use the template at
> [`docs/frds/_template.md`](docs/frds/_template.md). When told to "use the
> add-feature skill," follow that playbook; §4 below is the FRD outline.

---

## 2. Worktree convention

One worktree per change, branched off `main`:

```bash
git worktree add \
  ../copilot-worktrees/azure-functions-agents-runtime/<user>-<slug> \
  -b <user>/<slug> main
```

- **Branch:** `<user>/<slug>` (e.g. `vrdmr/agents-folder-indexing`).
- **Directory:** mirrors the branch under
  `copilot-worktrees/azure-functions-agents-runtime/`.
- Remove the worktree after the PR merges: `git worktree remove <path>`.

---

## 3. Canonical commands (the gate)

These mirror CI's lint/type-check/test steps (`eng/templates/jobs/ci-tests.yml`; Python **3.13** and
**3.14**). All three must pass before a PR is ready. The `--cov-report=xml` flag
is what CI runs; drop it (or use the fast loop below) for everyday local runs.
```bash
# One-time setup (editable install with dev extras)
python -m pip install --upgrade pip
python -m pip install -U -e .[dev]
# Lint
python -m ruff check src tests

# Type-check (strict)
python -m mypy src

# Test (with coverage, matching CI exactly)
python -m pytest --cache-clear --cov=./src/azure_functions_agents --cov-report=xml --cov-branch tests

# Fast local test loop
python -m pytest tests -q
```

> `samples/` is intentionally excluded from `ruff` and `mypy`. `tests/` is
> linted but excluded from strict `mypy`.

---

## 4. FRD: Feature Requirements Document

Required for medium+ features. **Location:** committed to the repo at
`docs/frds/<NNNN>-<slug>.md` (zero-padded sequence, e.g.
`docs/frds/0001-agents-folder-indexing.md`) — treated like a lightweight ADR so
the Decisions log is durable history. Start from
[`docs/frds/_template.md`](docs/frds/_template.md); see
[`docs/frds/README.md`](docs/frds/README.md) for numbering. Recommended sections:

1. **Summary** — one paragraph: what and why.
2. **Motivation / problem** — the pain today (e.g. AgentApps with many agents).
3. **Goals / Non-goals** — explicit scope boundaries.
4. **Proposed design** — modules touched, mapped to the `docs/architecture.md`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure/azure-functions-agents-runtime](https://github.com/Azure/azure-functions-agents-runtime) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
