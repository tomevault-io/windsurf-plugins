---
trigger: always_on
description: AI-assisted meta-analysis pipeline. This file is auto-loaded by Claude Code.
---

# Agent Instructions

AI-assisted meta-analysis pipeline. This file is auto-loaded by Claude Code.

---

## Quick Start

**First time setup?** → Run `bash setup.sh` then `bash verify_environment.sh` (see [ENVIRONMENT_QUICK_START.md](ENVIRONMENT_QUICK_START.md))
**New project?** → Say "brainstorm" or "help me find a topic" (see [ma-topic-intake](ma-topic-intake/SKILL.md))
**Have TOPIC.txt?** → Say "start" or "continue" (see [ma-end-to-end](ma-end-to-end/SKILL.md))
**At Stage 06?** → Say "complete manuscript" (see [ma-manuscript-quarto](ma-manuscript-quarto/SKILL.md))
**Want to see an example?** → Check `projects/ici-breast-cancer/` (99% complete meta-analysis)
**Want to validate the workflow?** → See [metadat Validation](ma-end-to-end/references/metadat-validation.md)

---

## Example: Completed Meta-Analysis

**Location**: `projects/ici-breast-cancer/`

A **real, 99% complete meta-analysis** on immune checkpoint inhibitors in triple-negative breast cancer (TNBC):

**Key Metrics**:

- 5 RCTs, N=2,402 patients
- Primary outcome: RR 1.26 (95% CI 1.16-1.37), p=0.0015, ⊕⊕⊕⊕ HIGH quality
- Manuscript: 4,921 words (compliant with Lancet Oncology, JAMA Oncology)
- Time invested: ~14 hours (vs 100+ hours manual)

**Quick Tour**:

1. `projects/ici-breast-cancer/README.md` - Complete navigation guide
2. `projects/ici-breast-cancer/00_overview/FINAL_PROJECT_SUMMARY.md` - All findings (415 lines)
3. `projects/ici-breast-cancer/07_manuscript/` - Full manuscript (5 sections + 7 tables)
4. `projects/ici-breast-cancer/06_analysis/` - All R scripts + results

**Use this as a template** when starting your own meta-analysis.

---

## 📁 Project Structure (IMPORTANT)

**All projects are now in `projects/<project-name>/` directory.**

```
meta-pipe/
├── ma-*/                    # Framework code modules (each has SKILL.md)
├── docs/archive/            # Archived documentation
├── tooling/                 # Shared tools and scripts
└── projects/                # All your meta-analysis projects
    ├── legacy/              # Historical data (migrated 2026-02-08)
    ├── ici-breast-cancer/   # Example: complete meta-analysis
    └── your-project/        # Your new projects go here
        ├── 01_protocol/
        ├── 02_search/
        ├── ...
        └── TOPIC.txt
```

**When running commands**: Replace `<project-name>` with your actual project name.

---

## Pipeline Stages & Skills

**Each stage has a dedicated skill with commands and workflow guidance**

| Stage | Skill                       | Key Tasks                           | Invoke                           |
| ----- | --------------------------- | ----------------------------------- | -------------------------------- |
| 00    | `/ma-topic-intake`          | Brainstorming, feasibility checks   | `/brainstorm` or use skill       |
| 01-02 | `/ma-search-bibliography`   | PROSPERO, search, dedupe            | Use skill for detailed commands  |
| 03    | `/ma-screening-quality`     | Dual-review screening, kappa        | Use skill for detailed commands  |
| 03b   | `/ma-screening-quality`     | **Analysis type confirmation gate** | Confirm NMA vs pairwise (Step 8) |
| 04    | `/ma-fulltext-management`   | PDF retrieval, Unpaywall            | Use skill for detailed commands  |
| 04b   | `/ma-fulltext-management`   | **Full-text eligibility screening** | `ai_screen.py --stage fulltext`  |
| 05    | `/ma-data-extraction`       | Data extraction, RoB assessment     | Use skill for detailed commands  |
| 06a   | `/ma-meta-analysis`         | Pairwise MA (R scripts 01-12)       | Use skill for detailed commands  |
| 06b   | `/ma-network-meta-analysis` | NMA (R scripts nma_01-10)           | Use skill for detailed commands  |
| 07    | `/ma-manuscript-quarto`     | Manuscript assembly, rendering      | Use skill for detailed commands  |
| 08    | `/ma-peer-review`           | GRADE assessment, SoF table         | Use skill for detailed commands  |
| 09    | `/ma-publication-quality`   | QA, overclaim audit, readiness      | Use skill for detailed commands  |
| 10    | `/ma-submission-prep`       | PROSPERO, final checks, submit      | Use skill for detailed commands  |

**Orchestration**: `/ma-end-to-end` - Complete workflow management | `/ma-agent-teams` - Agent team coordination

**Share your work**: `/post-to-discussion` - Post your completed project to [GitHub Discussions](https://github.com/htlin222/meta-pipe/discussions) with figures and results

**Note**: Skills are invoked using the `Skill` tool. Each skill contains both workflow guidance and complete command references.

---

## Agent Teams (Experimental)

Coordinate multiple Claude Code instances for parallel meta-analysis pipeline work.

### Quick Commands

- "Create a team for project X" → Full pipeline team (all stages)
- "Parallel screen project X" → Dual-review screening team only
- "Analysis team for project X" → Statistician + manuscript writer + QA auditor

### How It Works

- Lead reads `/ma-agent-teams` skill for orchestration playbook
- Teammates spawned with role-specific prompts from `ma-agent-teams/prompts/`
- Shared task list tracks dependencies; hooks enforce quality gates
- Each teammate owns specific directories (no cross-teammate file writes)

### Generate Spawn Prompts

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [htlin222/meta-pipe](https://github.com/htlin222/meta-pipe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
