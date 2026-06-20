---
trigger: always_on
description: End-to-end automated SCI paper writing for deep learning, machine learning, computer vision, NLP, multimodal learning, and related AI research. Chinese-first pipeline: raw project materials → structured digest → literature review → experiment analysis → polished English manuscript. v0.4 adds statistical testing, architecture extraction, citation network analysis, and result visualization. Use when the user wants to write a complete SCI paper from code, notes, experiment tables, framework diagram
---


# AI SCI Paper Writer v0.4.0

## Mission

One-stop pipeline that takes raw research materials and produces a polished English SCI manuscript. Default to Chinese-first writing unless the user explicitly asks for direct English drafting.

## Pipeline Overview

```
STAGE 0: INIT     → Inventory materials, create project state file
STAGE 1: DIGEST   → Extract paper-ready facts from project materials
STAGE 2: LIT      → Search, verify, organize literature
STAGE 3: EXPER    → Analyze experiments, compute improvements, validate claims
STAGE 4: WRITE    → Chinese draft → Chinese polish → EN conversion → EN polish → Self-critique → Template Rendering
```

Each stage saves its output to a shared project state file. The pipeline can be paused and resumed at any stage.

## Getting Started

When the user provides research materials, always begin at Stage 0. Ask for the project output directory where state files will be saved.

If the user points to an existing `project-state.md` file, read the `**Status**` field and resume from that stage.

---

## STAGE 0: INIT — Material Inventory & State Setup

### Purpose
Inventory all user-provided materials and create a project state file that tracks the pipeline.

### Actions
1. Ask the user for: project output directory, target venue/journal, and any specific requirements.
2. Identify every provided material: code repos, README, configs, logs, experiment tables (CSV/Excel), framework diagrams, notes (Word/TXT/Markdown), prior drafts, literature lists.
3. Create `<output_dir>/project-state.md` with this structure:

```markdown
# Project State — [Paper Title or Placeholder]

**Status**: STAGE 0 | INIT
**Created**: YYYY-MM-DD
**Last Updated**: YYYY-MM-DD

## Available Materials

| # | Type | Path/Description | Notes |
|---|------|-----------------|-------|
| 1 |       |                  |       |

## Target Venue & Requirements

## Pipeline Progress

- [ ] Stage 1: DIGEST
- [ ] Stage 2: LIT
- [ ] Stage 3: EXPER
- [ ] Stage 4: WRITE

## Stage Outputs

### Stage 1 Output (DIGEST)
*(empty — run Stage 1 to populate)*

### Stage 2 Output (LIT)
*(empty)*

### Stage 3 Output (EXPER)
*(empty)*

### Stage 4 Output (WRITE)
*(empty)*

## Missing Author Inputs
*(populated across stages)*

## Paper Storyline
*(populated after Stage 1)*
```

4. Update `**Status**` to `STAGE 1 | DIGEST`. Advance to Stage 1.

---

## STAGE 1: DIGEST — Project Digestion

### Purpose
Extract paper-ready facts from project materials. Produce a structured project brief.

### Required Inputs
- All materials listed in the project state file

### References
Read these files for detailed guidance:
- `digest/references/code-reading-guide.md` — code/material inspection methodology
- `digest/references/cv-nlp-task-taxonomy.md` — task classification
- `digest/references/method-evidence-rules.md` — evidence-grounding rules
- `digest/references/project-brief-template.md` — output template

### Actions
1. Read all available materials thoroughly.
2. Extract: research task, problem motivation, method pipeline, core modules, datasets, baselines, metrics, training/inference flow.
3. Link every method claim to concrete evidence from the materials.
4. Propose contribution candidates. Distinguish evidence-backed facts from interpretation.
5. Run `digest/scripts/summarize_repo.py <repo_path>` for a file inventory.
6. Run `digest/scripts/extract_architecture.py <repo_path>` for deep code analysis: extract nn.Module subclasses, loss functions, hyperparameters, framework detection, and training infrastructure.
7. If Jupyter notebooks (`.ipynb`) exist, run `digest/scripts/parse_notebooks.py <notebooks>` to extract model definitions, training loops, and results tables from notebook cells (NEW in v0.4).
8. Run `digest/scripts/trace_dependencies.py <repo_path> --entry <entry_point>` to build import graph, trace data flow, and identify core pipeline modules (NEW in v0.4).
9. Run `digest/scripts/synthesize_brief.py --arch <arch_report> --deps <dep_report> --notebooks <nb_report> --output <output_dir>/00_project_brief.md` to auto-generate the complete project brief, or use `--repo <path>` to run the full digest pipeline end-to-end (NEW in v0.4).

### Prompt Guidance

When performing the digest, use this chain-of-thought:

```
STEP 1: TASK IDENTIFICATION
- What is the research task? (classification / detection / segmentation / generation / retrieval / ...)
- What is the input modality? (image / text / audio / video / multimodal)
- What is the output? (class label / mask / bounding box / text / embedding)

STEP 2: METHOD EXTRACTION
- What is the backbone? (ResNet / ViT / BERT / GPT / custom)
- What are the novel modules? Identify every nn.Module subclass or custom function.
- What is the training objective? List every loss term.
- What is the inference pipeline? Describe step-by-step.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NGIWS830/AI-SCI-SKILLs](https://github.com/NGIWS830/AI-SCI-SKILLs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
