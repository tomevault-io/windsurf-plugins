---
trigger: always_on
description: > This file is automatically read when you open this folder with Claude Code.
---

# Claude Code for Academic Research

> This file is automatically read when you open this folder with Claude Code.
> Customise it with your own details — see comments marked with `<!-- CUSTOMISE -->`.

## Before You Start

Read these context files to understand the user's situation:

1. `.context/profile.md` — Who you are, your roles, research areas
2. `.context/current-focus.md` — What you're working on NOW
3. `.context/projects/_index.md` — Overview of all projects

## Key Information

<!-- CUSTOMISE: Replace with your own details -->

**Who I am:**
- PhD researcher
- Multiple active research projects
- Teaching responsibilities

**Research areas:**
- [Your field 1]
- [Your field 2]
- [Your field 3]

**How I work:**
- Flexible/reactive style
- Prefer questions over lists
- Daily reviews work better than weekly
- Full context in task descriptions

## Quick Commands

<!-- QUICK-COMMANDS:START -->
<!-- synced from private CLAUDE.md — do not edit manually -->
Just say these naturally:

| You say | Claude does |
|---------|-------------|
| "Plan my day" | Reads context, queries vault, asks questions, creates a plan |
| "What should I work on?" | Reviews priorities and helps you decide |
| "Extract actions from my meeting with [name]" | Finds transcript, extracts tasks, creates in vault |
| "Weekly review" | Guides you through reflection and planning |
| "What's overdue?" | Queries vault tasks and summarises |
| "Update my research pipeline" | Shows paper status, helps update stages |
| "Find references on [topic]" | Academic search with verified citations |
| "What did I accomplish this week?" | Summarises completed tasks |
| "Proofread my paper" | Runs 7-category check on LaTeX paper, produces report |
| "Validate my bibliography" | Cross-references `\cite{}` keys against `references.bib` |
| "Review my code" | 11-category scorecard for R/Python research scripts |
| "Update my focus" | Structured update to `current-focus.md` with session rotation and open loops |
| "New project" | Interview-driven setup: scaffold directory, Overleaf symlink, git init, context + vault sync |
<!-- QUICK-COMMANDS:END -->

## Conventions

<!-- CONVENTIONS:START -->
<!-- synced from private CLAUDE.md — do not edit manually -->
### LaTeX Compilation
- **Default method:** Use `/latex` — it compiles, auto-fixes errors, and runs a citation audit.
- Build artifacts go to `out/`, but the PDF is copied back to the source directory.
- Use `.latexmkrc` with `$out_dir = 'out'` and `an `END {}` block to copy the PDF back`.
- Never leave build artifacts (`.aux`, `.log`, etc.) in the source directory.

### Python & Package Management
- Always use `uv`: see `python-uv` rule (global).

### R
- Use `<-` for assignment, not `=`.

### Git & Remote
- Remote verification, push safety, and deploy order: see `git-safety` rule (global).
- **Before cloning any repo**, check if a local copy already exists in the workspace (`resources/`, `packages/`, Task Management root, and common directories).
<!-- CONVENTIONS:END -->

### Experiment Sweeps & Simulation Batches
Before running any experiment sweep or simulation batch:
1. Write sanity-check assertions first.
2. Implement the code.
3. Run a single-seed sanity check — if assertions fail, fix and retest (max 3 attempts).
4. Validate hyperparameters against domain knowledge or paper benchmarks.
5. Only then proceed to full experiments.

### Output Formats
- Academic papers: LaTeX.
- Documents for human use (consent forms, PILs, etc.): `.docx` via `pandoc`.

### Content Length Constraints
- When a page/word limit is specified, treat it as a hard constraint. Draft to 80%, then expand — never exceed and trim.
- Always report the actual page/word count after drafting.

## Research Vault

<!-- RESEARCH-VAULT:START -->
<!-- CUSTOMISE: Point this to your own Obsidian-style markdown vault -->
The Research Vault (`~/Research-Vault`) stores all dynamic research data as markdown files with YAML frontmatter. The `taskflow` MCP server reads/writes these files.

| Directory | Content |
|-----------|---------|
| `tasks/` | Personal tasks (GTD-style) |
| `pipeline/` | Research papers (stages: Idea → Published) |
| `submissions/` | Submission events (dates, outcomes) |
| `atlas/` | Research topics (nested by theme) |
| `venues/` | Journals, conferences, rankings |
| `people/` | Collaborators, supervisors |
| `themes/` | Research themes |

IDs are filename slugs (e.g., `cancel-leap-water-in-rugby`), not integers.
<!-- RESEARCH-VAULT:END -->

## Workflows

<!-- WORKFLOWS-POINTER:START -->
<!-- synced from private CLAUDE.md — do not edit manually -->
Detailed instructions in `.context/workflows/`:
- `daily-review.md` — How to help with daily planning
- `meeting-actions.md` — How to extract action items (see also [`docs/guides/minutes.md`](docs/guides/minutes.md) for full meeting system architecture)
- `weekly-review.md` — Weekly reflection template
- `replication-protocol.md` — 4-phase protocol for replicating paper results
- Feedback loop (skill improvement pipeline): [`docs/feedback-loop.md`](docs/feedback-loop.md)
<!-- WORKFLOWS-POINTER:END -->

<!-- COMPONENTS:START -->
## Skills Available


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flonat/claude-research](https://github.com/flonat/claude-research) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
