---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

Draft is a Claude Code plugin that implements Context-Driven Development methodology. It provides a two-tier command surface: 4 primary workflow commands (`/draft:init`, `/draft:new-track`, `/draft:implement`, `/draft:review`) plus 5 routers (`/draft:plan`, `/draft:ops`, `/draft:docs`, `/draft:discover`, `/draft:jira`) as the recommended public interface. 24 specialist commands are dispatched underneath the routers. The unified `/draft:jira` router supports `preview`, `create`, and the advanced `review <JIRA-ID>` qualification pipeline (deep-review + bughunt + coverage + test-gap analysis). Run `/draft` for the full intent map. Total surface: 33 skills.

Draft also ships a **knowledge graph engine** — `codebase-memory-mcp`, fetched on install to `~/.cache/draft/bin/` (not vendored; see `bin/README.md`) — driven by `scripts/tools/` (51 deterministic shell helpers). Skills are markdown (source of truth, processed by a bash build script into platform-specific integration files for Copilot and Gemini); the graph engine and shell helpers handle mechanical work that markdown can't.

## Build & Test Commands

```bash
make build              # Generate integration files from skills
make build-integrations # Same as above (explicit target)
make test               # Run all 72 test suites (skills, build, tools)
make lint               # Run shellcheck + markdownlint
make clean              # Remove generated integrations

# Run a single test
./tests/test-skill-frontmatter.sh
./tests/test-build-integrations.sh
./tests/test-tools-classify-files.sh
# etc. — any test in tests/ is independently executable

# Graph engine (codebase-memory-mcp — fetched on install, not vendored)
scripts/fetch-memory-engine.sh                          # install engine to ~/.cache/draft/bin/
scripts/tools/graph-snapshot.sh --repo .                # index repo + write draft/graph/schema.yaml gate
scripts/tools/hotspot-rank.sh --repo .                  # fan-in-ranked hotspots (live query)
scripts/tools/graph-impact.sh --repo . --symbol <name>  # blast radius for a symbol (live query)

# Prerequisites: Bash 4.0+, jq (graph tools), Node 18+ (draft CLI), shellcheck, markdownlint-cli (lint only)
```

Tests use a custom bash framework (`tests/test-helpers.sh`) with `assert()`, `pass()`, `fail()` helpers. No external test runner.

## Architecture

### Build Pipeline (the critical path)

```
skills/<name>/SKILL.md  ──┐
core/methodology.md       ├──→  scripts/build-integrations.sh  ──→  integrations/copilot/.github/copilot-instructions.md
core/shared/*.md          │                                          (~23,600 lines, auto-generated)
core/templates/*.md       ├──→  (Gemini uses bootstrap .gemini.md — no longer generated)
core/agents/*.md          ──┘
```

The build script (`scripts/build-integrations.sh`) reads `SKILL_ORDER`, `CORE_FILES`, and `TOOLS` from `scripts/lib.sh` (sourced) and:
1. Iterates `SKILL_ORDER` (33 skills in current two-tier model, order matters)
2. Validates YAML frontmatter (`name:` and `description:` required)
3. Validates body format: blank, `# Title`, blank, then content
4. Extracts body via `extract_body()`, skipping frontmatter
5. Applies syntax transforms (`/draft:command` → `draft command`; `@architect`, `@debugger`, etc. → `@workspace` for Copilot)
6. Inlines 66 core reference files (methodology, shared procedures, templates, agents, guardrails)
7. Writes atomically to a temp file then renames into place
8. Runs `verify_output()` — line count, completeness, syntax

### Source of Truth Hierarchy

1. **`core/methodology.md`** — Master methodology (update first)
2. **`skills/<name>/SKILL.md`** — Skill implementations (derive from methodology)
3. **`integrations/copilot/.github/copilot-instructions.md`** — GENERATED, never edit directly

### Key Directories

- **`core/shared/`** — Shared procedures loaded by skills (context loading, git metadata, pattern learning, cross-skill dispatch, Jira sync, **graph queries**, **parallel analysis**, VCS commands)
- **`core/agents/`** — Behavioral protocols for specialized agents (architect, debugger, planner, rca, reviewer, ops, writer)
- **`core/templates/`** — 28 templates for files that `/draft:init` generates in user projects (24 top-level + 4 `okf/` bundle templates for `DRAFT_INIT_MODE=okf`)
- **`bin/`** — Holds only `README.md`. The graph engine (`codebase-memory-mcp`) is **not vendored** — it is fetched on install (`scripts/fetch-memory-engine.sh`) to `~/.cache/draft/bin/` and resolved by `scripts/tools/_lib.sh:find_memory_bin()` (`DRAFT_MEMORY_BIN` → `$PATH` → cache). Output gate marker under `draft/graph/schema.yaml`; all structural data is queried live. CLI and schema documented in `bin/README.md`.
- **`scripts/tools/`** — 51 deterministic shell helpers (git-metadata, classify-files, hotspot-rank, cycle-detect, graph-* capability wrappers, `resolve-tools.sh`, etc.). Skills call these for mechanical work. All knowledge-graph Cypher lives in the sourced `_graph_queries.sh` module (single source of query truth); the `graph-*.sh` wrappers are thin arg-parse → builder → fail-loud JSON.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mayurpise/draft](https://github.com/mayurpise/draft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
