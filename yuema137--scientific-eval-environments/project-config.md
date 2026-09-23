---
trigger: always_on
description: The maintainer constitution is canonical in [`AGENT.md`](./AGENT.md). Read it before making changes.
---

# CLAUDE.md

The maintainer constitution is canonical in [`AGENT.md`](./AGENT.md). Read it before making changes.

## Layer model

There are **three co-equal aggregation axes** over the works layer: topics (methodology), domains (field), and activities (research task).

- **`works/`** — flat directory, one Markdown file per work. Factual references only.
  Template: Overview, **Topics** (metadata block), **Activities** (metadata block), Links, Summary, Tasks, Domains, Evaluation, Typical Duration, Main Contribution, Key Design Ideas, Strengths, Limitations, Related Works. **No** "Gap to Our Work" or positioning sections.
- **`topics/`** — literature reviews. Each topic owns its own comparison table. No global matrix.
- **`domains/`** — the **field axis**, a full knowledge layer co-equal with topics. Pages hold Scope + a Comparison table with fixed columns (`Work | Year | Scientific problem | Task form & scale | Domain verification | Card`, identical on every domain page, every cell verifiable from the card) + a Capability Matrix (fixed columns, split into coverage `Net E2E Cost MM Repro Real Inter` → **`Cov`** and rigor `Human Rubric Contam Verif Scale Fail` → **`Rig`**, rows ordered by `Cov` descending, then `Rig` descending within equal coverage — never by their sum, whose maxima differ (7 against 13) so that summing silently weights rigor higher; yes/no columns take `✔`/`✘`/`◐`/`?` and are ordered rarest-first, `Domain` carries per-page subfield abbreviations, `Verif`/`Scale` are graded `0`–`3` and `Fail` `0`–`4`; the two scores are summed **separately** because coverage and rigor pull against each other, and neither is a quality ranking; built from the card and then the paper, never from the Comparison row; `?` is a verification backlog, not a `✘`; updated **incrementally**, one row per new card, never re-derived wholesale) + bare Related Works links. No methodology synthesis or open questions — those stay in topics.
- **`activities/`** — the **research-activity / task axis** (what the agent actually does), a full knowledge layer co-equal with topics and domains. Pages hold Definition + Scope + a Task-Patterns synthesis + a Comparison table with fixed columns (`Work | Year | Activity instantiation | Task form / environment | Deliverable or success target | Card`) + bare Related Works links.

"Works" is broader than "benchmarks" — the layer holds cards for benchmarks, methodologies, evaluation frameworks, surveys, position papers, and evaluation-focused RL contributions on agents. Non-benchmark works fill inapplicable sections with `N/A` and a short note.

**Navigation flows Topic → Work → Paper, Domain → Work → Paper, and Activity → Work → Paper.** Topics (evaluation-research axis), domains (field axis), and activities (task axis) are three co-equal primary entry points; cards are references linked from all three.

**Topics and activities are not mutually exclusive.** A work may belong to multiple topics and multiple activities — each is a different perspective, not a unique category. Cross-membership is the intended pattern, not an exception.

A work card lists its topics in the `Topics` block and its activities in the `Activities` block; each topic/activity page lists its works in `Related Works`. These redundant mappings are the internal index for keeping the layers in sync. The `Activities` block is **mandatory on every card**: applicable works link one or more canonical activities; genuinely non-applicable works (surveys, pure methodology, general-purpose or safety/resource probes) carry an explicit `N/A — <reason>` and appear on no activity page. Assign activities conservatively (typically 1–3), from evidence in the card, never from title keywords.

## Canonical topic taxonomy (fixed)

| # | Topic | File |
|---|---|---|
| I | General Long-Horizon Agent Benchmarks | `long_horizon_evaluation.md` |
| II | Scientific Agent Benchmarks | `scientific_agents.md` |
| III | Planning & Decision-Making Evaluation | `planning_decision_evaluation.md` |
| IV | Hierarchical Decision Abstraction | `hierarchical_decision_abstraction.md` |
| V | Trajectory Evaluation | `trajectory_evaluation.md` |
| VI | Skill Hierarchy | `skill_hierarchy.md` |
| VII | Credit Assignment | `credit_assignment.md` |
| VIII | Resource-aware Evaluation | `resource_aware_evaluation.md` |
| IX | Evaluator Reliability & Validation | `evaluator_reliability_validation.md` |
| X | Benchmark Design, Validity & Contamination | `benchmark_design_validity_contamination.md` |
| XI | Skill Learning & Evolution | `skill_learning_evolution.md` |
| XII | Agent Harnesses & Scaffolding | `agent_harnesses_scaffolding.md` |
| XIII | Evaluation-Driven Data Curation | `evaluation_driven_data_curation.md` |
| XIV | Evaluation-Driven Post-Training | `evaluation_driven_post_training.md` |
| XV | Survey | `survey.md` |

Skill Hierarchy and Credit Assignment are independent topics — do not merge.

## Explanation style


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yuema137/scientific-eval-environments](https://github.com/yuema137/scientific-eval-environments) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
