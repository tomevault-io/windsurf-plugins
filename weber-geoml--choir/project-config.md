---
trigger: always_on
description: Guidance for humans and agents contributing to **Choir itself**. If you want to
---

# AGENTS.md — working in the Choir codebase

Guidance for humans and agents contributing to **Choir itself**. If you want to
*use* Choir — run a project or contribute proofs to one — read
[`README.md`](./README.md), then [`docs/agents/ORCHESTRATOR.md`](./docs/agents/ORCHESTRATOR.md)
(overseer) or [`docs/agents/CONTRIBUTOR.md`](./docs/agents/CONTRIBUTOR.md) (contributor).
This file is about changing Choir's own code.

## What Choir is

An open protocol for community-led formalization in kernel-checked proof
assistants — Lean 4 (the reference profile), Isabelle, and Rocq: tasks, claims,
and verification encoded as GitHub-native conventions, plus a reference
toolchain for overseers and contributors. It coordinates AI agents running on
contributors' own hardware, billed to contributors' own LLM accounts, behind a
deterministic trust/verify gate.

Four roles, and the names are used consistently throughout the code:

- **Overseer** — the human who owns a project: sets the goal, the audit policy,
  the automation level.
- **Orchestrator** — an LLM agent on the *overseer's* machine (their LLM
  account, their `gh` auth). Plans, decomposes, publishes tasks, reviews PRs,
  merges per the automation level, replans. Agent-agnostic: Choir ships a
  playbook (`docs/agents/ORCHESTRATOR.md`) and a toolkit (`orchestrator/`), never an
  LLM binding.
- **Workers** — contributor agents on contributor hardware and accounts; claim,
  solve, and submit via the `choir` CLI (`client/`).
- **Gate** — the deterministic GitHub Actions side (`gate/`): intake
  validation, verify audits, reconciliation. It never exercises judgment.

## Architecture in one paragraph

The substrate is Git + GitHub. Tasks are Issues, submissions are PRs, identity
is GitHub OAuth, and the audits run as GitHub Actions — event-triggered and
scheduled. There is no host-run server, no GitHub App, and no webhook endpoint:
gate logic lives as Python modules in `gate/`, invoked from thin workflow YAMLs
in `.github/workflows/`. The thinking happens at the edges — the orchestrator
plans and reviews, workers prove. The plan lives in the orchestrator's
repo-versioned roadmap, not in the substrate, and only ready tasks are
published; the substrate stays deliberately dumb.

## Non-negotiables

These are load-bearing for the design. Don't relax one without opening a
discussion first.

- **Trust + verification is Day-1.** Every PR passes the gate: clean-room
  rebuild from source; a statement-identity check that binds per prover
  (`gate/checks.py`'s `PROVER_OVERRIDES`) — on lean4, `verify-comparator`
  checks the target's statement at the kernel level across its whole dependency
  closure, and `statement-equiv` is advisory there; on isabelle and rocq,
  `statement-equiv`'s string comparison is the blocking check; a
  statement-immutability audit (every declaration already present in a changed
  file must arrive untouched); an axiom-dependency audit with per-prover trust
  vocabulary (`gate/provers/`); and a sorry-delta audit.
  **No merge path may bypass these** — including the
  orchestrator's: `merge_pr` runs its own preflight refusing any PR whose
  required checks aren't present and green, at every automation level. Branch
  protection is a second layer where available, not the enforcement.
- **Never proxy API keys.** Choir must not see, store, or forward contributor
  LLM credentials. The contributor's agent runs on the contributor's hardware,
  billed to their account.
- **Server-side rebuild is the merge gate**, not a contributor-claimed build. A
  local `lake build` is informational only.
- **Pull, not push.** Contributor agents poll for typed tasks. Cron-driven
  contributors don't fit push-dispatch.
- **Pinned versions.** Each project pins its prover toolchain and library
  dependencies. In-flight tasks see the version they started with; mid-run
  version bumps are blocked.
- **Multi-agent racing is not the default.** "Send the same task to N agents
  and pick the best" is an explicit escalation mode, never the standard path.
- **Apache-2.0 or MIT only.** No GPL/AGPL or non-commercial code may be
  vendored into the tree. Restrictively-licensed dependencies, if used at all,
  run as separate processes.

## Stack and conventions

- **Python 3.11+** (stdlib `tomllib`), type hints required on new code, `from
  __future__ import annotations` at the top of modules.
- **Async** for poll loops and any future HTTP layer.
- **Package manager:** `uv`.
- **Lint:** `ruff check` — run it before every commit.
- **Tests:** `pytest`. Tests for trust/verify components are load-bearing;
  emphasize them. Run the whole suite before committing. `pytest` deselects the
  `slow` marker — those tests run a real Isabelle or Lean toolchain to check
  `gate/provers/`'s model of the prover against the prover itself, and they
  skip where no toolchain is installed (CI installs none). Run `pytest -m slow`
  before changing anything under `gate/provers/` or `gate/verify/`.
- **Prover toolchain isolation:** `elan` per project via `lean-toolchain` for
  lean4 (Isabelle and Rocq analogs per design note 12). No Docker on the
  contributor side by default.

## Layout

```
choir/
├── gate/           # deterministic trust substrate, invoked from workflows

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Weber-GeoML/Choir](https://github.com/Weber-GeoML/Choir) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-03 -->
