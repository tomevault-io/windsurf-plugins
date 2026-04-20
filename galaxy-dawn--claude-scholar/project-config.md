---
trigger: always_on
description: **Claude Scholar** - Semi-automated research assistant for academic research and software development
---

# Claude Scholar Configuration

## Project Overview

**Claude Scholar** - Semi-automated research assistant for academic research and software development

**Mission**: Support Claude Code, OpenCode, and Codex CLI across ideation, coding, experiments, writing, publication, plugin development, and project management.

---

## User Background

### Academic Background
- **Degree**: Computer Science PhD
- **Target Venues**:
  - Top conferences: NeurIPS, ICML, ICLR, KDD
  - High-impact journals: Nature, Science, Cell, PNAS
- **Focus**: Academic writing quality, logical coherence, natural expression

### Tech Stack Preferences

**Python Ecosystem**:
- **Package manager**: `uv` - modern Python package manager
- **Config management**: Hydra + OmegaConf (config composition, overrides, type safety)
- **Model training**: Transformers Trainer

**Git Standards**:
- **Commit convention**: Conventional Commits
  ```
  Type: feat, fix, docs, style, refactor, perf, test, chore
  Scope: data, model, config, trainer, utils, workflow
  ```
- **Branch strategy**: master/develop/feature/bugfix/hotfix/release
- **Merge strategy**: rebase for feature branch sync, merge --no-ff for integration

---

## Global Configuration

### Language Settings
- **Respond in English to the user**
- Keep technical terms in English (e.g. NeurIPS, RLHF, TDD, Git)
- Do not translate proper nouns or names

### Working Directory Standards
- Plan documents: `/plan` folder
- Temporary files: `/temp` folder
- Auto-create folders if they don't exist

### Task Execution Principles
- Discuss approach before breaking down complex tasks
- Run example tests after implementation
- Make backups, avoid breaking existing functionality
- Clean up temporary files after completion

### Work Style
- **Task management**: Use TodoWrite to track progress, plan before executing complex tasks, prefer existing skills
- **Communication**: Ask proactively when uncertain, confirm before important operations, follow hook-enforced workflows
- **Code style**: Python follows PEP 8, comments in English, identifiers in English

---

## Core Workflows

### Research Lifecycle (7 Stages)

```
Ideation → ML Development → Experiment Analysis → Paper Writing → Self-Review → Submission/Rebuttal → Post-Acceptance
```

| Stage | Core Tools | Commands |
|-------|-----------|----------|
| 1. Research Ideation | `research-ideation` skill + `literature-reviewer` agent + Zotero MCP | `/research-init`, `/zotero-review`, `/zotero-notes` |
| 2. ML Project Dev | `architecture-design` skill + `code-reviewer` agent | `/plan`, `/commit`, `/tdd` |
| 3. Experiment Analysis | `results-analysis` skill + `results-report` skill | `/analyze-results` |
| 4. Paper Writing | `ml-paper-writing` skill + `paper-miner` agent | `/mine-writing-patterns` |
| 5. Self-Review | `paper-self-review` skill | - |
| 6. Submission & Rebuttal | `review-response` skill + `rebuttal-writer` agent | `/rebuttal` |
| 7. Post-Acceptance | `post-acceptance` skill | `/presentation`, `/poster`, `/promote` |

### Supporting Workflows

- **Automation**: 5 Hooks auto-trigger at session lifecycle stages (skill evaluation, env init, work summary, security check)
- **Zotero Integration**: Automated paper import, collection management, full-text reading, and citation export via Zotero MCP
- **Obsidian Knowledge Base**: Built-in filesystem-first project knowledge base for literature, plans, daily logs, experiments, results, writing, and archive management, with a compact vault structure and no MCP requirement
- **Knowledge Extraction**: `paper-miner` and `kaggle-miner` agents continuously extract knowledge from papers and competitions
- **Skill Evolution**: `skill-development` → `skill-quality-reviewer` → `skill-improver` three-step improvement loop

### Obsidian Project Knowledge Base Rule

- If the current repository contains `.claude/project-memory/registry.yaml`, automatically activate `obsidian-project-memory` and treat Obsidian as the default project knowledge base for this repo.
- If the repository is not yet bound but looks like a research project, automatically activate `obsidian-project-bootstrap` and import it into the vault.
- On every substantial project turn, update at least the daily note and the repo-local project memory file; touch `00-Hub.md` only when top-level project status really changes.
- Never require any extra Obsidian API configuration or API keys for this workflow.

---

## Skills Directory (47 skills)

### 🔬 Research & Analysis (5 skills)

- **research-ideation**: Research startup (5W1H, literature review, gap analysis, research question formulation, Zotero integration)
- **results-analysis**: Strict experiment analysis (rigorous statistics, scientific figures, ablation studies)
- **results-report**: Complete post-experiment summary reporting (retrospection, decision support, Obsidian results reports)
- **citation-verification**: Citation verification (multi-layer: format→API→info→content)
- **daily-paper-generator**: Daily paper generator for research tracking

### 📝 Paper Writing & Publication (7 skills)

- **ml-paper-writing**: ML/AI paper writing assistance
  - Top conferences: NeurIPS, ICML, ICLR, ACL, AAAI, COLM
  - Journals: Nature, Science, Cell, PNAS

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Galaxy-Dawn/claude-scholar](https://github.com/Galaxy-Dawn/claude-scholar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
