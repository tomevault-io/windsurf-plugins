---
trigger: always_on
description: This file is the authoritative agent contract for Mnemosyne. It provides guidance
---

# AGENTS.md

This file is the authoritative agent contract for Mnemosyne. It provides guidance
to Claude Code and any other AI agent working with this repository.

## Project Overview

Mnemosyne is the skills and session-memory store for the HomericIntelligence agentic
ecosystem. It stores, organizes, and shares learnings from experiments, debugging sessions,
and development work as flat skill files under `skills/`.

Mnemosyne is **not** a plugin marketplace — the `/advise` and `/learn` commands live in the
**Athena** plugin, which is the marketplace. Mnemosyne only holds the corpus those commands
read and write. Do not add a `.claude-plugin/marketplace.json` to this repo.

**Purpose**: Capture team knowledge so Claude can `/advise` before starting work and prevent
repeated mistakes.

**Ecosystem**: Works with ProjectOdyssey (training), ProjectKeystone (DAG execution), ProjectScylla (testing), Myrmidons (agent provisioning), AchaeanFleet (agent images), and ProjectHephaestus (shared utilities).

## Commands

### /advise

Search the skills registry before starting work.

**When to use**: At session start, before experiments, when debugging unfamiliar errors.

**Workflow**:

1. Read user's goal/question
2. Search the `skills/` corpus for related skill files
3. Read matching skill `.md` files
4. Return: what worked, what failed, recommended parameters

**Example**:

```text
User: /advise training a model with GRPO
Claude: Found 2 related skills...
- training/grpo-external-vllm: Use external vLLM server for GRPO training
  - Key finding: vllm_skip_weight_sync errors require separate GPU setup
  - Recommended: batch_size=4, learning_rate=1e-5
```

### /learn

Save learnings after a session (auto-creates PR).

**When to use**: After experiments, debugging sessions, or implementing new patterns.

**Workflow**:

1. Read entire conversation history
2. Extract: objective, steps taken, successes, failures, parameters
3. Auto-generate skill filename: `<topic>-<subtopic>-<short-4-word-summary>`
4. Create git worktree for branch isolation (all work done in worktree, not base repo)
5. Generate skill file from template:
   - `skills/<name>.md` with YAML frontmatter + all required sections
   - Optional `skills/<name>.notes.md` for raw session details
6. Create branch: `skill/<name>`
7. Commit and push
8. Create PR with summary
9. Clean up worktree with `git worktree remove`

**Auto-trigger**: UserPromptSubmit hook reminds about `/learn` when you type session-ending keywords.

**Format notes**:
- Category/name are auto-generated (no user prompting)
- Work isolation: git worktrees (auto-created per branch, cleaned up after PR)
- Single flat `.md` file with YAML frontmatter
- Branch name: `skill/<name>`

## Plugin Standards

### Required Structure

```text
skills/<name>.md             # Main skill file with YAML frontmatter + markdown content
skills/<name>.notes.md       # (Optional) Additional context from development session
```

All skills are now flat files in the `skills/` directory. Metadata is stored as YAML frontmatter in each `.md` file.

**Exception**: `plugins/tooling/mnemosyne/` stays in `plugins/` — it's the command infrastructure (advise, learn), not a skill.

### Required Fields

**YAML Frontmatter** (in `skills/<name>.md`):

- `name`: Lowercase, kebab-case identifier
- `description`: Trigger conditions with specific use cases ("Use when: (1) ..., (2) ...")
- `category`: One of 9 approved categories
- `date`: Creation date (YYYY-MM-DD)
- `version`: Semantic version (e.g., "1.0.0")
- `user-invocable`: Set to `false` for internal/sub-skills (declutters slash command menu)
- `tags` (optional): Searchable keywords array

**Markdown Sections**:

- Overview table (date, objective, outcome)
- When to Use (trigger conditions)
- Verified Workflow (what worked, including Quick Reference subsection)
- **Failed Attempts table (REQUIRED)** — must include: Attempt, What Was Tried, Why It Failed, Lesson Learned
- Results & Parameters (copy-paste ready configs, expected outputs)

### Categories

| Category | Description |
| ---------- | ------------- |
| `training` | ML training experiments and hyperparameters |
| `evaluation` | Model evaluation and metrics |
| `optimization` | Performance tuning and speedups |
| `debugging` | Bug investigation and fixes |
| `architecture` | Design decisions and patterns |
| `tooling` | Automation and developer tools |
| `ci-cd` | Pipeline configurations and CI fixes |
| `testing` | Test strategies and patterns |
| `documentation` | Paper writing, academic reviews, knowledge docs |

### Quality Rules

1. **Specific descriptions**: Include trigger conditions, not vague summaries
2. **Failures required**: Document what didn't work and why
3. **Copy-paste ready**: Parameters and configs should work immediately
4. **No duplication**: Link to external docs instead of copying

### Key Development Principles

1. KISS - *K*eep *I*t *S*imple *S*tupid -> Don't add complexity when a simpler solution works
1. YAGNI - *Y*ou *A*in't *G*onna *N*eed *I*t -> Don't add things until they are required
1. TDD - *T*est *D*riven *D*evelopment -> Write tests to drive the implementation
1. DRY - *D*on't *R*epeat *Y*ourself -> Don't duplicate functionality, data structures, or algorithms

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HomericIntelligence/Mnemosyne](https://github.com/HomericIntelligence/Mnemosyne) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
