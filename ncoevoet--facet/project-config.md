---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Rules

- **No backward-compatibility fallbacks.** When renaming or restructuring config keys, methods, or APIs, do NOT add legacy aliases, fallback lookups, or shims for old names. Update all references to use the new names directly. Old names should be removed completely.
- **No custom CSS classes in Angular components.** Use plain Tailwind CSS utilities exclusively. Never define custom CSS classes in component `styles`. Use Angular `host` property for `:host` styling (e.g., `host: { class: 'block h-full' }`). All styling must be done via Tailwind utility classes in templates.
- **Use pipes instead of method calls in Angular templates.** Never call component methods from template expressions (e.g., `{{ method(value) }}`). Use Angular pipes for data transformation in templates to avoid unnecessary change detection cycles.

## Code Review

Run `/agents:code-review-agent` to review commits and changes. Supports reviewing the last commit, uncommitted changes, or specific files with configurable depth (quick/standard/deep) and focus areas (security, performance, sql, i18n, config).

## Available Skills

| Skill | Triggers | Purpose |
|-------|----------|---------|
| `signal-patterns` | signal, computed, effect, UI not updating, array mutation, object mutation, zoneless, change detection | Signal-based state management for Angular 20 |
| `effect-safety-validator` | infinite loop, NG0101, Maximum call stack, ObjectUnsubscribedError, effect safety, form patchValue | Detect unsafe effect patterns in Angular signals |
| `test-creation` | create tests, fix test, TS2345, NullInjectorError, fakeAsync, flushEffects, test coverage | Test suites for Angular 20 zoneless signal components |
| `code-quality-analyzer` | duplicate code, DRY, refactor, code smell | Code smells and refactoring opportunities |
| `css-layout-patterns` | @apply, flex layout, overflow, dark theme, responsive | CSS/Tailwind v4 layout patterns |
| `chrome-devtools-debugging` | UI issue, button not working, network request, console error, 422 error, screenshot | Browser debugging with Chrome DevTools MCP |
| `/reflexion` | audit .claude, ecosystem health | Audit .claude/ ecosystem for quality and coherence |
| `/adaptive` | complex task, multi-step, orchestrate agents | Autonomous multi-agent workflow orchestrator |

## Patterns (`.claude/patterns/`)

Checklists for recurring multi-file changes — consult before starting:

| Pattern | When to use |
|---------|-------------|
| [`new-metric-checklist.md`](.claude/patterns/new-metric-checklist.md) | Adding a new scoring metric (schema, scorer, config validator, API, client) |
| [`i18n-sync.md`](.claude/patterns/i18n-sync.md) | Adding or renaming user-facing strings across all 5 languages |

## Project Overview

Facet is a multi-dimensional photo analysis engine that examines every facet of an image — from aesthetic appeal and composition to facial detail and technical precision — using an ensemble of vision models to surface the photos that truly shine.

**Documentation:** See `docs/` for detailed documentation:
- [docs/CONFIGURATION.md](docs/CONFIGURATION.md) - Full `scoring_config.json` reference with correct defaults
- [docs/COMMANDS.md](docs/COMMANDS.md) - All CLI commands
- [docs/SCORING.md](docs/SCORING.md) - Category system and weight tuning
- [docs/FACE_RECOGNITION.md](docs/FACE_RECOGNITION.md) - Face workflow and clustering
- [docs/VIEWER.md](docs/VIEWER.md) - Web gallery features
- [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md) - Production deployment (Synology NAS, Linux, Docker)

## Commands

```bash
# Score photos in a directory (auto multi-pass mode, VRAM auto-detection)
python facet.py /path/to/photos

# Force single-pass mode (all models loaded at once, requires high VRAM)
python facet.py /path/to/photos --single-pass

# Run specific pass only
python facet.py /path/to/photos --pass quality       # TOPIQ only
python facet.py /path/to/photos --pass quality-iaa   # TOPIQ IAA (aesthetic merit)
python facet.py /path/to/photos --pass quality-face  # TOPIQ NR-Face (face quality)
python facet.py /path/to/photos --pass quality-liqe  # LIQE (quality + distortion diagnosis)
python facet.py /path/to/photos --pass tags          # Configured tagger only
python facet.py /path/to/photos --pass composition   # SAMP-Net only
python facet.py /path/to/photos --pass faces         # InsightFace only
python facet.py /path/to/photos --pass embeddings    # CLIP/SigLIP embeddings only
python facet.py /path/to/photos --pass saliency      # BiRefNet subject saliency

# Force re-scan of already processed files
python facet.py /path/to/photos --force

# Preview mode - score sample photos without saving (default: 10 photos)
python facet.py /path/to/photos --dry-run
python facet.py /path/to/photos --dry-run --dry-run-count 20

# Re-tag photos with configured tagger model
python facet.py --recompute-tags
python facet.py --recompute-tags-vlm    # Re-tag using VLM tagger
python facet.py --recompute-iqa         # Recompute supplementary IQA (TOPIQ IAA, NR-Face, LIQE) from thumbnails

# List available models and VRAM requirements
python facet.py --list-models


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ncoevoet/facet](https://github.com/ncoevoet/facet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
