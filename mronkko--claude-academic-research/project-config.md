---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Deferred development ideas — things consciously not done yet but worth revisiting — live in [BACKLOG.md](BACKLOG.md). Consult it before starting non-trivial work; the current item may already be captured there with context for why it was deferred.

## What this repo is

An academic research **plugin** for Claude Code and Antigravity — not an application. It ships skills (prose rule-books), pipeline scripts, and templates for academic-research workflows. Claude Code users install via `/plugin marketplace add mronkko/claude-academic-research`, while Antigravity users install via `agy plugin install <url>`. Anything you change here is consumed by downstream agentic instances in user projects.

**The repo is a marketplace hosting more than one plugin.** `.claude-plugin/marketplace.json` lists them. The main plugin, `academic-research`, is sourced at the repo root (`./`) and is what most of this CLAUDE.md describes. A second, smaller plugin, `editorial-tools`, lives under `editorial-tools/` with its own `.claude-plugin/plugin.json` and `skills/` — it ships the `suggesting-reviewers` skill (peer-reviewer suggestion for journal editors/AEs, with a bundled ORM editorial-board roster). The two are independent installs: the root plugin only scans `./skills/`, so `editorial-tools/` does not leak into it, and users who install only `academic-research` never load the editorial skill. Do not assume one-plugin-per-repo.

## Common commands

```bash
# Default test run — unit tests only; live tests are deselected by marker.
pytest tests/ -q

# Single test file or test.
pytest tests/unit/test_zotero_io.py -q
pytest tests/unit/test_zotero_io.py::test_attach_pdf_raises_on_failure -q

# Live tests (real network, API keys required — opt in explicitly).
pytest -m live tests/live/
pytest -m live_browser tests/live/test_browser_publishers.py

# Lint (CI blocker).
ruff check scripts tests

# Lint with auto-fix for I001/UP037/F401/F541 etc.
ruff check scripts tests --fix
```

CI (`.github/workflows/ci.yml`) runs `ruff check scripts tests` then `pytest tests -v` on Python 3.11, 3.12, 3.13. Lint is a hard gate — a single error fails the whole matrix.

## Architecture

### Plugin surface (what users consume)

- **`skills/<name>/SKILL.md`** — each has YAML frontmatter (`name`, `description`) + a markdown body. The `description` is what the Claude Code harness matches on to decide whether to load the skill. Every procedural skill in this plugin follows the same shape: "Use when …" + `Trigger phrases: …` + a "Do NOT use for X — use Y instead" delegation rule. Breaking that shape causes the wrong skill to fire. Description bodies are kept under ~500 chars and contain no workflow summary — workflow summaries cause Claude to follow the description instead of reading the skill body. CSO doctrine is in `superpowers:writing-skills`.
- **`REQUIRED SUB-SKILL: <name>` contract.** When a skill body or per-subagent prompt names a sub-skill that way, the receiver is expected to **load the named skill via the `Skill` tool before proceeding** — never to inline the sub-skill's content into the prompt verbatim. This applies symmetrically: a main agent dispatching N parallel subagents (e.g. fact-check's per-citation `Agent` calls) tells each one which sub-skill to load; each subagent then loads it independently. The caller does *not* paste the sub-skill body into the prompt — that would re-introduce the duplication the sub-skill extraction was meant to eliminate, and force every prompt rewrite to ripple across callers. Current sub-skills used this way: `verifying-citations` (loaded by `fact-check` and by `critic-loop`'s evidence critic), `superpowers:dispatching-parallel-agents` (loaded by `fact-check` and `critic-loop` for the parallel-Agent dispatch pattern).
- **`templates/`** — copied into downstream user projects (`manuscript.qmd`, `manuscript_tables.py`, `manuscript_stats.py`, `test_citations.py`, `test_empirical_integrity.py`, `test_systematic_review.py`, `test_common.py`, `search_config.py`, `screening_config.py`, `sr_claude_md.md`, `manuscript_claude_md.md`). Changes here affect what a fresh project looks like.
- **`.claude-plugin/plugin.json`** — carries the version string. Bump only on user-visible releases, not on lint or CI fixes.

### Pipeline scripts

`scripts/pipelines/` contains the full systematic-review pipeline — one orchestrator script per stage, roughly in dependency order: `search.py` (plus four `search_<db>.py` single-DB wrappers for piloting) → `import_to_zotero.py` → enrichment (`enrich_abstracts.py`, `enrich_pdfs.py`, `enrich_dois.py`) → `abstract_screen.py` → `fulltext_code.py` → `audit_zotero_library.py` → `export_coded_includes.py` → `generate_bib.py`. The three `enrich_*` scripts replaced the pre-v0.3.0 `attach_pdfs.py` / `fetch_*.py` monolith (removed in v0.6.0). All of these orchestrators invoke:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mronkko/claude-academic-research](https://github.com/mronkko/claude-academic-research) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
