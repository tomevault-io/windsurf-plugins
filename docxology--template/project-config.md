---
trigger: always_on
description: This document provides documentation for the Research Project Template system, ensuring understanding of all functionality, configuration options, and operational procedures.
---

# 🤖 AGENTS.md - System Documentation

## 🎯 System Overview

This document provides documentation for the Research Project Template system, ensuring understanding of all functionality, configuration options, and operational procedures.

### 📄 Publication

**Title**: *A template/ approach to Reproducible Generative Research: Architecture and Ergonomics from Configuration through Publication*
**DOI**: [10.5281/zenodo.19139090](https://doi.org/10.5281/zenodo.19139090) · **Record**: [zenodo.org/records/19139090](https://zenodo.org/records/19139090)

`template/` applies Infrastructure as Code to the research lifecycle: version-controlled manuscripts, tests, and provenance, built through a ten-stage DAG. **Layer 1** (`infrastructure/`, **313** Python files under `infrastructure/` per `find infrastructure -name '*.py' | wc -l` on this tree) is separated from **Layer 2** (self-contained projects under `projects/`). Each directory carries `README.md` + `AGENTS.md`; infrastructure packages usually add `SKILL.md` for agent routing. Full paper, metrics, and claims: Zenodo record above and root [`README.md`](README.md).

### Documentation map

| Entry | Role |
| --- | --- |
| [`README.md`](README.md) | Human onboarding, badges, quick paths |
| [`CLAUDE.md`](CLAUDE.md) | Claude Code: commands, architecture, constraints |
| [`.github/README.md`](.github/README.md) | GitHub: CI overview, templates, Dependabot |
| [`.github/AGENTS.md`](.github/AGENTS.md) | Actions job names, coverage gates, branch protection hints |

## Learned User Preferences

- When a "Plan" / "Implementation Plan" file is attached, treat the existing todos as authoritative: do not recreate them, do not edit the plan file, mark each in_progress as you work, and do not stop until every todo is complete.
- Purge legacy / historical / outdated / deprecated narrative from docs, manuscripts, and code on every review pass; never leave superseded references behind.
- When pointing users to past Cursor agent transcripts, cite only **parent** transcript files (UUID link text with a short title, UUID without the `.jsonl` suffix in the path); do not cite or discuss subagent transcripts or their IDs.
- Prefer understated, semantically necessary wording in docs and names; trim hype adjectives such as "enhanced", "real", and "new" unless they change meaning.
- In manuscript introductions and related work, prefer building on, juxtaposing, and extending prior work over oppositional "against" framing; show, do not tell.

## Learned Workspace Facts

- Manuscript metrics, counts, and variables are auto-injected from per-project `output/data/manuscript_variables.json` at render time; never hand-author values that should be injected. The PDF cover DOI is a separate path: `publication.doi` in `projects/{name}/manuscript/config.yaml` is read by `infrastructure/rendering/_pdf_latex_helpers.py` and emitted as `\href{https://doi.org/<doi>}{DOI: <doi>}` on the title page.
- **Rotating Lean-toolchain project (e.g. `fep_lean`, when checked out under `projects/`)**: real Lean 4 + Mathlib via `lake build`, OpenGauss `gauss` CLI, and the Hermes LLM pipeline (no mocks); validate end-to-end through `./run.sh`. Combined-PDF link colours come from `hyperref` / `\hypersetup` in the project's `manuscript/preamble.md` (red `fepred` for link, URL, and citation colours). Infrastructure only injects red when it rewrites a `hidelinks` draft in the emitted `.tex`.
- For that rotating project, Hermes and per-topic Gauss markdown reports are built from the pipeline stage payload (`TopicRunResult.as_dict()`), not by re-reading SQLite artifacts; that dict must include the Hermes fields the reporter uses (`tokens_used`, `explanation`, `refined_lean_sketch`, `hermes_model`, `cache_hit`, `hermes_lean_compiles`) or aggregate and per-topic reports will show empty tokens and missing sections.
- Topic sketch sources of truth (when checked out) are the catalogue in the project's `scripts/catalogue_sketches.py` and `config/topics.yaml` (one `TopicEntry` per topic: `lean_sketch` + `latex_equations`); the project's `lean/FepSketches/` is for `Basic.lean`, `fep_all.lean`, and ephemeral `_verify_*` verifier files, not long-lived per-topic `.lean` files for the full catalogue. The manuscript emits a **unified** `09z_unified_formalism_catalogue.md` (B+C material juxtaposed per topic; PDF refs `sec:appendix_b_full_topic_lean_catalogue` and `sec:appendix_c_latex_equations` both resolve in that chapter). The project's `scripts/theorem_latex_signatures.py` drives display-math strings aligned with Lean. The pipeline may write LaTeX `equation` environments with `\label{eq:…}` for autonumbered cross-refs, not only `$$` display blocks.
- That project's manuscript-variables injector reads only `run_*/verification_manifest.json` artifacts and ignores `verify_*/` standalone re-verifications, so the canonical compile-rate cited in the abstract is the Hermes-refined run (e.g. `49/50` with one fallback) — standalone verifier-only runs reporting `50/50` do not propagate into manuscript metrics.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [docxology/template](https://github.com/docxology/template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
