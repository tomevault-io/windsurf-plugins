---
trigger: always_on
description: Generate a DeepWiki-style structured wiki for any codebase to accelerate developer onboarding. Use this skill whenever a user wants to understand, document, or onboard into an unfamiliar repository — including requests like 'explain this codebase', 'generate documentation for this repo', 'help me understand this project', 'create an onboarding guide', 'map out the architecture', or 'how does this codebase work'. Also trigger when a user provides a repo URL or file tree and asks for analysis, or 
---


# Codebase Onboarding Wiki Generator

Generate a hierarchical, source-linked, diagram-rich wiki from any repository — modeled after DeepWiki's format.

## When to Use

- User provides a repository (URL, path, or uploaded files) and wants to understand it
- User asks to onboard developers into a codebase
- User wants architecture documentation generated from code
- User asks "how does this repo work" or "map this codebase"

## Workflow Overview

```
Phase 1: Reconnaissance ──→ Phase 2: Architecture Mapping ──→ Phase 3: Deep Docs ──→ Phase 4: Assembly
  (scripts/analyze.py)        (you + diagrams)                  (you + templates)      (final output)
```

### Phase 1 — Reconnaissance (Automated)

Run the analysis script to gather structured data about the codebase:

```bash
pip install -r scripts/requirements.txt  # optional, graceful degradation
python scripts/analyze.py <repo_path> --output codebase-analysis.json
```

## Execution Contract

Use this contract to reduce output variance across models and harnesses.

**MUST**
- Run `scripts/analyze.py` first when a repository path is available.
- Read the JSON report before writing any architecture claims.
- Check `summary.capabilities_missing` and adapt output scope accordingly.
- Cite source files for every technical claim using `path/to/file.ext:L45-L87` format.
- Mark unverifiable claims as `[NEEDS INVESTIGATION]`. Target **at least 1 per content page** — flag architectural decisions you inferred rather than directly observed in code. A wiki with zero `[NEEDS INVESTIGATION]` markers is almost certainly overconfident.
- Include **all 7 required sections** on every content page (not 00-index.md): `TL;DR` · `Relevant Source Files` · architecture diagram · `Key Concepts` table · detailed prose with citations · `Cross-references` · `Active Development Areas`.
- The index page (`00-index.md`) is navigation-only — it does **not** require TL;DR, citations, or diagrams.

**SHOULD**
- Install optional Python dependencies for better structural analysis: `pip install -r scripts/requirements.txt`.
- Prioritize `key_entities` and `git.hotspots` when choosing what to document first.
- Keep diagrams and tables aligned with observed code structure, not assumptions.
- Write **at least 400 words per content page**. Trace one code path thoroughly rather than listing many superficially. Depth beats breadth.
- Include **at least 3 `file:line` citations per major prose section**. Aim for ≥10 citations per page total.

**MAY**
- Continue with manual reconnaissance when script execution is not possible.
- Produce a reduced-scope onboarding document when capabilities are limited.

## Quality Tiers

Use these tiers to set expectations explicitly:

- **Tier A (high confidence):** `tree_sitter`, `networkx`, and `git` available. Full wiki with ranked entities and ownership/hotspot analysis.
- **Tier B (medium confidence):** At least one of `tree_sitter` or `networkx` missing. Full wiki allowed, but include limitations section.
- **Tier C (baseline):** Script unavailable or major capabilities missing. Produce structured overview only and explicitly call out unknowns.

The script runs 7 phases automatically and reports what it could and couldn't do:
1. **File discovery** — .gitignore-aware traversal (pathspec)
2. **Language stats** — Accurate LOC by language (tokei/scc)
3. **Manifest parsing** — Dependencies from package.json, pyproject.toml, Cargo.toml, go.mod, .csproj (proper parsers, not regex)
4. **Framework detection** — Cross-referenced against actual dependencies, not just filenames
5. **Code structure** — Classes, functions, interfaces, types via tree-sitter AST parsing
6. **Importance ranking** — PageRank over cross-file reference graph (networkx)
7. **Git insights** — Hotspot files, top contributors, per-directory ownership

Read the JSON output before proceeding. The `key_entities` field tells you which code entities are most important — start documentation there.

#### How to Use the Analysis Report

Each field in the JSON report drives specific documentation decisions. Follow this mapping:

**`key_entities` → Page Structure & Priority**
The PageRank-scored symbols tell you what matters most. Use them to:
- Decide which components deserve their own wiki page (top 10-15 entities almost always do)
- Determine documentation order — document highest-ranked entities first
- Identify hub abstractions: entities with high rank are referenced across many files, meaning they're architectural load-bearing walls
- Build the "Key Concepts" table on the Overview page from the top ~10 entries

Example: if `RepomixConfigMerged` ranks #1, it's central to the codebase — it gets prominent placement in Overview, its own section, and every page that touches config cross-references it.

**`symbols.by_kind` → Wiki Depth Decisions**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eabait/codebase-onboarding-skill](https://github.com/eabait/codebase-onboarding-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
