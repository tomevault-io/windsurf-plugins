---
trigger: always_on
description: This is a **Qiongli Zhengche** (`穷理证澈`) system: a contract-driven academic workflow covering the full lifecycle from literature review through manuscript production, compliance checking, submission, and presentation.
---

# CLAUDE.md — Qiongli

## Project Overview

This is a **Qiongli Zhengche** (`穷理证澈`) system: a contract-driven academic workflow covering the full lifecycle from literature review through manuscript production, compliance checking, submission, and presentation.

Canonical standard (cross-model):
- `standards/research-workflow-contract.yaml` — Task IDs, required outputs, quality gates
- `standards/mcp-agent-capability-map.yaml` — MCP tool mapping & primary/fallback agents
- `qiongli-workflow/references/workflow-contract.md` — **Task ID table** (A1–K4)
- `qiongli-workflow/references/platform-routing.md` — Task ID → workflow mapping
- Artifact root: `RESEARCH/[topic]/`

Local validation:
```bash
python3 scripts/validate_research_standard.py --strict
python3 -m unittest tests.test_orchestrator_workflows -v
python3 -m bridges.orchestrator doctor --cwd .
./scripts/install_qiongli.sh --target all --project-dir /path/to/project --doctor
```

## Quick Commands

```
/paper [topic] [venue]                # Master router — choose paper type + task ID
/lit-review [topic] [year range]     # Systematic literature review (PRISMA)
/paper-read [URL or DOI]             # Deep paper analysis
/find-gap [research area]            # Identify research gaps
/build-framework [theory/concept]    # Build theoretical framework
/academic-write [section] [topic]    # Academic writing assistance
/synthesize [topic] [outcome_id]     # Evidence synthesis / meta-analysis
/paper-write [topic] [type] [venue]  # Full manuscript drafting
/study-design [topic]                # Empirical study design
/ethics-check [topic]                # Ethics / IRB pack
/compliance-check [topic]            # Pre-submission compliance check (G1-G4)
/submission-prep [topic] [venue]     # Submission package
/rebuttal [topic]                    # Rebuttal / response to reviewers
/code-build [method] --domain ...    # Build academic research code
/proofread [topic]                   # AI de-trace / final proofreading
/academic-present [topic]            # Academic presentation preparation
```

For consistency, ask users for `paper_type + task_id` when using `/paper`.

## How Task-ID Routing Works

When a user requests a specific task (e.g. "I need to do A1_5 hypothesis generation"), use the `/paper` workflow as the master router:

1. Read `.agent/workflows/paper.md` → it maps **every task ID** (A1–K4) to the correct sub-workflow or skill card
2. Follow the routing. Examples:
   - `A1` → use `question-refiner` skill → output `RESEARCH/[topic]/framing/research_question.md`
   - `A3` → `/build-framework` workflow
   - `B1` → `/lit-review` workflow
   - `F3` → `/paper-write` workflow
   - `H1` → `/submission-prep` workflow
   - `K1` → `/academic-present` workflow
3. For detailed execution guidance on any task ID, read the corresponding **stage playbook**:
   - `qiongli-workflow/references/stage-A-framing.md` (tasks A1–A5)
   - `qiongli-workflow/references/stage-B-literature.md` (tasks B1–B6)
   - `qiongli-workflow/references/stage-C-design.md` (tasks C1–C5)
   - `qiongli-workflow/references/stage-D-ethics.md` (tasks D1–D3)
   - `qiongli-workflow/references/stage-E-synthesis.md` (tasks E1–E5)
   - `qiongli-workflow/references/stage-F-writing.md` (tasks F1–F6)
   - `qiongli-workflow/references/stage-G-compliance.md` (tasks G1–G4)
   - `qiongli-workflow/references/stage-J-proofread.md` (tasks J1–J4)
   - `qiongli-workflow/references/stage-H-submission.md` (tasks H1–H4)
   - `qiongli-workflow/references/stage-I-code.md` (tasks I1–I8)

## Skill Loading Strategy

Skills are organized in two tiers to optimize token usage:

### Default Mode (Token-Efficient)
Use `skills-core.md` for the consolidated skill reference (~10KB). Contains core purpose, process, and output format for each skill.

### Detailed Mode (Full Reference)
Load full skill files from `skills/[stage]/[skill-name].md` only when:
- First encounter with complex edge cases
- Need detailed output format templates
- Error recovery requiring fallback strategies

### Invocation Pattern
When a workflow says "Use the **skill-name** skill":
1. Check `skills-core.md` for the skill's core process
2. If sufficient: execute using core reference
3. If need detail: load `skills/[stage]/[skill-name].md` for full templates

### Skill Directory Structure

```
skills/
├── A_framing/       (question-refiner, hypothesis-generator, theory-mapper, gap-analyzer, venue-analyzer)
├── B_literature/    (academic-searcher, paper-screener, paper-extractor, citation-snowballer, fulltext-fetcher, citation-formatter, concept-extractor, literature-mapper, reference-manager-bridge)
├── C_design/        (study-designer, rival-hypothesis-designer, robustness-planner, dataset-finder, variable-constructor)
├── D_ethics/        (ethics-irb-helper, deidentification-planner)
├── E_synthesis/     (evidence-synthesizer, quality-assessor, publication-bias-checker)
├── F_writing/       (manuscript-architect, analysis-interpreter, effect-size-interpreter, table-generator, figure-specifier, meta-optimizer)
├── G_compliance/    (prisma-checker, reporting-checker, tone-normalizer)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jxpeng98/qiongli](https://github.com/jxpeng98/qiongli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
