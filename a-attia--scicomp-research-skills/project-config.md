---
trigger: always_on
description: provides defaults; projects own their specifics.
---

# scicomp-research-skills / AGENTS.md

**You are reading the root `AGENTS.md` of a shared agent-skills
repository.** Read this file first before doing anything else here or in
any project that references it.

This file follows the [agents.md](https://agents.md/) open standard.
Agent clients that look for other filenames (e.g. `CLAUDE.md`) read this
same content via symlinks created by `bin/install.sh`.

> **PROVISIONAL FRAMEWORK** (as of 2026-05-14): some skill content
> here is well-grounded (research-paper-writing, literature-survey,
> paper-skeleton template); other skill content is informed prediction
> from prior-art audits but **has not yet been validated by any real
> research-project session**. When a rule feels speculative or
> doesn't quite fit the situation, **surface that to the user
> explicitly** + append an entry to the project's
> `notes/agent_feedback.md` (per
> `agent-resource-discipline/references/persistent-memory.md`). See
> [`STATUS.md`](STATUS.md) at the repo root for the honest map of
> what is tested vs speculative.

---

## 1. What this repository is

This repository holds **agent skills and workflow templates for research
in scientific computing** -- covering both research **papers** (drafts,
literature surveys, reviewer responses) and research **software**
(libraries, codes, reproducibility infrastructure) in domains such as
computational PDEs, inverse problems, optimal experimental design,
uncertainty quantification, optimisation, and scientific machine learning.

The repository exists so that:

- **Conventions are defined once and inherited everywhere.** A
  per-project `AGENTS.md` is short and project-specific; the generic
  conventions live here as **skills** loaded on demand.
- **The same conventions work across multiple agent clients** -- OpenCode,
  Claude Code, Codex, Cursor, Aider, Gemini CLI, etc. Any client that
  reads markdown can consume this repository.
- **The same conventions work across multiple machines.** A canonical
  checkout at `~/.scicomp-research-skills/` on each machine is refreshed
  via `git pull`; one source of truth.
- **Updates are versioned.** Every change to a convention or skill is a
  commit with a message; `git log` shows when and why.

This repository **starts from upstream
[Master-cai/Research-Paper-Writing-Skills](https://github.com/Master-cai/Research-Paper-Writing-Skills)**
(MIT-licensed) and intentionally diverges to broaden scope. See
`ATTRIBUTION.md` for the full lineage.

## 2. Local layout (per machine)

Two checkouts of this repository exist on each machine:

- **Development checkout**: anywhere EXCEPT `~/.scicomp-research-skills/`
  (a common convention is to keep it under your usual code-projects
  directory).
  - This is where edits + commits happen.
  - Other research projects + agents on the machine **ignore** this
    checkout completely.
  - Push from here to the GitHub remote when changes are ready.
- **Canonical checkout**:
  `~/.scicomp-research-skills/`
  - Read-only from the user's perspective; refreshed via
    `~/.scicomp-research-skills/bin/refresh.sh`
    (or `git -C ~/.scicomp-research-skills pull --ff-only`).
  - This is the location that agents read from.
  - A pre-commit hook (in `.githooks/pre-commit`) refuses commits in this
    checkout, so accidental edits cannot be committed back.
  - Per-project `AGENTS.md` files reference paths like
    `~/.scicomp-research-skills/skills/<name>/SKILL.md`.

Set up the canonical checkout on a fresh machine via (try SSH first;
fall back to HTTPS if SSH keys are not configured for GitHub):

```bash
git clone git@github.com:a-attia/scicomp-research-skills.git ~/.scicomp-research-skills \
  || git clone https://github.com/a-attia/scicomp-research-skills.git ~/.scicomp-research-skills
~/.scicomp-research-skills/bin/install.sh
```

## 3. How agents should consume this repository

When an agent is given a project that references this repository, the
agent's reading order is:

1. Read this root `AGENTS.md` (you are here).
2. Read any skill the project's `AGENTS.md` directs you to. Skills are
   loaded **on demand**, not all at once -- see Section 5.
3. Read the project's own `AGENTS.md` for project-specific overrides /
   facts.
4. Read the project's `PLAN.md` (or equivalent plan-of-record) if one
   exists; this is typically the project's content contract.
5. Then proceed with the user's request.

**Universal rule**: when in doubt, the project's `AGENTS.md` and `PLAN.md`
override any conflicting guidance from this repository. This repository
provides defaults; projects own their specifics.

### OpenCode-specific consumption

OpenCode supports referencing remote instructions natively via
`opencode.json`:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "instructions": [
    "https://raw.githubusercontent.com/a-attia/scicomp-research-skills/main/AGENTS.md"
  ]
}
```

Skill files are auto-discovered by OpenCode at
`~/.config/opencode/skills/<name>/SKILL.md` and the Claude-compatible
fallback `~/.claude/skills/<name>/SKILL.md`. The simplest deployment is
a symlink:

```bash
ln -s ~/.scicomp-research-skills/skills ~/.config/opencode/skills
# or, for Claude Code compatibility:
ln -s ~/.scicomp-research-skills/skills ~/.claude/skills
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [a-attia/scicomp-research-skills](https://github.com/a-attia/scicomp-research-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
