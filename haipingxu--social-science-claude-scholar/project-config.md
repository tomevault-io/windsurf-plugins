---
trigger: always_on
description: **Claude Scholar** - Personal Claude Code configuration system for academic research and software development
---

# Claude Scholar Configuration

## Project Overview

**Claude Scholar** - Personal Claude Code configuration system for academic research and software development

**Mission**: Cover the complete academic research lifecycle (from ideation to publication) and software development workflows, with plugin development and project management capabilities.

---

## User Background

### Academic Background
- **Degree**: PhD in Social Science (Political Science / Economics)
- **Fields**: Comparative Politics, Political Economy, Political Institutions, Public Economics, Economic History
- **Methods**: Causal inference econometrics (DID, IV, RDD, Synthetic Control), Text as Data / NLP, Applied Machine Learning, Applied Game Theory, Qualitative Historical Comparative Methods
- **Target Venues**:
  - Political Science: APSR, AJPS, JOP, Comparative Political Studies (CPS), International Organization (IO), World Politics
  - Economics: AER, QJE, JPE, REStud, Econometrica, Journal of Public Economics (JPubE)
  - Cross-disciplinary: PNAS, Journal of Conflict Resolution, Political Analysis
- **Focus**: Credible causal identification, rigorous empirical analysis, clear theoretical motivation, precise academic writing

### Tech Stack Preferences

**Primary Tools**:
- **Writing**: LaTeX (primary), Overleaf for collaboration
- **Statistical analysis**: Stata (current primary) → Python (transitioning)
- **Package manager**: `uv` - modern Python package manager

**Python Ecosystem (transitioning from Stata)**:
- **Causal inference**: `statsmodels`, `linearmodels`, `econml`, `doubleml`, `causalml`
- **Data**: `pandas`, `numpy`, `pyarrow`
- **NLP / Text as Data**: `transformers`, `gensim`, `spacy`, `bertopic`
- **Visualization**: `matplotlib`, `seaborn`, `plotnine` (ggplot2 equivalent)
- **Stata bridge**: `pystata`, `stata_setup`

**Git Standards**:
- **Commit convention**: Conventional Commits
  ```
  Type: feat, fix, docs, style, refactor, perf, test, chore
  Scope: data, analysis, paper, config, utils, workflow
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
- **Operation Logging**: For any session involving 3+ tool calls or file modifications, maintain a running session log

### Operation Log Format

Log file location: `~/.claude/session-logs/YYYY-MM-DD.md`
Auto-create the file and directory if they don't exist. Append entries throughout the session.

```markdown
## Session: YYYY-MM-DD HH:MM

### [HH:MM] <Action summary>
- **Tool**: Write / Edit / Bash / WebFetch / etc.
- **Target**: `path/to/file` or `command run`
- **Outcome**: Success / Failed / Partial
- **Notes**: Any important context or decision rationale

### [HH:MM] <Next action>
...
```

Log every: file creation, file edit, bash command with side effects, key decisions. Skip: read-only file reads, web fetches that are purely informational.

### Work Style
- **Task management**: Use TodoWrite to track progress, plan before executing complex tasks, prefer existing skills
- **Communication**: Ask proactively when uncertain, confirm before important operations, follow hook-enforced workflows
- **Code style**: Python follows PEP 8, comments in English, identifiers in English

---

## Core Workflows

### Research Lifecycle (7 Stages)

```
Ideation → Literature Review → Data & Empirics → Paper Writing → Self-Review → Submission/Rebuttal → Post-Acceptance
```

| Stage | Core Tools | Commands |
|-------|-----------|----------|
| 1. Research Ideation | `research-ideation` skill + `gpt-researcher` skill + `literature-reviewer` agent + Zotero MCP | `/research-init`, `/zotero-review`, `/zotero-notes` |
| 2. Literature Review | `daily-paper-generator` skill + `gpt-researcher` skill + Zotero MCP | `/zotero-notes`, `/zotero-review` |
| 3. Data & Empirical Analysis | `results-analysis` skill + `data-analyst` agent | `/analyze-results` |
| 4. Paper Writing | `social-science-paper-writing` skill + `paper-miner` agent | - |
| 5. Self-Review | `paper-self-review` skill + `academic-paper-reviewer` skill | - |
| 6. Submission & Rebuttal | `review-response` skill + `rebuttal-writer` agent | `/rebuttal` |
| 7. Post-Acceptance | `post-acceptance` skill | `/presentation`, `/poster`, `/promote` |

### Supporting Workflows

- **Automation**: 7 Hooks auto-trigger at session lifecycle stages (skill evaluation, env init, work summary, security check, pre-compact save, post-compact restore)
- **Context Survival**: `pre-compact.py` saves state before context compression; `post-compact-restore.py` restores context on resume — plans and decisions survive session boundaries

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HaipingXu/social-science-claude-scholar](https://github.com/HaipingXu/social-science-claude-scholar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
