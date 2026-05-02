---
trigger: always_on
description: This directory contains life sciences **skills** in two flavors: **code-centric** (pipeline/toolkit/database) and **prose-centric** (guide).
---

# Skills — Workflow Guide

This directory contains life sciences **skills** in two flavors: **code-centric** (pipeline/toolkit/database) and **prose-centric** (guide).
Claude Code uses this guide to author new entries when given a topic.

## Directory Layout

```
├── CLAUDE.md              ← you are here
├── registry.yaml          ← index of all entries
├── templates/
│   ├── SKILL_TEMPLATE.md          ← Pipeline-style skills (code-centric)
│   ├── SKILL_TEMPLATE_TOOLKIT.md  ← Toolkit-style skills (code-centric)
│   └── SKILL_TEMPLATE_PROSE.md    ← Guide-style skills (prose-centric)
└── skills/                ← all entries (SKILL.md per entry)
    ├── molecular-biology/
    ├── genomics-bioinformatics/
    ├── proteomics-protein-engineering/
    ├── structural-biology-drug-discovery/
    ├── systems-biology-multiomics/
    ├── cell-biology/
    ├── biostatistics/
    ├── data-visualization/
    ├── lab-automation/
    ├── scientific-computing/
    └── scientific-writing/
```

---

## Workflow: Topic → Entry (5 Steps)

When given a topic (e.g., "CRISPR guide RNA design"), follow these steps:

### Step 1. Classify — Code-centric vs Prose-centric, then Sub-type

#### 1a. Code-centric vs Prose-centric

All entries are Skills (SKILL.md). Choose the content style:

| Criteria | → Code-centric (pipeline/toolkit/database) | → Prose-centric (guide) |
|----------|---------------------------------------------|------------------------|
| Primary content | Executable code, pipelines, tool usage | Concepts, decision frameworks, best practices |
| Code blocks | 3+ substantial, runnable examples | Optional; illustrative only |
| User action | "Run this analysis" | "Understand this domain" |
| Example | "DESeq2 differential expression pipeline" | "When to use bulk vs single-cell RNA-seq" |

**Rule of thumb**: If the entry's core value is *running code*, it's code-centric. If it's *making informed decisions*, it's prose-centric (guide).

#### 1b. Sub-type

For code-centric entries, classify the sub-type (pipeline/toolkit/database). For prose-centric entries, the sub-type is always `guide`.

| Sub-type | Characteristics | Examples |
|----------|----------------|----------|
| **Pipeline** | Input→processing→output linear flow; one analysis process | scanpy, AutoDock Vina, DESeq2, STAR aligner |
| **Toolkit** | Collection of independent functional modules; multiple use-cases | RDKit, matplotlib, pandas, BioPython, scikit-learn |
| **Database** | API wrapper for external database queries; search/retrieve pattern | PubChem, KEGG, ChEMBL, UniProt, Ensembl via gget |

**Decision question**: "Can this tool be explained as **one pipeline** (load→process→output), does it require describing **multiple independent modules**, or is it primarily an **API/database accessor**?"

- One pipeline → **Pipeline** → use `templates/SKILL_TEMPLATE.md`
- Multiple modules → **Toolkit** → use `templates/SKILL_TEMPLATE_TOOLKIT.md`
- API/database queries → **Database** → use `templates/SKILL_TEMPLATE_TOOLKIT.md` with database adaptations
- Prose-centric guide → **Guide** → use `templates/SKILL_TEMPLATE_PROSE.md`

> **Type-specific adaptations** (Database, ML model, Model zoo, Platform integration, Non-Python, Visualization, Hardware/protocol, Data infrastructure, Document generation, Hybrid cases, Cross-cutting tools): see `references/tool-type-adaptations.md`

### Step 2. Choose Category

Pick the best-fit category directory under `skills/`:

| Category | Scope |
|----------|-------|
| `molecular-biology` | PCR, cloning, CRISPR, gene expression, central dogma, gene regulation |
| `genomics-bioinformatics` | NGS, alignment, variant calling, RNA-seq, genome architecture |
| `proteomics-protein-engineering` | Mass spec (proteomics AND metabolomics), protein design, structure prediction |
| `structural-biology-drug-discovery` | Docking, virtual screening, ADMET, drug design principles |
| `systems-biology-multiomics` | Pathway analysis, multi-omics integration, network biology |
| `cell-biology` | Imaging, flow cytometry, cell culture analysis, digital pathology |
| `biostatistics` | Statistical tests, experimental design, power analysis, study design |
| `data-visualization` | Plotting libraries, figure generation, scientific graphics |
| `lab-automation` | Robotics, LIMS, automated protocols |
| `scientific-computing` | General-purpose math/computation: symbolic math, numerical methods, MATLAB, data infrastructure, ML tools, geospatial, EDA, reproducibility |
| `scientific-writing` | Paper structure, figure design, peer review, research ideation and brainstorming, presentation skills |

**Cross-domain entries**: When an entry spans all scientific domains, place it in the category that best matches the entry's **primary audience**. Note the cross-domain nature in the description field. If no category fits well, prefer `scientific-computing` as the catch-all.

### Step 3. Gather Reference Material

Choose reference sources based on the entry type:

**For code-centric Skills** (primary — use these first):

| Source | When to use | URL pattern |
|--------|-------------|-------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jaechang-hits/SciAgent-Skills](https://github.com/jaechang-hits/SciAgent-Skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
