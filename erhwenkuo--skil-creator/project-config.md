---
trigger: always_on
description: This workspace contains a refactored version of the `skill-creator` Claude Code skill, extended to also work as a skill for [LangChain deepagent-cli](https://docs.langchain.com/oss/python/deepagents/cli/overview).
---

# skill-creator: Implementation Details

## Project Overview

This workspace contains a refactored version of the `skill-creator` Claude Code skill, extended to also work as a skill for [LangChain deepagent-cli](https://docs.langchain.com/oss/python/deepagents/cli/overview).

The skill lives at `skills/skill-creator/`. All Python scripts that support the skill (evals, benchmarking, description optimization) live in `skills/skill-creator/scripts/`.

---

## Environment Setup

This project uses [uv](https://docs.astral.sh/uv/) for Python environment management (Python 3.13).

```bash
# Create and activate the virtual environment
uv venv
source venv/bin/activate

# Install project + dev dependencies (pytest, pytest-cov)
uv pip install -e ".[dev]"
```

Always activate the venv before running any script:
```bash
source venv/bin/activate
python -m scripts.run_loop ...
```

---

## Architecture Overview

```
User query
    │
    ▼
Agent (Claude Code or deepagent-cli)
    │  reads name+description from skill discovery paths
    ▼
Skill triggers? ──No──► Agent answers directly
    │
   Yes
    ▼
SKILL.md body loaded into context
    │
    ├─► Subagent spawned per test case (Agent tool / task tool)
    │       └─► outputs saved to <workspace>/iteration-N/eval-*/
    │
    ├─► Grader subagent scores assertions → grading.json
    │
    ├─► aggregate_benchmark.py → benchmark.json + benchmark.md
    │
    └─► generate_review.py → browser viewer (or --static HTML)
            │
            ▼
        User reviews → feedback.json
            │
            ▼
        improve_description.py (via claude -p or deepagents -n)
            │
            ▼
        run_loop.py iterates until score peaks or max_iterations
```

The description optimization path is separate from the skill-creation loop — it runs *after* the user is satisfied with the skill content, to tune the SKILL.md `description` field for better trigger accuracy.

---

## Skill File Structure

```
skills/skill-creator/
├── SKILL.md                    # Main skill instructions (YAML frontmatter + markdown)
├── agents/
│   ├── analyzer.md             # Subagent: analyze benchmark results
│   ├── comparator.md           # Subagent: blind A/B comparison
│   └── grader.md               # Subagent: grade eval assertions
├── assets/
│   └── eval_review.html        # Template for description eval review UI
├── eval-viewer/
│   ├── generate_review.py      # Launches the eval results viewer
│   └── viewer.html             # Viewer UI template
├── references/
│   └── schemas.md              # JSON schemas for evals.json, grading.json, etc.
└── scripts/
    ├── __init__.py
    ├── aggregate_benchmark.py  # Aggregates run results into benchmark.json
    ├── generate_report.py      # Generates HTML report for description optimization
    ├── improve_description.py  # Calls claude -p or deepagents -n to improve skill description
    ├── install.sh              # Cross-platform installation helper
    ├── package_skill.py        # Packages skill into .skill archive
    ├── quick_validate.py       # Validates a skill directory against the spec
    ├── run_eval.py             # Trigger evaluator — Claude Code backend (claude -p)
    ├── run_eval_deepagents.py  # Trigger evaluator — deepagent-cli backend (deepagents -n)
    ├── run_loop.py             # Optimization loop (calls run_eval or run_eval_deepagents)
    └── utils.py                # Shared: parse_skill_md()
```

---

## Platform Compatibility

Both Claude Code and deepagent-cli implement the [Agent Skills specification](https://agentskills.io/specification). The `SKILL.md` format is identical between platforms.

### Allowed SKILL.md Frontmatter Keys

deepagent-cli **rejects unknown frontmatter keys**. The complete allowed set is:

| Key | Required | Notes |
|-----|----------|-------|
| `name` | Yes | Must match directory name exactly; lowercase alphanumeric + hyphens, ≤64 chars |
| `description` | Yes | ≤1024 chars |
| `license` | No | SPDX identifier |
| `compatibility` | No | ≤500 chars |
| `metadata` | No | Arbitrary key-value dict |
| `allowed-tools` | No | Space-delimited list of tool names |

### Skill Discovery Paths

| Platform | User-scope | Project-scope |
|----------|-----------|---------------|
| Claude Code | `~/.claude/skills/` | `.claude/skills/` |
| deepagent-cli | `~/.deepagents/<agent>/skills/` | `.deepagents/skills/` |
| Both (shared standard) | `~/.agents/skills/` | `.agents/skills/` |

Install to `~/.agents/skills/skill-creator` to make the skill available across both platforms.

---

## Scripts Reference

### `run_eval.py` — Claude Code Trigger Evaluator

Tests whether a skill description causes `claude -p` to trigger (read the skill) for a given query.

**Mechanism:**
1. Writes a temporary command file to `.claude/commands/` with the test description
2. Runs `claude -p <query> --output-format stream-json --include-partial-messages`
3. Detects triggering via stream events: `Skill` tool call containing the command name, or `Read` tool call on the skill path
4. Returns early on detection; kills process on timeout

**Usage (via run_loop.py):**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [erhwenkuo/skil-creator](https://github.com/erhwenkuo/skil-creator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-23 -->
