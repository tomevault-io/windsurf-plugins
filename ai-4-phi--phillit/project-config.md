---
trigger: always_on
description: **PhilLit** is a multi-agent system to (a) author academic literature reviews for philosophy research, and (b) improve these agents.
---

**PhilLit** is a multi-agent system to (a) author academic literature reviews for philosophy research, and (b) improve these agents.

# Mode

**Production mode** (default): When the user asks for a literature review, invoke the `/literature-review` skill to begin the 6-phase workflow.

**Development mode**: Only if user explicitly asks to develop, improve, or test agents/skills. Work on definitions in `.claude/agents/` and `.claude/skills/`.

# Objectives

**Priority order for literature reviews** (and agent development):

1. **Accurate** — Only cite verified papers; never fabricate references
2. **Comprehensive** — Cover all major positions and key debates
3. **Rigorous and concise** — Analytical depth, tight prose; balanced presentation of positions
4. **Reproducible** — Structured workflow, standard BibTeX output, Chicago author-date citations

**NOT priorities**:
- ❌ Speed — Quality over fast completion
- ❌ Context efficiency — Use full context as needed; don't optimize for token savings

# File Structure

- `reviews/` — All existing and new literature reviews. Each review has its own subdirectory with an informative short name. Gitignored (local only).
- `.claude/skills/literature-review/` — Main orchestration skill for the 6-phase workflow. `scripts/` contains Phase 6 tools: `assemble_review.py`, `normalize_headings.py`, `dedupe_bib.py`, `enrich_bibliography.py`, `generate_bibliography.py`, `lint_md.py`.
- `.claude/skills/philosophy-research/` — API search scripts for academic sources (Semantic Scholar, OpenAlex, CORE, arXiv, SEP, IEP, PhilPapers, NDPR), abstract resolution, encyclopedia context extraction, and citation verification (CrossRef). Includes Brave web search fallback and caching.
- `.claude/agents/` — Specialized subagent definitions invoked by the literature-review skill.
- `.claude/hooks/` — Git/Claude hooks: `bib_validator.py`, `validate_bib_write.py`, `metadata_validator.py`, `metadata_cleaner.py`, `block_background_bash.py`, `subagent_stop_bib.sh`, `setup-environment.sh`.
- `.claude/docs/` — Shared specifications (ARCHITECTURE.md, conventions.md, permissions-guide.md).
- `.claude/settings.json` — Hook definitions and permissions (checked in).
- `.claude/settings.local.json` — Local settings overrides (gitignored).
- `tests/` — pytest tests for API scripts and hooks.
- `docs/` — Project documentation and known issues.
- `GETTING_STARTED.md` — Setup guide for local and cloud environments, API key configuration.

# Typical Usage: Literature Review

When asked to perform a new literature review:
1. Invoke the `/literature-review` skill to begin the 6-phase workflow
2. The skill creates a new directory in `reviews/` with an informative short name (e.g., `reviews/epistemic-autonomy-ai/`)
3. The skill coordinates specialized subagents via the Task tool to complete all phases

# Workflow Architecture

**`/literature-review` skill** — Main entry point. Runs in main conversation with Task tool access. Coordinates the 6-phase workflow:
- Phase 1: Verify environment and determine execution mode
- Phase 2: Task tool invokes `literature-review-planner` — Decomposes topic into domains
- Phase 3: Task tool invokes `domain-literature-researcher` ×N (parallel) — Uses `philosophy-research` skill for API searches; outputs BibTeX files
- Phase 4: Task tool invokes `synthesis-planner` — Reads BibTeX files; designs outline emphasizing debates and positions
- Phase 5: Task tool invokes `synthesis-writer` ×N (parallel) — Writes sections using relevant BibTeX subsets
- Phase 6: Assemble final review, deduplicate BibTeX, generate bibliography, lint, clean up, optional DOCX

**Specialized subagents** (invoked via Task tool, cannot spawn other subagents):
- `literature-review-planner` — Decomposes topic into domains and search strategies
- `domain-literature-researcher` — Searches academic sources, produces BibTeX with rich annotations
- `synthesis-planner` — Designs tight outline from collected literature
- `synthesis-writer` — Writes individual sections of the review

# Development

For agent architecture and design patterns, see `.claude/docs/ARCHITECTURE.md`.

## Windows Compatibility

This repository works natively on Windows without WSL. Claude Code on Windows requires Git for Windows and uses Git Bash to execute hooks and commands. The SessionStart hook (`.claude/hooks/setup-environment.sh`) detects the platform and activates the correct venv path (`.venv/Scripts/activate` on Windows, `.venv/bin/activate` on Unix).

## Setup

```bash
uv sync          # Create venv and install all dependencies (including dev)
```

Regular users get only production dependencies — the SessionStart hook runs `uv sync --no-group dev`.

API keys are required for literature searches. See `GETTING_STARTED.md` for setup instructions, or run:
```bash
$PYTHON .claude/skills/philosophy-research/scripts/check_setup.py
```

## Testing

Run tests with: `pytest tests/`

## Principles

- **Keep the repository lean** — Do not keep files only for reference if the functionality is already documented elsewhere (e.g., in `pyproject.toml`). Remove deprecated files rather than marking them as such.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AI-4-Phi/PhilLit](https://github.com/AI-4-Phi/PhilLit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
