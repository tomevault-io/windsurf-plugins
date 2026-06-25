---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This repo is an **Agent Skills repository** for publishing performance testing skills to [skills.sh](https://skills.sh) / [agentskills.io](https://agentskills.io). It does not contain runnable tests — it contains skill definitions that guide AI agents in building them.

Each skill lives in `skills/<skill-name>/` and is independently installable. The repo is designed to grow: new skills for other tools (k6, JMeter, Locust, etc.) follow the same structure.

## Repository Structure

```
skills/
└── <skill-name>/          ← directory name must match the `name` field in frontmatter
    ├── SKILL.md            ← required; YAML frontmatter + markdown instructions
    ├── evals/              ← optional; evals.json test cases for quality assurance
    │   └── evals.json
    ├── references/         ← optional; topic files loaded on demand by agents
    │   └── PROTOCOLS.md
    └── scripts/            ← optional; automation scripts referenced from SKILL.md
        ├── scaffold.sh
        └── validate.sh
```

The standard discovery path for the skills CLI is `skills/<skill-name>/SKILL.md`. No category subfolders.

## SKILL.md Format

Every skill file must follow the [Agent Skills specification](https://agentskills.io/specification):

```yaml
---
name: skill-name          # required: lowercase, hyphens only, max 64 chars, must match parent dir name
description: >            # required: max 1024 chars, must say WHAT the skill does AND WHEN to use it
  ...
license: MIT              # optional but recommended
compatibility: ...        # optional; list supported agents (Claude Code, Cursor, Windsurf) and runtimes
model: sonnet             # optional; which Claude model to use (sonnet recommended for code generation)
allowed-tools: Read, Bash # optional; restricts tools when skill is active (omit to allow all tools)
metadata:                 # optional
  author: ...
  version: "1.0"
---
```

**`name` constraints:** lowercase + hyphens only · no leading/trailing hyphens · no consecutive hyphens (`--`) · max 64 chars · must exactly match the parent directory name.

**`description` constraints:** must include both what the skill does and the trigger conditions (when to use it) · max 1024 chars · include keywords agents can match on.

## Content Guidelines

- Keep `SKILL.md` under 500 lines. Move detailed reference material to `references/`.
- Files in `references/` are loaded on demand — keep them focused and single-topic.
- The body has no format restrictions; write whatever helps agents perform the task.
- Recommended sections: When to Use · Instructions · step-by-step DSL guidance · Best Practices.

## Adding a New Skill

1. Create `skills/<new-skill-name>/SKILL.md` — the directory name must equal the `name` frontmatter field.
2. Validate frontmatter against the constraints above before committing.
3. If the body exceeds ~500 lines, extract reference material into `skills/<new-skill-name>/references/`.
4. Add the skill to the `Current Skills` table below and to `README.md`.

## Testing a Skill

There are three levels of testing, from fastest to most thorough.

### Level 1 — Structural validation (automatic, every push)

The CI workflow (`.github/workflows/validate-skills.yml`) runs `scripts/validate_skills.py` on every push to `main`. It validates frontmatter, name constraints, line count, and evals.json structure. Free, no API key required.

Run locally before committing:

```bash
python3 -m venv /tmp/skills-venv
/tmp/skills-venv/bin/pip install PyYAML -q
/tmp/skills-venv/bin/python scripts/validate_skills.py
```

### Level 2 — Manual smoke test (quick check)

After any change to a skill, install and run a prompt in a clean directory:

```bash
# 1. Create a clean test directory
mkdir ~/skills-test && cd ~/skills-test

# 2. Install the updated skill
npx skills add rcampos09/performance-testing-skills --skill <skill-name> --yes

# 3. Open Claude Code and run a test prompt
# 4. Analyze the output — look for missing patterns, wrong syntax, or incomplete scripts
# 5. Fix issues in the skill files, commit, push
```

### Level 3 — Benchmark with skill-creator (before releases)

Use [skill-creator](https://github.com/anthropics/skills/tree/main/skills/skill-creator) to run the `evals/evals.json` test cases and measure quality quantitatively. Runs each eval **with** and **without** the skill, then grades the output against assertions using Claude as a judge.

**One-time setup:**

```bash
mkdir ~/skill-creator-workspace
cd ~/skill-creator-workspace
npx skills add anthropics/skills --skill skill-creator --yes
```

**Running a benchmark** (open Claude Code in `~/skill-creator-workspace`):

```
Test my skill <skill-name> located at
/path/to/skills/<skill-name>/
Use the evals in evals/evals.json.
Create a workspace at ~/skill-creator-workspace/<skill-name>-iteration-1
```

**What the benchmark produces:**

| Metric | Meaning |
|---|---|
| Pass rate with skill | % of assertions that pass when skill is active |
| Pass rate without skill | Baseline — what the model does alone |
| Delta | The value the skill adds (+pp = percentage points) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rcampos09/performance-testing-skills](https://github.com/rcampos09/performance-testing-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
