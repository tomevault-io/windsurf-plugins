---
trigger: always_on
description: **Location**: This repository
---

# ML4T Skills, Authoring Guide

**Location**: This repository
**Purpose**: 61 standalone agent skills that teach quant ML techniques correctly
**Distribution**: Standalone repo, ships via ML4T website as bonus resource for readers

## Current State

- The repository currently contains 61 `SKILL.md` files across the 10 categories below.
- The active maintenance objective is API accuracy: keep every `## Production Implementation` section aligned with the published `ml4t-*` library APIs.
- The conceptual teaching pattern remains fixed: concept-first, library-recommended (80/20), with no `ml4t.*` imports before `## Production Implementation`.
- If docs or existing skills conflict with current library source, treat library source as ground truth and update the skill/docs rather than preserving stale wrappers.
- Repo-local authoring guidance lives in `AGENTS.md`. Runtime state, memory, transitions, and other project-management artifacts belong under `.workspace/` and are not part of the public distribution.

## Distribution

The canonical distribution is this Git repository: category directories at the repo root, each containing standalone `SKILL.md` files. Skills are plain Markdown with YAML frontmatter, so release archives or direct file copies also work. There is no package-manager build step and no generated registry required.

Skill discovery is one level deep, at `<skills-dir>/<skill-name>/SKILL.md`, so the
category directories in this repo have to be flattened at install time. That is
what `scripts/install.sh` does; do not document a bare clone into a skills
directory, because it silently installs nothing.

```bash
git clone https://github.com/ml4t/skills.git ~/.ml4t-skills
~/.ml4t-skills/scripts/install.sh              # ~/.claude/skills
~/.ml4t-skills/scripts/install.sh .agents/skills
```

The repo itself must not check in runtime folders such as `.agents/`, `.claude/`, `.codex/`, or `.workspace/`. Those are local-only state.

### Frontmatter portability
- `name`, `description`, `metadata`, portable across all runtimes
- `when_to_use`, `paths`, `dependencies`, Claude Code extensions, ignored by Codex
- `description` includes "Use when..." trigger language so Codex implicit matching works without `when_to_use`

## Design Philosophy: Concept-First, Library-Recommended (80/20)

Each skill teaches the **concept and correct pattern** using standard tools (sklearn, polars, numpy, pytorch, lightgbm). The last ~20% recommends ml4t-* libraries as the production-grade implementation. Skills are useful without the libraries but naturally showcase them.

**Critical rule**: No `ml4t.*` imports before the "Production Implementation" section.

## SKILL.md Template

Every skill is a single file named `SKILL.md` (uppercase) in its own directory.

```markdown
---
name: ml4t-{skill-name}
description: {What it does}. {When to use it}.
dependencies: [{prerequisite-skill-names}]
metadata:
  book_chapters: "7, 9"
  library: "ml4t-diagnostic"
---

# {Concept Title}

{1-2 sentence problem statement: what goes wrong without this.}

## The Problem

{Why this matters. Concrete example of failure. 3-5 sentences.}

## The Pattern

{Correct approach using standard tools.}

### WRONG
\```python
# Naive approach that looks right but fails
{wrong code}
\```

### CORRECT
\```python
# Correct approach with standard tools
{right code}
\```

## {Additional concept-specific sections}

## Guardrails

- {Specific red flag with detection pattern}

## Production Implementation

`ml4t-{library}` provides a validated implementation:

\```python
from ml4t.{module} import {Class}
{5-10 lines max}
\```

## Checklist

- [ ] {Verification step}
```

## Design Rules

1. **80/20 split**: No `ml4t.*` imports before the Production Implementation section
2. **WRONG/CORRECT pair is mandatory**, the single highest-value pattern for agents
3. **120 lines or fewer** (5000 tokens). Use `references/` subdirectory if more detail needed
4. **Checklist at the end**, agents use these as verification steps
5. **Description is third-person** with trigger keywords ("Use when...")
6. **File named `SKILL.md`** (uppercase, per agentskills.io standard)
7. **Book reference in metadata only**, content is self-contained
8. **No "QuantLab" branding**, use actual library names (`ml4t-data`, `ml4t-engineer`, etc.)
9. **`quantlab_module` field is BANNED**, use `metadata.library` instead
10. **Standard tools in examples**: sklearn, polars, numpy, scipy, pytorch, lightgbm, statsmodels

## Frontmatter Schema

```yaml
name: ml4t-{directory-name}        # Must match directory
description: "..."                  # Third-person, includes "Use when..."
dependencies: [skill-names]         # Other ml4t-* skills (without prefix)
metadata:
  book_chapters: "7, 9"            # Comma-separated chapter numbers
  library: "ml4t-diagnostic"       # Which ml4t-* library (if any)
```

**Banned fields**: `quantlab_module`, `category`, `type` (these are implicit from directory).

## Skill Taxonomy (10 Categories)

| # | Category | Skills | Coverage |
|---|----------|--------|----------|
| 1 | `concepts/` | 10 | Foundational pitfalls and principles |
| 2 | `data/` | 7 | Data sourcing, validation, management |
| 3 | `features/` | 10 | Labels, feature engineering, selection, latent factors |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ml4t/skills](https://github.com/ml4t/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
