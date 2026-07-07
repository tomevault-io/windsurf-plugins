---
trigger: always_on
description: Guidance for AI coding agents working in this repository.
---

# AGENTS.md

Guidance for AI coding agents working in this repository.

## Repository Overview

This repo is **ml-research-skills** — a collection of agent skills for ML researchers. Each skill lives under `skills/<skill-name>/` as an instruction bundle centered on `SKILL.md`, with optional `references/`, `scripts/`, `templates/`, and `agents/` directories.

This repo is **one pack** in the broader portable [Skill OS matrix](https://github.com/a-green-hand-jack/skill-os) for AI research workflows. The matrix has six sibling pack repos plus a hub repo: [`skill-os`](https://github.com/a-green-hand-jack/skill-os) (hub: kernel schema, install/repo-split contract, installer, registry), [`core-ops-skills`](https://github.com/a-green-hand-jack/core-ops-skills) (substrate), [`automation-skills`](https://github.com/a-green-hand-jack/automation-skills), [`paper-reading-skills`](https://github.com/a-green-hand-jack/paper-reading-skills), [`research-distillation-skills`](https://github.com/a-green-hand-jack/research-distillation-skills), [`quick-experiment-skills`](https://github.com/a-green-hand-jack/quick-experiment-skills), and `ml-research-skills` (this repo, full ML research lifecycle). Kernel schema and contracts are authoritative in `skill-os`; this pack owns its own kernel example (`schemas/skill-kernel/examples/ml-research.kernel.json`) and a sliced `profiles/profile-index.yaml`. Codex / Claude / future agents are adapter targets, not the workflow source of truth.

The repo also has shared project memory under `memory/`. **Before any substantial maintenance work, read `memory/BRIEFING.md`** (the ≤30-line compact snapshot) and `memory/current-status.md`. Update the smallest relevant board after durable workflow, routing, validation, or architecture decisions. Keep local sidecar artifacts under `.agent/sidecars/`; they are ignored and should not be committed unless explicitly sanitized and requested.

Visual documentation assets live under `asset/`. Before adding, replacing, or renaming a figure, read `asset/README.md`; keep filenames semantic, update README links, and refresh the figure inventory in `memory/evidence-board.md` plus `memory/current-status.md` when the visual system changes materially.

Recommended global install uses only the bootstrap skill:

```bash
npx skills add a-green-hand-jack/ml-research-skills -g -a codex claude-code -s ml-research-bootstrap -y
```

If the global agent must initialize new ML research project roots, also install:

```bash
npx skills add a-green-hand-jack/ml-research-skills -g -a codex claude-code -s project-init -y
```

Install the full collection only from an initialized ML research project root:

```bash
npx skills add a-green-hand-jack/ml-research-skills
```

To install one specific skill globally for both agents when explicitly needed:

```bash
npx skills add a-green-hand-jack/ml-research-skills -g -a codex claude-code -s remote-project-control -y
```

Full global installs are maintainer/debug-only because this full ML research lifecycle bundle can interfere with ordinary non-ML projects.

These files are primarily agent instructions and templates, not an application with automated runtime tests.

## Git Closeout Policy

For Git commits, pushes, branch operations, worktrees, merge/rebase/cherry-pick, lock-file errors, or sandbox/network Git failures, use `safe-git-ops`.

For routine branch pushes after preflight has identified the repo, remote, and branch, use:

```bash
project-push <repo> <remote> <branch>
```

Do not use raw `git push`, `git -C <repo> push`, `cd <repo> && git push`, or shell-wrapped push variants for routine closeout unless `project-push` is unavailable. `git -C <repo> ...` is still fine for inspection and non-push repo-local commands.

## Current Repository Structure

```text
ml-research-skills/
├── README.md
├── CLAUDE.md
├── AGENTS.md
├── asset/
│   ├── README.md
│   └── *.png
├── memory/
│   ├── project.yaml
│   ├── current-status.md
│   ├── decision-log.md
│   ├── action-board.md
│   ├── risk-board.md
│   ├── phase-dashboard.md
│   ├── claim-board.md
│   ├── evidence-board.md
│   ├── provenance-board.md
│   ├── handoff-board.md
│   ├── component-index.yaml
│   └── source-visibility-board.md
├── profiles/
│   ├── README.md
│   └── profile-index.yaml         (sliced to ml-research entry only)
├── schemas/
│   └── skill-kernel/
│       ├── README.md
│       ├── skill-kernel.schema.json
│       └── examples/
└── skills/
    ├── add-git-tag/
    │   └── SKILL.md
    ├── research-project-memory/
    │   ├── SKILL.md
    │   ├── references/
    │   └── templates/
    ├── research-idea-validator/
    │   ├── SKILL.md
    │   └── references/
    ├── literature-review-sprint/
    │   ├── SKILL.md
    │   └── references/
    ├── reference-library-manager/
    │   ├── SKILL.md
    │   ├── agents/
    │   ├── references/
    │   ├── scripts/
    │   └── templates/
    ├── reference-reading-summarizer/
    │   ├── SKILL.md
    │   ├── agents/
    │   ├── references/
    │   └── templates/
    ├── reference-project-synthesizer/
    │   ├── SKILL.md
    │   ├── agents/
    │   ├── references/
    │   └── templates/
    ├── reference-corpus-analyzer/
    │   ├── SKILL.md
    │   └── templates/
    ├── algorithm-design-planner/
    │   ├── SKILL.md

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [a-green-hand-jack/ml-research-skills](https://github.com/a-green-hand-jack/ml-research-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
