---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

This repo is the distribution for a single Claude Code **skill**, `pptx-from-layouts`, that generates consultant-grade PowerPoint decks from markdown outlines. The skill itself lives at `.claude/skills/pptx-from-layouts/`; everything else (`docs/`, `examples/`, `templates/`, `alternatives/`) is supporting material.

Installation is just a copy: `cp -r .claude/skills/pptx-from-layouts ~/.claude/skills/`. There is no package to build, no test suite, no lockfile — dependencies are `Python 3.10+` plus `pip install python-pptx pydantic` (also in `requirements.txt`). **python-pptx alone is not enough**: the `schemas/` package imports pydantic, so generation fails with `ModuleNotFoundError: No module named 'pydantic'` without it.

The skill also ships three subagents in `.claude/skills/pptx-from-layouts/agents/` (`pptx-outline-architect`, `pptx-template-onboarder`, `pptx-deck-qa`). Copies are mirrored in this repo's `.claude/agents/` so they're active during skill development; installers copy them to `~/.claude/agents/`. Keep the two copies in sync when editing.

## Commands

All entry-point scripts live in `.claude/skills/pptx-from-layouts/scripts/`. They can be run directly — they set their own `PYTHONPATH` to include the skill's `lib/`, `schemas/`, and `scripts/` when shelling out to sibling scripts.

```bash
# Generate a deck from an outline
python .claude/skills/pptx-from-layouts/scripts/generate.py outline.md -o deck.pptx

# Same, with validation pass
python .claude/skills/pptx-from-layouts/scripts/generate.py outline.md -o deck.pptx --validate

# Parse only (outline → layout_plan.json, no PPTX)
python .claude/skills/pptx-from-layouts/scripts/generate.py outline.md --layout-only -o layout.json

# Edit an existing deck (surgical text replacement / reorder)
# Replace workflow: dump inventory, edit paragraph "text" fields, pass file back.
python .claude/skills/pptx-from-layouts/scripts/edit.py deck.pptx --inventory -o inv.json
python .claude/skills/pptx-from-layouts/scripts/edit.py deck.pptx --replace inv.json -o edited.pptx
python .claude/skills/pptx-from-layouts/scripts/edit.py deck.pptx --reorder "0,2,1,3,4" -o reordered.pptx

# Validate output (structural + content + layout coverage)
python .claude/skills/pptx-from-layouts/scripts/validate.py deck.pptx
python .claude/skills/pptx-from-layouts/scripts/validate.py deck.pptx --template template.pptx
python .claude/skills/pptx-from-layouts/scripts/validate.py deck.pptx --diff other.pptx -o diff.md

# Profile a new template (emits a visual-type → layout-index config)
python .claude/skills/pptx-from-layouts/scripts/profile.py template.pptx --generate-config
```

### Default template + config

`generate.py` resolves its defaults to `<project-root>/templates/inner-chapter.pptx` and `<project-root>/templates/inner-chapter-config.json` (both bundled), so from this repo root a bare `generate.py outline.md -o out.pptx` works with zero `--template`/`--config` flags. The config is committed (generated via `generate_config.py --generate-ic-defaults`) and doubles as the reference example of a well-formed template config.

```bash
# Works with no flags from the repo root:
python .claude/skills/pptx-from-layouts/scripts/generate.py \
    examples/q1-strategy/outline.md -o out.pptx
```

The example in `examples/q1-strategy/` is the canonical smoke test — regenerate it and diff against `examples/q1-strategy/output.pptx` to verify changes.

## Architecture

The whole skill is organized around one thesis: **use the template's slide-master layouts, don't overlay text boxes.** Every design choice flows from that.

### Pipeline

```
outline.md
   │  ingest.py        (markdown + visual-type declarations → structured plan)
   ▼
layout_plan.json
   │  generate_pptx.py (looks up layout index per visual type, fills placeholders)
   ▼
output.pptx
   │  quality_check.py / validate_pptx.py
   ▼
validation report
```

Template profiling is a separate offline step: `profile_template.py` walks a `.pptx`'s slide masters and emits a config that maps visual-type names (`hero-statement`, `process-3-phase`, `table`, …) to layout indices. Generation is just plan + config + template → PPTX.

### Directory map (skill-internal)

```
.claude/skills/pptx-from-layouts/
├── SKILL.md                    # Skill manifest + user-facing quick start
├── scripts/                    # CLI entry points and core pipeline steps
│   ├── generate.py  edit.py  validate.py  profile.py   ← user-facing entry points
│   ├── ingest.py  generate_pptx.py  profile_template.py  validate_pptx.py
│   ├── content_fitter.py  content_splitter.py  content_recovery.py
│   ├── diff_pptx.py  preview_layout.py  gantt_renderer.py
│   └── quality_check.py  visual_validator.py  validation_orchestrator.py
├── lib/                        # Reusable modules imported by scripts/
│   ├── inventory.py  replace.py  rearrange.py    # edit-mode primitives
│   ├── font_fallback.py  margins.py  pptx_compat.py
│   ├── graceful_degradation.py  performance.py  confidence.py  thumbnail.py
├── schemas/                    # Pydantic models (layout_plan, template_config,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tristan-mcinnis/pptx-from-layouts-skill](https://github.com/tristan-mcinnis/pptx-from-layouts-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
