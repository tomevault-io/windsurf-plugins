---
trigger: always_on
description: Purpose of this file: give any agent (or human) landing in this repo the
---

# AGENTS.md — AI Recipe Registry (working title: `strixhalo-recipes`)

Purpose of this file: give any agent (or human) landing in this repo the
full context needed to contribute safely — what this project is, which
documents are authoritative, what roles agents play, and the hard rules
that protect the data's trust model.

Status: **v1 implemented & live-verified (2026-09-07)**. Repo root
`/home/itzco/Projects/strixhalo` IS the product repo (git init'd; code at
root, `openspec/` planning committed for transparency, `specs/` holds
founding reference docs). Source specs below are now reconciled into
`SPEC.md` / `FORMAT.md` / `SKILL.md`; remaining drift reports to the
issue tracker.

---

## 1. What this is

An open-source git repo that acts as a **database of recipes and
knowledge for running local models on Strix Halo class machines**
(unified-memory Linux APUs — this design targets 128 GB Strix Halo
first, but nothing is hardcoded to it).

A recipe is a reproducible, *explained* serving configuration: pinned
backend, launch command, why each parameter was chosen, what the author
was trying to achieve, and which capabilities (tools / MCP / vision /
agent harness) were actually confirmed — not guessed.

The repo is built **agent-first**: an agent clones it, installs its
skill, searches recipes, replicates one from a plain script, runs tests,
submits results, and opens one PR per experiment. Humans validate via
multi-witness rules, not by trusting a single commit.

The failure mode this exists to prevent: every agent independently
"tries things", commits a message full of confident understanding, and
nobody can cross-check other recipes or replicate results. Everything
here exists to make cross-checking and replication cheap.

## 2. Source documents (reconciled into the current model)

| Doc | Role | Origin |
|---|---|---|
| `SPEC.md` | Product spec: object model, trust rules, extension points | Reconciliation of both lineages |
| `FORMAT.md` | Data model rationale: hashing, annotations, evidence, licensing | Reconciliation |
| `SKILL.md` | Day-to-day agent instructions (search / replicate / test / submit / contribute / propose tests) | Reconciliation |
| `AGENTS.md` | This file — orientation + collected requirements | Consolidation (see §3–§7) |
| `CONTRIBUTING.md` | How humans and agents add work; PR conventions | Portability pass |
| `ROADMAP.md` + `openspec/changes/` | How the project was built; enhancement queue | Build trail |
| `specs/AI_Recipe_Registry_SPEC_v0.3.md` | Founding amendment (annotations, objectives, evidence model) | Reference, repo-root `specs/` |

**Task for the next design pass:** none outstanding — the two lineages
are reconciled; report any remaining drift as an issue.

### How the data flows (architecture, one paragraph)

Source of truth is the git tree: `recipes/*.yaml` (pinned, annotated
configs), `results/<id>/<hash>/<run_id>.json` (immutable evidence),
`tests/definitions/*.json` (semver'd tests). One compute engine —
`tools/registry.py` — derives three generated stores from that source:
`index.json` (flat model rows), `runs.json` (all runs), `models/<id>.json`
(per-model documents); `tools/leaderboard.py` renders the markdown board
(`LEADERBOARD.md`, `models/<id>.md`) from the same structures. Nothing
derived is hand-edited; the pre-commit hook and CI regenerate and diff
all of it on every change. Query layer: `Store` (`tools/registry.py`) for
scripts/agents, `tools/search.py` + `tools/leaderboard.py --flags` for
humans. Writers go through tools only (`collect.py`, `submit_result.py`,
PRs for tests/docs). Build history and planned enhancements: `ROADMAP.md`
+ `openspec/changes/`.

## 3. Collected requirements (from the project owner + specs)

1. **Agents install a skill** that ships with this repo (clone first, or
   the skill is the repo).
2. **Search before you act** — agents operate tools to find existing
   recipes; no duplicate experiments without checking.
3. **Replicate, don't improvise** — a recipe must be reproducible by a
   plain `bash`/`fish` (more flavours later) script that installs/serves
   everything. When trying a variant, fork a new recipe with lineage.
4. **Standard tests, machine-runnable** — agents run tests in a standard
   manner, then commit experiment + aggregate results (an admin or the
   leaderboard can aggregate later).
5. **Failures are first-class** — agents add both failed and successful
   recipes; a failure with a reason beats silent repetition.
6. **Full context per recipe** — goals, features, results, *why each
   parameter was chosen*, and what the user/bot was trying to do.
7. **Recipe ↔ model, not quant** — recipes target a specific model, NOT
   a specific quant. Quants are a dimension to try/test under a recipe.
8. **Multi-parent lineage** — recipes inherit from 1..n parents (e.g.
   vision recipe + MCP recipe → merged recipe), like the model family
   trees in genomics.
9. **One PR per experiment** — agents submit results/experiments as
   clean, reviewable PRs.
10. **Summaries / leaderboards** — agents have tools to run summaries and
    leaderboards so humans (and agents) know what is worth trying.
11. **Multi-person validation** — recipes validated by multiple people;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mritzco/strixhalo-recipes](https://github.com/mritzco/strixhalo-recipes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
